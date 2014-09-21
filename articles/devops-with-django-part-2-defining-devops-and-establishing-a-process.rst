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
For this exercise we're going to build a Q&A site. We'll want a basic homepage, and a seperate django app for the Q&A module. We're going to implement this in two iterations. The first iteration we'll build the root site. In the second iteration we'll create an application for the Q&A module of the site. With each iteration we'll tag a *release* version, which we'll use for packaging our project later in the series. For now, we won't be concerned with deploying our project.

Dynamic Home Page
~~~~~~~~~~~~~~~~~
Here's a rundown of our task list for the home page:

  - Create a model to represent pages on our root application
  - Create a view to fetch our page content and render a presentation
  - Create a base an index template
  - Modify url dispater to route to our default view
  - Create a superuser and use the admin page to publish the home page content
  - Add unit tests to verify end-to-end db to view
  - Change the look and feel

After completing the above items, tag the related change sets at version 0.0.1.

Q&A Module
~~~~~~~~~~
Here's the task list for the Q&A module.

  - Create a new application called qanda
  - Create a models for Questions, Answers, and Users
  - Create views for the module index, question details, and user profile
  - Create templates for each of the above views
  - Add routes for each of the above views
  - Add the necessary unit tests
  - Tweak the theme

As usual, create a tag for release when these features are complete.

Starting Development with the Home Page
---------------------------------------
Obviously, the above task lists will have to be broken down further. And, it's to be expected that we'll need to adapt our plan as we go along when necessry. Generally, when develop webapps with Django, the workflow goes something like this:

  - Create model and apply migrations
  - Create views
  - Create templates
  - Add routes
  - Add test cases
  - Tweak the look and feel


Model the Data
~~~~~~~~~~~~~~~
Our first model will be for page content on our root site. Models in django typically map to tables in your backend. For this exercise, we'll use the default sqlite3 provider. The page model will contain a Title, a Name, and Content.

.. code:: python

  #johnnycage/models.py

  from django.db import models

  class Page(models.Model):
      name = models.CharField(max_length=32)
      content = models.TextField()
      title = models.CharField(max_length=200, default='Change Me')
      
      def __str__(self):
          return self.name

Wire up the Model
~~~~~~~~~~~~~~~~~
After saving the model, edit <code>johnnycate/settings.py</code> and add 'johnnycage' to the list of INSTALLED_APPS.

It's handy to know what and how many apps are "installed". Use grep like so to print the INSTALLED_APPS section:

.. code:: python

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

.. code:: sql

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

.. code:: python

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

Create the View
~~~~~~~~~~~~~~~

.. code:: python

  # johnnycage/view.py

  from django.shortcuts import render
  from johnnycage.models import Page

  def index(request):

      p = Page.objects.get(name__iexact='home')

      return render(request, 'johnnycage/index.html',
                    {'page': p})

In the above view, we've setup a mechanism which depends on a Page with the name "home". This is another potential pain point that needs to be communicated clearly. Otherwise, users of the CMS might get confused, and understandably so since we don't leave any clue that you need to create a specially named page for the site to display properly.

At this point, our site won't display for two reasons. One, we haven't create the view's template. Two, we haven't told the url dispatcher about our new view.


Create the Templates
~~~~~~~~~~~~~~~~~~~~
First, create a directory to put templates for our root site.

::

  $ mkdir -p johnnycage/templates/johnnycage

Then create the template for the index view.

.. code:: django

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

.. code:: django

  {% extends "johnnycage/base.html" %}

  {% block title %}Johnny Cage{% endblock %}

  {% block content %}
  <div id="center_col">
      <h1>{{ page.title }}</h1>
      {{ page.content|safe }}
  </div>
  {% endblock %}


Make the Route Accessible
~~~~~~~~~~~~~~~~~~~~~~~~~
Modify johnnycage/urls.py and set a route to the home page.

.. code:: python

  from django.conf.urls import patterns, include, url
  from django.contrib import admin

  urlpatterns = patterns('',
      # Examples:
      # url(r'^$', 'johnnycage.views.home', name='home'),
      # url(r'^blog/', include('blog.urls')),

      url(r'^$', 'johnnycage.views.index', name='home'),
      url(r'^admin/', include(admin.site.urls)),
  )

At this point, if we run the app, we'll notice a critical flaw that was pointed out earlier. The database entry for our home page content hasn't been created yet.

::

  Exception Type:  DoesNotExist
  Exception Value:  Page matching query does not exist.

Now, many developers might consider this ok, after all, we can just login to the admin interface and create the page. For now, we'll go ahead and do that however this will become a problem when we have to deliver the software to the client.

After creating a page with the name "home" you should be able view the home page. Later on, we'll see how.

Add Test Cases
~~~~~~~~~~~~~~
We'll want to cover creating a page and rendering a view with, asserting that the page content from the database is contained in the response.

Create the file **johnnycage/tests.py** and insert the following content.

