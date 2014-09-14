How to plan a project and prepare for development 
=================================================

In this section we will start off by contemplating our CMS, identifying some desired features, and reviewing how to implement those features in Django. Then, we will cover a basic developer workflow, introducing concepts such as Unit Testing and Static Analysis along the way. Next, we'll begin to implement our changes, keeping an eye out for gotchas and process optimization opportunities. Following that, we'll begin to define "devops" more formally for our context and lay some ground rules for devops process development. Finally, we'll review our new position, reflecting on our development process and begin to set forth a basic, yet scalable devops practices.

A little Detour...
------------------
Every since I first started DevOps consulting, way back before it was even called that, it's always sort of felt like cleaning up after sloppy developers. I've seen some really dumb things, like install prefixes set to /usr/share. Or, web apps coded in such a way that they open a database connection for every single HTTP connection (and doesn't close them). I could go on, but I won't because this sort of ranting and negative attitude is not at all useful for a DevOps Engineer. If you find yourself thinking, "How stupid!", try to have a Zen moment, and just be at peace with the fact that people often do dumb things when the're learning. This is especially true in IT, a field so vast you can dedicate your entire career to it and still only master a small part. A good DevOps Engineer is tolerant and friendly. When faced with something that could have obviously been done better, it's their role to educate and improve. Not berate. The primary purpose of DevOps is to smooth over the maintenance cycle and improve process at the point where engineering meets operations. Fundamentally, it's means making tools and establishing processes that make everybody's jobs easier. Over the years I've learned that, practically, "DevOps" really means "Operationalizing" software and smoothing, perhaps even completely automating, the release and maintenance process.

What does "operationalize" mean in this context? It could mean requiring a laundry list of tasks for seamless OS/Platform integration. Items for an Enterprise Linux implementation might include:

  - Creating rpm and source rpm files of tagged releases
  - Changing location of user or system generated files, such as user uploaded content or application logs
  - Wrapping, modifying or otherwise tailoring the program's runtime initialization such as systemd.service files and rc init that wrap them
  - Patching the source code for a specific reason that usually fixes or improves how the program runs on the target platform

Often times, these changes should be implemented only for the target platform, and thus it is not appropriate for DevOps for force unreasonable policies onto developers. Dredging them down with demands such as proper spec file creation, when they don't even develop on EL based Linux, is not going to go well. So, how do we deal with the need to integrate with the platform without bogging engineers down with all this mind-blowing (or more often basic Linux) System Administration stuff? That is the challenge of DevOps and, not surprisingly, this is remarkably similar with what distro maintainers deal with on the regular. They take a project, apply their changes so it integrates cleanly with their distro, and repackages it as a deb, rpm or whatever.

Planning our CMS
----------------
For this exercise we're going to build a Q&A site. We'll want a basic homepage, and a seperate django app for the Q&A module. We're going to implement this in two iterations. The first iteration we'll build the root site. In the second iteration we'll create a application for the Q&A module of the site.

Here's a rundown of our task list:

  - Create a model to represent pages on our root application
  - Create a view to display our page content
  - Modify url dispater to route to our default view
  - Add unit tests to verify end-to-end db to view

Model your data
~~~~~~~~~~~~~~~
Our first model will be for page content on our root site. Models in django typically map to tables in your backend. For this exercise, we'll use the default sqlite3 provider. The page model will contain a Title, a Name, and Content.

::

  #johnnycage/models.py

  from django.db import models

  class Page(models.Model):
      name = models.CharField(max_length=32)
      content = models.TextField()
      title = models.CharField(max_length=200, default='Change Me')
      
      def __str__(self):
          return self.name

Wire up the model
~~~~~~~~~~~~~~~~~
After saving the model, edit <code>johnnycate/settings.py</code> and add 'johnnycage' to the list of INSTALLED_APPS.

It's handy to know what and how many apps are "installed". Use grep like so to print the INSTALLED_APPS section:

::

  $ grep -A 8 INSTALLED_ johnnycage/settings.py
  INSTALLED_APPS = (
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      'johnnycage',
  )

Next, we have to make the root project migrations aware:

::

  $ ./manage.py makemigrations johnnycage
  Migrations for 'johnnycage':
    0001_initial.py:
      - Create model Page

Now let's look at the sql our initial migration will generate:

::

  $ ./manage.py sqlmigrate johnnycage 0001
  BEGIN;
  CREATE TABLE "johnnycage_page" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "name" varchar(32) NOT NULL, "content" text NOT NULL, "title" varchar(200) NOT NULL);

  COMMIT;

