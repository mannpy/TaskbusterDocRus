Часть II – Файлы настроек и Управление Версиями
===============================================

.. role:: red
.. role:: redbold
.. role:: bolditalic
.. role:: orange
.. |смайл| image:: _static/1f609.png

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

    Django==1.8.13
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
    $ echo "Django==1.8.5" >> base.txt

If you have another version, write yours instead!

Now let’s make the three other files to inherit
the packages of the :red:`common.txt` file.

.. code-block:: bash

    $ echo "-r base.txt" | tee -a development.txt testing.txt production.txt

.. _Different-settings.py:

Different settings.py for each enviroment
-----------------------------------------

.. _Production-settings.py-Debug-False:

Production settings.py – Debug False
------------------------------------

.. _Django-security-and-the-Secret-Key:

Django security and the Secret Key
----------------------------------

.. _Initialize-a-Git-repository-and-Commit:

Initialize a Git repository and Commit
--------------------------------------

.. _Upload-your-project-into-Bitbucket:

Upload your project into Bitbucket
----------------------------------
