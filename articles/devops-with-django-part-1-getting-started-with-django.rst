How to make a basic Content Management System with Django
=========================================================

Django is a robust, all-purpose web application development framework, born from the news industry. Django requires a "model-first" approach to development, on which many aspects of the code base are centered. From a developer's perspective, this means writing code which denotes the the data and from there generating SQL schema, scaffolding code, and much more. The latest version, Django-1.7 comes complete with many attractive features including, but not limited to the following:

  - Object Relational Mapping
  - Auto-generated Administration
  - Rich Management Capabilities on the CLI

In this article we will dive right into Django and quickly move through the various stages in the Django development cycle. First we will install the necessary dependencies, including Python and virtualenv. Next we'll introduce virtual environment and setup a project. Then we'll begin modeling data, creating views, configuring the url dispatcher, and implementing unit test. All along we'll explain the Django SDK as necessary to provided just enough context to understand what's happening.

Why Django? Well, honestly for no reason other than I wanted to jump into Python. Although, I've heard the Django deployment is "hard" so it seemed to be a good choice for Dev/Ops research.

So, without further ado, let's get started!

Setup the Developer Environment
-------------------------------
One of the most important tenants of devops is to be flexible. This is core to the Linux community. Quality software tends to easier to port. Software is often developed on one type of Linux and ported to another when it's migrated to Production, or maybe sometime after. It is common for software to be developed on a Debian-based system and then later ported to run on Enterprise Linux. To illustrate this concept -- the need for quality, portable development practices -- we'll develop on Arch Linux, a rolling distro often much more bleeding edge, and run on CentOS 7.

Install System-Level Dependencies
+++++++++++++++++++++++++++++++++

.. code:: bash

  $ sudo pacman -Sy python virtualenv

This will install python3

Verify python installation by checking the version

.. code:: bash

  $ python --version
  Python 3.4.1

And also virtualenv

.. code:: bash

  $ virtualenv --version
  1.11.6

Arch Linux allows both Python2 and Python3 to be installed side-by-side as system Python. By default, we installed Python3 and that's the one that virtualenv will use. If there is any doubt, use virtualenv3.

The Hitchhiker's Guide to Python provides a great into to virtualenv available `here <http://docs.python-guide.org/en/latest/dev/virtualenvs/>`_.

Configure the Virtual Environment
+++++++++++++++++++++++++++++++++
Create directories for the virtual environment and the project source code. We use the virtualenv command to create an environment named "johnnycage". This will be the code name of our CMS. You may want to install the tree command if you prefer a tree view, otherwise, use ls to verify the directory structure.

.. code:: bash

  $ mkdir ~/python_envs ~/sandbox/
  $ pushd ~/python_envs
  $ virtualenv johnnycage
  Using base prefix '/usr'
  New python executable in johnnycage/bin/python3
  Also creating executable in johnnycage/bin/python
  Installing setuptools, pip...done.
  $ tree -L 1 johnnycage/
  johnnycage/
  ├── bin
  ├── include
  └── lib

  3 directories, 0 files

Activate, the new virtualenv, install Django and create our project.

.. code:: bash

  $ . ~/pythonenvs/johnnycage/bin/activate
  $ pip install django

Activating the virtualenv will update your PS1, adding the virtualenv name to it like (johnnycage)user@host. For the purposes of this tutorial, and because by PS1 is highly customized, I've left it out of the code examples.

Beginning Project Development
+++++++++++++++++++++++++++++
Using the django-admin utility, create the initial project.

.. code:: bash

  $ pushd ~/sandbox
  $ django-admin startproject johnnycage

This will generate the base project structure. From here we can begin to plan our app, adding models, views, templates, tests and whatever else as necessary.

