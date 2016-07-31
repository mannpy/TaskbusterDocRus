Часть II – Файлы настроек и Управление Версиями
===============================================

.. role:: red
.. role:: redbold
.. role:: bolditalic
.. role:: orange
.. |смайл| image:: _static/1f609.png
.. |smile| image:: _static/1f642.png

In :doc:`Part I </part_1>` of this tutorial we built the working environment
and created the :redbold:`TaskBuster` project.

Now, we will configure the different environments for testing, developing and
production, editing different Django settings files.

Moreover, we will remove Django’s :orange:`SECRET_KEY` from these files
in order to **keep it Secret**.

Next, we will create a new repository to keep our code
in version control and uploaded it into Bitbucket.

Ready for the next part? Here’s the guideline:

* :ref:`virtual-Environments`
* :ref:`Different-settings.py`
* :ref:`Production-settings.py-Debug-False`
* :ref:`Django-security-and-the-Secret-Key`
* :ref:`Initialize-a-Git-repository-and-Commit`
* :ref:`Upload-your-project-into-Bitbucket`

.. |br| raw:: html

   <br />

.. _virtual-Environments:

Virtual Environments and Requirements files
-------------------------------------------

One important thing when working on a Project, is to control the version
of your packages. For example, imagine that you’re developing in one computer
that has Django 1.8 installed, and you are deploying in one server that has an
older version of Django, let’s say 1.6. Your code works fine locally, but when
deploying it, some incompatible errors may occur. And the same could happen if
more than one developer is working in the same project, each of them with its
own package versions installed.

.. figure:: _static/taskbuster_virtual_environments-297x300.jpg
       :alt: taskbuster_virtual_environments
       :align: center

       Use Virtual Environments to install custom packages,
       and make Django look like a Cobra!

The standard solution to this problem is to unify all the packages and save the
versions used in a file named :red:`requirements.txt`. This file will contain
something like:

.. code-block:: bash

    Django==1.8.14
    selenium==2.48.0

which are the packages we have installed so far (don’t worry if the
versions of your environment are different).

You can see the packages installed in a virtual environment by typing

.. code-block:: bash

    $ pip freeze

You might see another package, :red:`wheel`, that is installed by default in
some versions (don’t worry if you don’t have it). Therefore, you can create
automatically the :red:`requirements.txt` file by saving the output of
the previous command into the file:


.. code-block:: bash

    $ pip freeze > requirements.txt

However, as you may notice, :red:`Selenium` is only needed for the testing environment,
so there is no need that the developing or the production environments
have this package installed.

Let’s work out this issue by creating a requirements folder and creating a file
for each environment. Go inside the :red:`taskbuster_project` folder and type:

.. code-block:: bash

    $ mkdir requirements
    $ touch requirements/{base.txt,development.txt,production.txt,testing.txt}

Note: don’t add any extra spaces in the previous command or it won’t
work |смайл| And you can delete the previous :red:`requirements.txt` file,
as we won’t need it. Moreover, you can also define a :red:`staging.txt` file
if you are planning to run a semi-private version of
your site on a production server.

Let’s edit first the file :red:`base.txt`. This file will contain all
the packages that are common for all the environments. Now, it should only
contain the Django version:

.. code-block:: bash

    $ cd requirements
    $ echo "Django==1.8.14" >> base.txt

If you have another version, write yours instead!

Now let’s make the three other files to inherit
the packages of the :red:`common.txt` file.

.. code-block:: bash

    $ echo "-r base.txt" | tee -a development.txt testing.txt production.txt

Finally, we need to add Selenium into the testing file:

.. code-block:: bash

    $ echo "selenium==2.48.0" >> testing.txt

again, write your version here.

Ok, we have everything ready now. When a new programmer joins our team, we
will tell him to create two different environments, one for testing and
another for developing (the production environment is for deployment).

Next, he only needs to activate each of these environments and install
the packages saved in each requirements files:

.. code-block:: bash

    $ workon tb_dev
    $ pip install -r requirements/development.txt

    $ workon tb_test
    $ pip install -r requirements/testing.txt

.. _Different-settings.py:

