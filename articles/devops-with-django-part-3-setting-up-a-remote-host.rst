How to setup Django, mod_wsgi, and nginx on CentOS 7
====================================================
At this point in the series we have a working CMS with a nice layout and
functional dynamic content. We're also setup to use database migrations which
will save us a lot of hassle when deploying updates. Now, it's time to put the
CMS on a remote host. In this part of the series, DevOps w/Django, we'll get 
Virtual Private Server with CentOS 7 pre-installed from Digital Ocean, set it
up with nginx, Apache httpd with mod_wsgi (and talk about why mod_wsgi?),
configure firewalld and otherwise deploy our CMS. Once our site is live, we'll
update the content via the administration interface. Finally, we'll review and
discuss all this from a "DevOps" perspective.


Setup the VPS
+++++++++++++

In this tutorial, couter to the devops way, we'll setup up everything manually.
This is to follow a similar evolotution of many real world projects. Often,
projects are created by a single developer and work only on their machines.
When it comes time to install the solution in another environment, often times
it's painful. Later in the series we'll scale up our development process,
adding developers and changing the deployment process.

Here's the basic checklist for our implementation:

- Install the necessary system packages
- Configure httpd, mod_wsgi, and firewalld
- Configure nginx, reconfigure httpd bind address.
- Setup a bare clone of our git repo with a post-receive hook to deploy changes
- Create a Makefile with a local deploy target
- Test the commit deployment by making a commit


Install System Dependencies
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Install python2, httpd, nginx and any other programs you need such as vi improved.

.. code:: bash
 
  $ sudo yum install httpd nginx python2

Create a python virtual environment for our cms johnnycage.

.. code:: bash

  # mkdir /opt/johnnycage
  # virtualenv /opt/johnnycage


Setup Bare Git Repo
~~~~~~~~~~~~~~~~~~~

Create a bare git repository for our project.

.. code:: bash

  $ mkdir /srv/git/
 
From the development environment, create the bare repo and copy it to the remote host.

.. code:: bash

  $ git clone --bare johnnycage johnnycage.git
  $ scp -r johnnycage.git remotehost:/srv/git/

Add a git remote in the development repo pointing to the bare repo on the remote host.

.. code:: bash

  $ git remote add remotehost ssh://remotehost:/srv/git/johnnycage.git

In the bare repo on the remote host, add the following post-receive hook:

.. code:: bash

  #!/bin/sh

  DEST=/var/www/remotehost_com
  echo "Deploying into $DEST"

  GIT_WORK_TREE=$DEST git checkout --force
  cd $DEST
  make deploy


Configure Apache HTTPD
~~~~~~~~~~~~~~~~~~~~~~

Reconfigure httpd to listen on 127.0.0.1:8080.

.. code:: bash

  $ sudo vim /etc/httpd/conf/httpd.conf

On line 41 change Listen to:

  Listen 127.0.0.1:8080

Create an httpd virtual host container for hosting our application.

.. code::

  $ sudo vim /etc/httpd/conf.d/remotehost_com.conf
 
Insert the following:

.. code::

  <VirtualHost 127.0.0.1:8080>
      ServerName remotehost.com
      ServerAdmin contact@remotehost.com

      Alias /robots.txt /var/www/remotehost_com/static/robots.txt
      Alias /favicon.ico /var/www/remotehost_com/static/favicon.ico

      AliasMatch ^/([^/]*\.css) /var/www/remotehost_com/static/styles/$1

      Alias /media/ /var/www/remotehost_com/media/
      Alias /static/ /var/www/remotehost_com/static/

      <Directory /var/www/remotehost_com/static>
          Require all granted
      </Directory>

      <Directory /var/www/remotehost_com/media>
          Require all granted
      </Directory>

      WSGIDaemonProcess remotehost_com python-path=/var/www/remotehost_com:/opt/pyenvs/johnnycage/lib/python2.7/site-packages
      WSGIProcessGroup remotehost_com
      WSGIScriptAlias / /var/www/remotehost_com/johnnycage/wsgi.py

      LogLevel info
      CustomLog /var/log/httpd/remotehost_com_access.log combined
      ErrorLog /var/log/httpd/remotehost_com_error.log

      <Directory /var/www/remotehost_com/johnnycage>
          <Files wsgi.py>
              Require all granted
          </Files>
      </Directory>

  </VirtualHost>

Call the post-receive hook to trigger a deployment to the httpd virtualhost
container. This will checkout the latest commit and create the directory
structue httpd expects.

.. code:: bash

  $ /srv/git/johnnycage.git/hooks/post-receive

Test httpd server configuration.

.. code:: bash

  $ apachectl -t
 

Configure nginx
~~~~~~~~~~~~~~~
The edit the main nginx config file (/etc/nginx/nginx.conf) as shown below (comments removed):

.. code:: bash

  user  nginx;
  worker_processes  1;

  error_log  /var/log/nginx/error.log;
  pid        /run/nginx.pid;

  events {
      worker_connections  1024;
  }

  http {
      include       /etc/nginx/mime.types;
      default_type  application/octet-stream;

      log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                        '$status $body_bytes_sent "$http_referer" '
                        '"$http_user_agent" "$http_x_forwarded_for"';

      access_log  /var/log/nginx/access.log  main;

      sendfile        on;
      keepalive_timeout  65;

      include /etc/nginx/conf.d/*.conf;

      index   index.html index.htm;

      server {
          listen       192.241.206.174:80;
      
          server_name  remotehost.com;
          root         /var/www/remotehost_com;

          location / {
            proxy_pass http://127.0.0.1:8080;
            proxy_set_header X-Forwarded-Host $server_name;
            proxy_set_header X-Real-IP $remote_addr;
          }

          location /static/ {
            # the trailing slash is necessary
            alias /var/www/remotehost_com/static/;
          }

          error_page  404              /404.html;
          location = /40x.html {
          }

          error_page   500 502 503 504  /50x.html;
          location = /50x.html {
          }
      }
  }

