Часть III – Создание домашней страницы используя РЧТ, Статические файлы и настройки Шаблонов
============================================================================================

.. role:: red
.. role:: redbold
.. role:: bolditalic
.. role:: orange
.. |;)| image:: _static/1f609.png
.. |:)| image:: _static/1f642.png

.. |br| raw:: html

   <br />

Once we have our working environment set, we can focus
on **creating the Home page** of our site.

However, this won’t be the usual blank home page with only “hello world” in it.

We will configure both static files and templates,
implement HTML5 Boilerplate and Twitter Bootstrap,
and create an **much better** version of the usual home page
(yeah, it will have hello world in it too…)

Moreover, we will obey the Testing Goat and follow **Test Driven Development**
(TDD) to create the Home Page. You’ll learn a lot here! |;)|

Here’s what we will cover:

* :ref:`Static-Files-Settings`
* :ref:`Templates-Settings`
* :ref:`Initializr-HTML5-Boilerplate-and-Twitter-Bootstrap`
* :ref:`Home-Page-with-TDD-Tests-first`
* :ref:`Home-Page-with-TDD-Code-next`
* :ref:`Commit-again-to-your-local-repository-and-Bitbucket`

.. _Static-Files-Settings:

Static Files Settings
---------------------

Open the common settings file (:red:`settings/base.py`) file and look for the variable
``INSTALLED_APPS``. Make sure it contains the app :redbold:`'django.contrib.staticfiles'`.

Next go at the end of the file and find the line:

.. code-block:: python

    STATIC_URL = '/static/'

This line tells Django to look for :orange:`static` files in
a folder named static inside each of our apps.

However, some static files are used for the whole project and shouldn’t
be inside a specific app. Go inside the :red:`taskbuster` folder, at the
same level of the settings files, and create a directory named :orange:`static`.

.. code-block:: bash

    $ cd taskbuster
    $ mkdir static

This directory will contain all the static files that are global
for the project, like CSS or javascript files.

If you look at the beginning of the :red:`settings/base.py` file, you will find

.. code-block:: python

    BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

which points to the folder containing the folder that contains
the actual file, i.e. the folder :red:`taskbuster`.

.. note::
    if you are working with a single :red:`settings.py` file located
    inside the :red:`taskbuster` folder (without the extra :red:`settings` folder
    we created here) you’ll have to redefine :orange:`BASE_DIR` in order to point
    to the :red:`taskbuster` folder. This is because without the extra settings
    folder in the middle, the previous :orange:`BASE_DIR` would point to the top
    :red:`taskbuster_project` folder, instead of the :red:`taskbuster` folder.
    You should redefine it with

.. code-block:: python

    BASE_DIR = os.path.dirname(os.path.abspath(__file__))

To tell Django to look for static files in the :red:`taskbuster/static`
directory that we just created, write the following
after the :orange:`STATIC_URL` statement:

.. code-block:: python

    STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
    )

Don’t forget the coma at the end! With this configuration, Django will
look for static files in a folder named :red:`static` inside each app
and into the :red:`taskbuster/static` folder we just created.

.. _Templates-Settings:

Templates Settings
------------------

A similar thing happens with templates. By default,
the Django template loader looks for templates in a
folder named :red:`templates` inside each app.

But let’s create a :red:`templates` folder inside :red:`taskbuster` to contain
all the templates that will be used throughout all
the project, like :red:`base.html` or some error page.

.. code-block:: bash

    $ cd taskbuster
    $ mkdir templates

Next, update the settings files and edit the :red:`DIRS` key in
the :red:`TEMPLATE` variable with:

.. code-block:: python

    # Templates files
    TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, "templates")],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                 django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
    ]

Like with the static files, Django will look for :red:`templates` located at
a folder named templates inside each app and inside the
:red:`taskbuster/templates` folder we just created.

.. _Initializr-HTML5-Boilerplate-and-Twitter-Bootstrap:

Initializr: HTML5 Boilerplate and Twitter Bootstrap
---------------------------------------------------

In order to check that the configuration of the templates and
static files work, and because I like developing in Django
with at least some CSS, we are going to include
`HTML5 Boilerplate <http://html5boilerplate.com/>`_
and `Bootstrap <http://getbootstrap.com/>`_.
These tools help you create responsive templates that work in many browsers.

Here, we will use `Initializr <http://www.initializr.com/>`_,
a mixed version that combines
both HTML5 Boilerplate and Bootstrap. Go to its website,
click on Get Custom Build, and select your project’s priorities.
The configuration I selected for this project is the one shown
on the image (except for the Apple Touch Icons,
that are not used in this project):

.. figure:: _static/taskbuster_part2_initializr-1024x868.png
       :alt: initializr
       :align: center