Different settings.py for each enviroment
-----------------------------------------

Each environment defined previously has a different purpose, and therefore,
they will need different configurations. For example, the database
configuration for production and developing might be different, or the
testing environment might use some Django apps that are not needed
in the other environments (like :red:`selenium`).

That’s why we will specify different setting files for each environment.
First, we will create a folder to contain our setting files,
inside the :red:`taskbuster` folder:

.. code-block:: bash

    $ mkdir taskbuster/settings

This folder will contain:

* :red:`__init__.py` file to make this folder a Python package
* :red:`base.py` will contain all the settings that are common in all environments. The other setting files inherit from this one.
* :red:`development.py` is for local development.
* :red:`testing.py` is for testing.
* :red:`production.py` will be used in the production environment.
* :red:`staging.py` if you want to run a staging version on the production server of your project.

Let’s create these files, all inside the :red:`taskbuster/settings` folder:

.. code-block:: bash

    $ cd taskbuster/settings
    $ touch __init__.py development.py testing.py production.py staging.py

And edit each of them (:red:`development.py`, :red:`testing.py`,
:red:`production.py` and :red:`staging.py`) to make them inherit from
the :red:`base.py` file — we’ll create this file in a second :-):

.. code-block:: python

    # -*- coding: utf-8 -*-
    from .base import *

And finally, we have to move and rename the :red:`settings.py` file created
by Django, to be our :red:`base.py` file inside the settings folder.
Working inside the settings folder, you have to type:

.. code-block:: bash

    $ mv ../settings.py base.py

After creating these files, we need to specify the virtual
environment to work with the correct setting file.

With :red:`virtualenvwrapper`, there is a way to configure
different hooks that are sourced before or after activating
the virtual environment, and before or after deactivating it.
This means that we can define a set of statements that will be run
at different stages of the virtual environment lifecycle. These hooks
are saved inside the :red:`bin` folder inside the virtual environment folder,
and their names are :red:`preactivate`, :red:`postactivate`,
:red:`predeactivate` and :red:`postdeactivate`.

In our case, we will set a :red:`postactivate` script that will set
the ``DJANGO_SETTINGS_MODULE``  variable just after activating the virtual
environment, and a :red:`predeactivate` that will clean it up before deactivating it.

.. code-block:: bash

    $ workon tb_dev
    $ cd $VIRTUAL_ENV/bin

The last command will take you to the virtual environment
folder, where the different hooks reside.
Edit the :red:`postactivate` file by adding:

.. code-block:: bash

    export DJANGO_SETTINGS_MODULE="taskbuster.settings.development"

and edit the :red:`predeactivate` file by adding:

.. code-block:: bash

    unset DJANGO_SETTINGS_MODULE

Do the same with the testing environment, with the only change:

.. code-block:: bash

    export DJANGO_SETTINGS_MODULE="taskbuster.settings.testing"

It’s time to check! Go back to the :red:`taskbuster_project` folder
and activate your development environment. Next, run:

.. code-block:: bash

    $ python manage.py runserver

and in the output displayed you should see a line
indicating that you are using the
:red:`taskbuster.development_settings` file, something like:

.. code-block:: bash

    Django version 1.8, using settings 'taskbuster.settings.development'

Open another tab (and leave the previous server active), and activate
the testing environment. Check that it’s using the
:red:`taskbuster.testing_settings` file. Probably it will complain
saying that that port is already in use by the development
environment, so specify another port with

.. code-block:: bash

    $ python manage.py runserver 127.0.0.1:8001

Next, quit the server in the testing environment and run the functional test:

.. code-block:: bash

    $ python functional_tests/all_users.py

Yes! We didn’t break anything |smile|

.. _Production-settings.py-Debug-False:

Production settings.py – Debug False
------------------------------------

One **important** thing to remember is to set the variable
``DEBUG`` to ``False``  in your production settings files.

Note: Previous to Django 1.8 you also need to set
``TEMPLATE_DEBUG`` to false. However, with the introduction of the new
``TEMPLATE`` setting, the old ``TEMPLATE_DEBUG`` is set automatically
equal to ``DEBUG``. If you really want to specify a value, check
`the official documentation <https://docs.djangoproject.com/en/1.8/ref/settings/#template-debug>`_.