.. code:: python

  from django.test import TestCase
  from django.core.urlresolvers import reverse
  from django.utils import timezone

  from johnnycage.models import Page

  def create_page(name, content):
      Page.objects.create(name=name, content=content)

  class PageCrudTests(TestCase):
      def test_create_page(self):
          """
          Creating a page should cause no errors.
          """
          p = Page(name='home', content='Test page')
          p.save()
          self.assertEqual(p.pk, 1)

      def test_create_multiple_pages(self):
          """
          Creating 3 pages should result in 3 pages stored in the database.
          """
          create_page(name="test1", content="Test page 1")
          create_page(name="test2", content="Test page 2")
          create_page(name="test3", content="Test page 3")
          page_count = Page.objects.count()
          self.assertEqual(page_count, 3)

      def test_delete_page_with_multiple_pages(self):
          """
          Deleting a page should remove the correct page from the database.
          """
          create_page(name="test1", content="Test page 1")
          create_page(name="test2", content="Test page 2")
          create_page(name="test3", content="Test page 3")
          page3 = Page.objects.get(name="test3")
          page3.delete()
          pages = Page.objects.all()
          self.assertEqual(pages.count(), 2)
          for p in pages:
              self.assertNotEqual(p.name, "test3")

      def test_page_update_is_applied(self):
          """
          Updating a page should persist.
          """
          create_page(name="test1", content="Test page 1")
          p = Page.objects.get(name="test1")
          p.name = "test1updated"
          p.save()
          p = Page.objects.first()
          self.assertEqual(p.name, "test1updated")


  class HomePageViewTests(TestCase):
      def test_index_view_is_rendered_with_home_page_content(self):
          create_page(name="home", content="test home page")
          response = self.client.get(reverse('home'))
          self.assertEqual(response.status_code, 200)
          self.assertContains(response, "test home page")


We've added 5 tests in two classes. Each class implements TestCase which provide the assert functions. It is a good idea to group seperate types of tests into classes, with names that describe their general purpose. In our example we have two groups, one to test database CRUD -- that is Create Retreive Update and Delete -- and the other for testing view responses.

Create a Theme
~~~~~~~~~~~~~~
Finally, we're able to wrap up our first iteration by modifying the look and feel of our site. To do this, we'll create a layout and apply styles using HTML and CSS. We won't get into JavaScript just yet there will be plenty of those articles to come.

For base our theme, we'll use an existing, free-css theme available `here <http://www.free-css.com/free-css-templates/page1/businesstoday#shout>`_.

From the zip file, copy index.html into johnnycage/templates/johnnycage/base2.html. Edit the template as required. Remember to change the template being rendered by the home page view. Next, copy the static files -- the css and images -- into place.

Below is the completed template, after our modifications.

base2.html
++++++++++

.. code:: django

  {% load staticfiles %}
  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
  <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="EN" lang="EN" dir="ltr">
  <head profile="http://gmpg.org/xfn/11">
  {% block title %}<title>{{ page.title|safe }}</title>{% endblock %}
  <meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1" />
  <meta http-equiv="imagetoolbar" content="no" />
  <link rel="stylesheet" href="{% static "johnnycage/styles/layout.css" %}" type="text/css" />
  </head>
  <body id="top">
  <div class="wrapper col1">
    <div id="header">
      <div id="logo">
        <h1><a href="#">{{ page.title|safe }}</a></h1>
        {# A no fuss Q &amp; A site #}
        <p><strong>{{ page.slogan|safe }}</strong></p>
      </div>
      <br class="clear" />
    </div>
  </div>
  <div class="wrapper col2">
    <div id="topbar">
      <div id="topnav">
        <ul>
          <li class="active"><a href="index.html">Home</a></li>
          <li><a href="#">Q &amp; A</a></li>
          <li class="last"><a href="#">Categories</a>
            <ul>
              <li><a href="#">Computer Science</a></li>
              <li><a href="#">History</a></li>
              <li><a href="#">Off Topic</a></li>
            </ul>
          </li>
        </ul>
      </div>
      <div id="search">
        <form action="#" method="post">
          <fieldset>
            <legend>Site Search</legend>
            <input type="text" value="Search Our Website&hellip;"  onfocus="this.value=(this.value=='Search Our Website&hellip;')? '' : this.value ;" />
            <input type="submit" name="go" id="go" value="Search" />
          </fieldset>
        </form>
      </div>
      <br class="clear" />
    </div>
  </div>
  {% block content %}{% endblock %}
  <div class="wrapper col6">
  <div class="wrapper col7">
    <div id="copyright">
      <p class="fl_left">Copyright &copy; 2011 - All Rights Reserved - <a href="#">Adri.Codes</a></p>
      <p class="fl_right">Template by <a href="http://www.os-templates.com/" title="Free Website Templates">OS Templates</a></p>
      <br class="clear" />
    </div>
  </div>
  </body>
  </html>


index.html
++++++++++

.. code:: django

  {% extends "johnnycage/base2.html" %}
  {% load staticfiles %}

  {% block content %}
  <div class="wrapper col3">
    <div id="intro">
      <div class="fl_left">
        <h2>{{ page.subtitle }}</h2>
        <p>{{ page.mainstatement|safe }}</p>
        {# Questions &amp; Answers&raquo; #}
        <p class="readmore"><a href="#">{{ page.mainreadmorebtn|safe }}</a></p>
      </div>
      <div class="fl_right"><img src="{% static "johnnycage/images/demo/380x300.gif" %}" alt="" /></div>
      <br class="clear" />
    </div>
  </div>
  <div class="wrapper col4">
    <div id="services">
    </div>
  </div>
  {% endblock %}

styles/layout.css:138
+++++++++++++++++++++

.. code:: css

  #intro{
    padding:30px 0 25px 0;
    font-size:16px;
    font-family:Georgia, "Times New Roman", Times, serif;
  }