Once you download the package, unzip it and reorganize its contents:

* Move the files :red:`index.html`, :red:`404.html`, :red:`humans.txt` and :red:`robots.txt` into the :red:`taskbuster/templates` folder.
* Change the name of :red:`index.html` to :red:`base.html`. The index file is usually the template of your Home page, but we will use it as a base template — all our site templates will inherit from this base template.
* Move the other files and folders into the :red:`taskbuster/static` folder
* If you have an icon you would like to use for your app, replace it for the :red:`favicon.ico` file (I recommend you use the same name).
* I also removed the files :red:`apple-touch-icon.png`, :red:`browserconfig.xml`, :red:`tile-wide.png` and :red:`tile.png`.

.. _Home-Page-with-TDD-Tests-first:

Home Page with TDD – Tests first
--------------------------------

In order to see if the static files and templates are loading
correctly, we will need a test. You know… Obey
the testing Goat! :bolditalic:`Test first, test first!`

.. figure:: _static/obey_the_testing_goat-300x290.jpg
       :alt: testing goat
       :align: center

       Сначала тест, Сначала тест!

Actually, to work with Test Driven Development (TDD) we should have written
the test even before setting the templates and static
folders. But I wanted to finish editing the settings files first.

First, we will convert the :red:`functional_tests` folder into a package
by including an empty file named :red:`__init__.py` inside.

.. code-block:: bash

    touch functional_tests/__init__.py

This way, we can run our functional tests with:

.. code-block:: bash

    $ python manage.py test functional_tests

However, the test runner only finds files that
start with test, so let’s rename :red:`all_users.py` to :red:`test_all_users.py`.

We will let git to manage this, so that the repository detects the movement correctly:

.. code-block:: bash

    $ git mv functional_tests/all_users.py functional_tests/test_all_users.py

Run a development server using the :red:`tb_dev` environment, and then run
the functional tests in your :red:`tb_test` environment.
It should work as before, nothing broken!

But I’m sure you don’t like this game with the two environments, right?
Having to run the server using tb_dev and testing with :red:`tb_test`.
**Why can’t the test itself create the server?**

**Moreover, changes made by these functional tests are persistent.**
Imagine that during one test, we create an instance of a model
(for example, a new user). We want that after running the test,
that instance (the user) disappears from our database, right?
But with functional tests, we are just running the development server
and playing around with the development database, so these changes
persist even after the test is finished.

But don’t worry, there is the :orange:`LiveServerTestCase` to make
our live much easier! |;)|

As we will see, instances of this class create a server with
a **testing database**, like when we run **unittests**.

Now, let’s edit the file :red:`functional_tests/test_all_users.py` and test
that both templates and static folders work as expected.
For example, we can test these two different things.

* The title in the home page is “TaskBuster”
* The text color of the h1 header in the home page is :red:`rgba(200, 50, 255, 1)` ~ pink color.

So let’s create this test!|br|
Note: The file :red:`test_all_users.py` contains
the ``NewVisitorTest`` created in Part I. You should replace that test by this one:

.. code-block:: python
    :linenos:

    # -*- coding: utf-8 -*-
    from selenium import webdriver
    from django.core.urlresolvers import reverse
    from django.contrib.staticfiles.testing import LiveServerTestCase


    class HomeNewVisitorTest(LiveServerTestCase):

        def setUp(self):
            self.browser = webdriver.Firefox()
            self.browser.implicitly_wait(3)

        def tearDown(self):
            self.browser.quit()

        def get_full_url(self, namespace):
            return self.live_server_url + reverse(namespace)

        def test_home_title(self):
            self.browser.get(self.get_full_url("home"))
            self.assertIn("TaskBuster", self.browser.title)

        def test_h1_css(self):
            self.browser.get(self.get_full_url("home"))
            h1 = self.browser.find_element_by_tag_name("h1")
            self.assertEqual(h1.value_of_css_property("color"),
                             "rgba(200, 50, 255, 1)")

Let’s go through that code step by step:

* First, we define an auxiliar function named ``get_full_url`` that takes one argument, the :orange:`namespace`
    * A :orange:`namespace` is an identifier for a url. It’s a nice thing about Django: when you work with identifiers, you can change the url to whatever you want that the *code just works* as before.
    * ``self.live_server_url`` gives you the local host url. We use this method because the test server uses another url (usually ``http://127.0.0.1:8021``), and this method is more flexible.
    * reverse gives you the **relative** url of a given namespace, here ``/``
    * The resulting function gives you the absolute url of that namespace (the sum of the previous two), ``http://127.0.0.1:8021/``.
* The ``test_home_title`` method tests that the home page title contains the word :red:`TaskBuster`. We will create a template for that, so if the title exists it means that the template has been loaded correctly.
* The ``test_h1_css`` method tests that the h1 text has the desired color. The css rule for the text color will be on a CSS file, which means that if the test passes, staticfiles are loading correctly.
* Finally, note that we have removed the ``if __name__ == '__main__'`` statement, as :red:`functional_tests` is now a package that will run with the Django test runner.

Once we have our test created, TDD tells us to follow the cycle:

* run the test and see it fail
* write some code so that it corrects the test error message (only write the code that corrects the error message shown by the test failure, don’t anticipate other possible errors).

We have to follow this cycle until the full test passes.
It will be more clear in the next section.

.. _Home-Page-with-TDD-Code-next:

Home Page with TDD – Code next
------------------------------

Now that we have the functional test for our home page,
we can run it and see how it fails. In our :red:`tb_test` environment:

.. code-block:: bash

    $ python manage.py test functional_tests

We see that the first error it founds is that the namespace :redbold:`"home"`
is not defined. Open the file :red:`taskbuster/urls.py` and import
the view ``home`` from :red:`views.py` (depending on your Django version,
you might need to create this file first: :red:`taskbuster/views.py`):

.. code-block:: python

    from .views import home

Note that we are using a relative import to import the ``home`` view.
This way, we can change the name of our project or app without breaking the urls.

Next, add the following url:

.. code-block:: python

    urlpatterns = [
    ...
    url(r'^$', home, name='home'),
    ...
    ]

If you run the test again it will fail, as we don’t have any home view defined.
Let’s define a silly one, create the file :red:`taskbuster/views.py` and define the view:

.. code-block:: python

    def home(request):
         return ""

which gives a test failure because the the home page
doesn’t have TaskBuster in its title.

Let’s focus on our templates now: open the file :red:`taskbuster/templates/base.html`
file with a browser to have an idea of what’s in it. Quite awful right?
That’s because the static files are not loading.

This :red:`base.html` will be our base template, and all the other project
templates will inherit from this one, including the Home page.

So, it’s time for a unittest. Yeah, I know you just want to code
the home page template but… :redbold:`Obey the Testing Goat!` |;)|

Unittests are meant to test small pieces of code, from the point
of view of the developer. For example, it’s clear that the user
doesn’t care if the home page template inherits from another
template as long as it displays the contents. But the developer
cares, and that’s why we should write a unittest. Moreover, I
realized that when I have to think about the test, I write cleaner code.
I guess it’s because having to define tests makes you think about what
**exactly** you want the code to do. And that clears your mind |;)|

Create a :red:`test.py` file inside the :red:`taskbuster` folder
and write the following test:

.. code-block:: python
    :linenos:

    # -*- coding: utf-8 -*-
    from django.test import TestCase
    from django.core.urlresolvers import reverse


    class TestHomePage(TestCase):

        def test_uses_index_template(self):
            response = self.client.get(reverse("home"))
            self.assertTemplateUsed(response, "taskbuster/index.html")

        def test_uses_base_template(self):
            response = self.client.get(reverse("home"))
            self.assertTemplateUsed(response, "base.html")

You can run these tests using

.. code-block:: bash

    $ python manage.py test taskbuster.test

Obviously, they will fail… First, let’s create the
:red:`taskbuster/index.html` template:

.. code-block:: bash

    $ cd taskbuster/templates
    $ mkdir taskbuster
    $ touch taskbuster/index.html

and edit the :red:`taskbuster/views.py` so that it has:

.. code-block:: python

    # -*- coding: utf-8 -*-
    from django.shortcuts import render

    def home(request):
        return render(request, "taskbuster/index.html", {})

where we have used the shortcut render, which lets you load
a template, create a context adding a bunch of variables by default,
such as information about the current logged-in user, or the current
language, render it and return an :red:`HttpResponse`, all in one function.
Note: the information added by default depends on the template context
processors that you have included in your settings file.

If you run again the unittests, you’ll see that the first one passes,
indicating that the home page uses the :red:`taskbuster/index.html` template.
We just need to make this template to inherit from the :red:`base.html` template.

So let’s edit the :red:`base.html` template. For now, we are only interested in the
**title tag** inside the head tag. Look for it in
the file and write the following inside:

.. code-block:: html

    <head>
    ...
    <title>{% block head_title %}{% endblock %}</title>
    ...
    </head>

These two template tags, ``{% block head_title %}`` and ``{% endblock %}`` mark
the beginning and the end of a content block, which contents can be replaced
by child templates. You will see it clearly in a minute.

Edit again the :red:`index.html` file, make it inherit from the :red:`base.html`
file and add it a title:

.. code-block:: html

    {% extends "base.html" %}
    {% block head_title %}TaskBuster Django Tutorial{% endblock %}