First, cut the DEBUG variable from the :red:`base.py`
settings file and copy it into the :red:`development.py` and
:red:`testing.py` settings files:

.. code-block:: python

    DEBUG = True

Next, add it on the :red:`production.py` setting file and make it :orange:`False`:

.. code-block:: python

    DEBUG = False

This way, each environment will have the correct value
of this variable. If you’ve also defined a :red:`staging.py`
file, copy it there too.

.. _Django-security-and-the-Secret-Key:

Django security and the Secret Key
----------------------------------

If you open the file :red:`taskbuster/settings/base.py`
you will see a variable named ``SECRET_KEY``. This variable
**should be kept secret**, and therefore **out of version control**.

One option would be to add the :red:`base.py` file into the :red:`.gitignore`
file, that is, remove it from the version control. However, during
project development this file suffers many changes, and it’s quite
useful to have it in version control, specially if you want to share
it with your coworkers. Therefore, a better approach is to remove the
secret key variable and import it from somewhere else. And this somewhere
else is the one that should remain out of version control.

The approach we will follow here is to put the secret key inside our
virtual environment configuration, and get the key from the environment
by importing it in the :red:`base.py` file.

Note: If you’re using Apache this method won’t work. The best option is that
you save your ``SECRET_KEY`` in some file, and import it into the :red:`base.py` file.
The key file should be removed from version control by including it
in the :red:`.gitignore` file. I recommend you read the
`Two Scoops of Django 1.6 <http://www.amazon.com/Two-Scoops-Django-Best-Practices/dp/098146730X>`_
book, section 5.4. (You might want to check they new
version, updated for Django 1.8!)

To include the secret key inside the virtual environment
we will also work with the virtualenvwrapper’s
:red:`postactivate` and a :red:`predeactivate` hooks.

Activate your tb_dev environment and go to its :red:`bin`
folder using the shortcut

.. code-block:: bash

    $ cd $VIRTUAL_ENV/bin

If you type :orange:`ls` you will see that it contains the files
we just described. Edit the :red:`postactivate` file and add the secret key line

.. code-block:: bash

    export SECRET_KEY="your_secret_django_key"

Note: don’t put any spaces around the = sign.

Next edit the :red:`predeactivate` file and add the line:

.. code-block:: bash

    unset SECRET_KEY

This way, if you type:

.. code-block:: bash

    $ workon tb_dev
    $ echo $SECRET_KEY
    your_secret_django_key
    $ deactivate
    $ echo $SECRET_KEY

Where the last line indicates that there is no output.
This means that the variable ``SECRET_KEY``  is only visible
when working in this virtual environment, as we wanted.

Repeat the same process for the :red:`tb_test` virtual environment.

Next, deactivate and activate each environment to make these changes effective.

And finally, edit the :red:`base.py` file, remove the ``SECRET_KEY``  and add the following lines:

.. code-block:: python

    from django.core.exceptions import ImproperlyConfigured

    def get_env_variable(var_name):
        try:
            return os.environ[var_name]
        except KeyError:
            error_msg = "Set the %s environment variable" % var_name
            raise ImproperlyConfigured(error_msg)

    SECRET_KEY = get_env_variable('SECRET_KEY')

The function ``get_env_variable``  tries to get the variable ``var_name``
from the environment, and if it doesn’t find it, it raises an
``ImproperlyConfigured``  error. This way, when you try to run your
app and the ``SECRET_KEY`` variable is not found, we will be able
to see a message indicating why our project fails.

Let’s check that it all works as expected. Save the :red:`base.py`,
deactivate both environments and activate them again, in
different terminal tabs.

Run the development server in the :red:`tb_dev` environment

.. code-block:: bash

    $ python manage.py runserver

and run the functional test in the :red:`tb_test` environment

.. code-block:: bash

    $ python functional_tests/all_users.py

Hope the test also works for you!! |smile|

Note: When deploying your app, you will have to specify the ``SECRET_KEY``
in your server. For example, if you are using :red:`Heroku`, you can use:

.. code-block:: bash

    $ heroku config:set SECRET_KEY="your_secret_key"

But don’t worry, we’ll cover Heroku latter in this tutorial!! |smile|

.. _Initialize-a-Git-repository-and-Commit:

Initialize a Git repository and Commit
--------------------------------------

Ok! now we are ready to commit our project into a new repository!
Note: you can read a
`basic git tutorial here <http://www.marinamele.com/2014/07/git-tutorial-create-a-repository-commit-git-branches-and-bitbucket.html>`_.

Go into the :red:`taskbuster_project` directory and type:

.. code-block:: bash

    $ git init .

to initialize the repository in the current folder.
You will see a new folder named :red:`.git`, containing the new repository.

Before adding files into the repository, we have to think if there
are files that we want to keep away from version control.

Note that after running the development server, we have the extra files:

* :red:`db.sqlite3` – a database
* :red:`__pycache__` –  a folder containing all the `*.pyc` files.

These two should be removed from version control.
Create a :red:`.gitignore` file inside the :red:`taskbuster_project` foler and write:

.. code-block:: text

    db.sqlite3
    __pycache__
    TaskBuster.sublime-workspace

where we have also included the sublime text workspace (as we saw
in part I, sublime generates two different files when creating
a project. We want the one ending with -project to be on version
control but not the one ending with -workspace). |br|
Next, let’s add all the files of the current directory into the staging area
(except the ones in the :red:`.gitignore` file)

.. code-block:: bash

    $ git add .

And check the files added into the staging area with:

.. code-block:: bash

    $ git status

You should see something like this, with all the new files added in green:

.. figure:: _static/git_changes_to_add.png
       :alt: git_changes_to_add
       :scale: 50 %
       :align: center

If you see some file that you don’t want to commit, you can remove it using

.. code-block:: bash

    $ git rm --cached path_of_file

don’t forget to add it into the :red:`.gitignore` file for subsequent commits.

Finally, let’s commit our changes:

.. code-block:: bash

    $ git commit -m "Taskbuster project created"

The :orange:`-m` flag indicates that the following text will be used
to describe this commit. If you simply type :orange:`git commit`,
an editor will open to write the description of the commit
(by default, this editor is VI).

You can see the commit made with

.. code-block:: bash

    $ git log

.. _Upload-your-project-into-Bitbucket:

Upload your project into Bitbucket
----------------------------------

Create a Bitbucket account if you don’t have one, and create
a **new empty repository**. We use Bitbucket because it allows for
a private repository, but the steps described here will work
almost equal with GitHub.

You will have to determine the URL of this repository,
which will be something like

:red:`https://user_name@bitbucket.org/user_name/repository_name.git`

You can find it in the :red:`Overview` –> :red:`Command line` –>
:red:`I have an existing project.`

First we need to add Bitbucket as a remote repository.
Go to the :red:`taskbuster_project` folder and type:

.. code-block:: bash

    $ git remote add origin https://user_name@bitbucket.org/user_name/repository_name.git

where you should change the url with your repository url.
Note: The previous command is a single line. This will create the
alias origin to refer to your Bitbucket repository (using origin
as an alias for a remote repository is a common convention).

Next, let’s push our existing repository into this
new Bitbucket repository with:

.. code-block:: bash

    $ git push -u origin --all

where the :orange:`–all` flag makes all the refs under refs/heads
to be pushed, and the :orange:`-u` flag stands for :orange:`–set-upstream`
(add a tracking reference). You will have to enter your password.

At the end, you will see the message:

.. code-block:: bash

    Branch master set up to track remote branch master from origin.

You can see your active branches with

.. code-block:: bash

    $ git branch -a
    * master
    remotes/origin/master

Okey! Now that we have our first project with a good working environment,
and in version control, we can work on our Home Page!

But… I’m not talking about creating models yet, just configure
the static files and templates to have a nice Home Page with some CSS
— I hate developing without a basic CSS, so it’s one of the first
things I usually include.

Find out this and more in the next part of the tutorial!
:doc:`Create a Home Page with TDD, Staticfiles and Templates settings </part_3>`

Please, share this Tutorial with your developer friends! |смайл|