Next, let's apply the migration.

::

  $ ./manage.py migrate johnnycage 0001
  Operations to perform:
    Target specific migration: 0001_initial, from johnnycage
  Running migrations:
    Applying johnnycage.0001_initial... OK

Finally, we'll want to wire up the model to our admin interface so we can edit pages.

::

  # johnnycage/admin.py

  from django.contrib import admin
  from . import models

  admin.site.register(models.Page)

At this point, if you haven't already, create a superuser account for this project.

::

  $ ./manage.py createsuperuser
  Username (leave blank to use 'rot'): admin
  Email address: admin@localhost
  Password: 
  Password (again): 
  Superuser created successfully.

Be sure to set the username, email address and password appropriately.

Now you should be able to login to the admin interface and create your first page.

Create the view
~~~~~~~~~~~~~~~

::

  # johnnycage/view.py

  from django.shortcuts import render
  from johnnycage.models import Page

  def index(request):

      p = Page.objects.get(name__iexact='home')

      return render(request, 'johnnycage/index.html',
                    {'page': p})

In the above view, we've setup a mechanism which depends on a Page with the name "home". This is another potential pain point that needs to be communicated clearly. Otherwise, users of the CMS might get confused, and understandably so since we don't leave any clue that you need to create a specially named page for the site to display properly.

At this point, our site won't display for two reasons. One, we haven't create the view's template. Two, we haven't told the url dispatcher about our new view.


Create the templates
~~~~~~~~~~~~~~~~~~~~
First, create a directory to put templates for our root site.

::

  $ mkdir -p johnnycage/templates/johnnycage

Then create the template for the index view.

::

  {% load staticfiles %}
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <link rel="stylesheet" href={% static "johnnycage/style.css" %} />
      <title>{% block title %}My amazing site{% endblock %}</title>
  </head>

  <body>
    <div id="navbar">
      {% block navbar %}
      <p>Site Navigation:</p>
      <ul>
        <li><a class="active" href="/">Home</a></li>
        <li><a href="{% url 'articles:index' %}">Blog</a></li>
      </ul>
      {% endblock %}
    </div>

    <div id="content">
      {% block content %}{% endblock %}
    </div>

    {% block ga %}{% endblock %}
  </body>
  </html>

And also the index template.

::

  {% extends "johnnycage/base.html" %}

  {% block title %}Johnny Cage{% endblock %}

  {% block content %}
  <div id="center_col">
      <h1>{{ page.title }}</h1>
      {{ page.content|safe }}
  </div>
  {% endblock %}


Wire up the URL
~~~~~~~~~~~~~~~
Modify johnnycage/urls.py and set a route to the home page.

::

  from django.conf.urls import patterns, include, url
  from django.contrib import admin

  urlpatterns = patterns('',
      # Examples:
      # url(r'^$', 'johnnycage.views.home', name='home'),
      # url(r'^blog/', include('blog.urls')),

      url(r'^$', 'johnnycage.views.index', name='home'),
      url(r'^admin/', include(admin.site.urls)),
  )

At this point, if we run the app, we'll notice a critical flaw that was pointed out earlier. The home page content doesn't not exist in the database so we get the error 
  Exception Type:  DoesNotExist
  Exception Value:  Page matching query does not exist.

Now, many developers might consider this ok, after all, we can just login to the admin interface and create the page. For now, we'll go ahead and do that however this will become a problem when we have to deliver the software to the client.

After creating a page with the name "home" you should be able view the home page.