The idea is that :red:`index.html` inherits from :red:`base.html` (it uses all its contents)
except for the blocks marked with these special template tags. In that case,
it substitutes the content inside the template tag of :red:`index.html` into
the corresponding block in :red:`base.html`.

Let’s run again the unittests:

.. code-block:: bash

    $ python manage.py test taskbuster.test

Ok…!! Perfect, they passed!

What about the functional tests?

.. code-block:: bash

    $ python manage.py test functional_tests

One passed, one failed. That’s something!
But **we still have to take care about static files!**

First, let’s define our custom CSS file by editing
the file :red:`taskbuster/static/css/main.css` and adding:

.. code-block:: css

    .jumbotron h1 {
    color: rgba(200, 50, 255, 1);
    }

Then, edit again the :red:`base.html` and add this at the beginning
of the file (even before the ``<!DOCTIPE html>`` statement):

.. code-block:: html

    {% load staticfiles %}

then, look for all the links to static files and scripts of javascript like

.. code-block:: html

    <link rel="stylesheet" href="css/xxx.css">
    <script src="js/xxx.js"></script>

and change them into this:

.. code-block:: html

    <link rel="stylesheet" href="{% static 'css/xxx.css' %}">
    <script src="{% static 'js/xxx.js' %}">

(you might want to remove the apple-touch-icon.png link).

Be careful with the ``""`` and ``''`` . Moreover,
at the end of the file you will see something like

.. code-block:: html

    <script>window.jQuery || document.write('<script src="js/vendor/jquery-1.11.0.min.js"><\/script>')</script>

We cannot add the ``"{% static 'xxx' %}"`` tag because we would break the string.
In this case, you can include the static file specifying the relative path:

.. code-block:: html

    <script>window.jQuery || document.write('<script src="static/js/vendor/jquery-1.11.0.min.js"><\/script>')</script>

.. note::
    although both methods for importing static files work, it’s better to use
    the template tag if you are planning to use a content delivery network (CDN) for serving static files.

Ok, let’s run the functional test again! Oh nooo! I got
an ugly error that wasn’t there before!

This is because the :orange:`LiveServerTestCase` doesn’t support static files…

But don’t worry, as usual, Django has a solution for it! I has another
test class that supports static files!

To make the changes, edit the :red:`functional_tests/test_all_users.py`,
remove the lines with a minus sign and add the ones with a plus:

.. code-block:: python

    - from django.test import LiveServerTestCase
    + from django.contrib.staticfiles.testing import StaticLiveServerTestCase

    - class HomeNewVisitorTest(LiveServerTestCase):
    + class HomeNewVisitorTest(StaticLiveServerTestCase):

Run your tests again and yes! They passed! |:)|

If you want to run both the unittests and functional tests
together, you can use

.. code-block:: bash

    $ python manage.py test

You can also take a look at the localhost and see how pretty
the home page is now that the CSS files are loading correctly |;)|

… probably not the best choice for the h1 color text, though!

.. note::
    if when running the functional tests you get an error with Selenium
    that says *The browser appears to have exited before we could connect*,
    try to upgrade selenium in your working environment:

.. code-block:: bash

    $ pip install -U selenium

It should fix your problem |;)|

.. _Commit-again-to-your-local-repository-and-Bitbucket:

Commit again to your local repository and Bitbucket
---------------------------------------------------

It’s a good time to make another commit!

.. code-block:: bash

    $ git add .
    $ git status

make sure you add only the files you want to commit.
Moreover, at the beginning of the output it says

.. code-block:: bash

    Your branch is up-to-date with 'origin/master'

which means that the actual master branch is at the same state as the origin
branch in Bitbucket. Let’s see what happens after we commit the new changes:

.. code-block:: bash

    $ git commit -m "Settings, static files and templates"

And let’s check the status again…

.. code-block:: bash

    $ git status
    On branch master
    Your branch is ahead of 'origin/master' by 1 commit.
    (use "git push" to publish your local commits)

    nothing to commit, working directory clean

So the local master branch is one commit ahead of the origin master branch.
Let’s fix that by pushing the recent commit into the Bitbucket repository:

.. code-block:: bash

    $ git push origin master

So now, our branch is again up-to-date with the origin/master in Bitbucket.

That’s all for now, we have created a nice Home Page!

In the next part of this TaskBuster Django Tutorial, we will talk about
configuring other files download with the Initializr package:
:red:`robots.txt`, :red:`humans.txt` and the :red:`favicon.ico` image.

Moreover, I’ll show you how to use coverage, a
tool to measure how many code is covered by tests.

Check the next part: :doc:`Website files and Testing with coverage </part_4>`

And remember to share it with your friends! |:)|
