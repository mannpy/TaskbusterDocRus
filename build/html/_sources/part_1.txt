Часть I – Рабочая среда и начало проекта Django
===============================================
Создание хорошей рабочей среды очень важно для вашей производительности.
Может быть, это займет некоторое время, для того чтобы настроить ее,
но когда это хорошо сделано - то стоит потраченного времени!

Например, виртуальная среда поможет вам держать ваши пакеты организованными и контролируемыми,
а хороший текстовый редактор поможет вам с именами переменных, методами пакетов, подсветкой синтаксиса.

Да, и если вы не читали введение, этот Django учебник в актуальном состоянии.

Мы будем использовать Django 1.8 и Python 3. Это не означает, что вы не можете
следовать учебнику, если вы используете старую версию.
Это означает только то, что вам придется самим находить и устранять, возникающие ошибки,
чтобы решить проблемы, которые я не могу здесь предвидеть.


.. role:: red
.. role:: redbold
.. role:: bolditalic
.. role:: orange
.. |смайл| image:: _static/1f609.png
.. |smile| image:: _static/1f642.png

Создание вашей рабочей среды
----------------------------

Для того, чтобы следовать этому Django учебнику вам понадобится :red:`Python3`,
менеджер пакетов :red:`pip3` и следующие пакеты: :red:`virtualenv` и :red:`virtualenvwrapper`.

.. figure:: _static/taskbuster_toolbox-300x296.jpg
       :alt: python
       :align: center

       Комплект инструментов TaskBuster: Python3, pip3,
       virtualenv, virtualenvwrapper, Git и Sublime Text

Если вы используете Mac OS X вы можете посмотреть этот пост:
`Установить Python 3 на Mac OS X и использовать virtualenv и virtualenvwrapper <http://www.marinamele.com/2014/07/install-python3-on-mac-os-x-and-use-virtualenv-and-virtualenvwrapper.html/>`_

Вам также понадобится хороший текстовый редактор для управления всеми файлами
Django, которые мы будем создавать.
Я рекомендую Sublime Text, но если вы чувствуете себя комфортно с другим
текстовым редактором - то вперед!
Смотрите здесь:
`как установить и настроить Sublime Text 3 для Django. <http://www.marinamele.com/2014/03/install-and-configure-sublime-text-3.html/>`_

Несмотря на то, что мы будем говорить об этом в следующем посте, нам будет нужен Git,
система управления пакетами.
Если более чем один пользователь будет управлять кодом, то я рекомендую вам использовать
Git репозиторий, такие как `GitHub`_ или `Bitbucket`_ (последний из которых предлагает
частные репозитории для бесплатных аккаунтов). Вы можете узнать основы о системе Git по этой ссылке
`Git Tutorial.`_

.. _GitHub: https://github.com/
.. _Bitbucket: https://bitbucket.org/
.. _Git Tutorial.: http://www.marinamele.com/2014/07/git-tutorial-create-a-repository-commit-git-branches-and-bitbucket.html

Хорошо, теперь мы готовы начать наш супер-крутой проект Django |смайл|

Установка Django 1.8
--------------------

Во-первых, мы создадим виртуальную среду с :red:`Python3`, по умолчанию как Python.
В вашем терминале, введите:

.. code-block:: bash

    $ which python3

чтобы узнать ваш путь к :red:`python3`. В моем случае это было ``/usr/local/bin/python3``. Далее,
создайте виртуальную среду для разработки, и укажите Python3 как Python по умолчанию
(напомню, что с virtualenvwrapper виртуальные среды создаются в папке
:red:`~/.virtualenvs`, независимо от рабочей папки):

.. code-block:: bash

    $ mkvirtualenv --python=/usr/local/bin/python3 tb_dev

Измените свой путь к Python, если он отличается от моего, а также измените имя
виртуальной среды :red:`tb_dev` на другой, если хотите.
Вы увидите, что эта виртуальная среда теперь активна - командная оболочка
будет выглядить как то так: ``(tb_dev)[user@host]$``.

Вы можете отключить виртуальную среду написав:

.. code-block:: bash

    $ deactivate

и активировать заново так:

.. code-block:: bash

    $ workon tb_dev

Обратите внимание, что если вы работаете в этой среде, когда вы вводите :red:`python`
вы активируете :red:`python3`, и при вызове :red:`pip`, вы вызываете версию python3,
:red:`pip3`. Круто!

Готовы установить последнюю версию Django?! С активным tb_dev, введите:

.. code-block:: bash

    $ pip install Django==1.8.13

Примечание: Вы можете проверить, есть ли новая версия, доступная `здесь <https://www.djangoproject.com/download//>`_.

Рабочий каталог и Sublime Text
----------------------------------

Создайте новую папку на вашем компьютере, названную :red:`taskbuster_project`:

.. code-block:: bash

    $ mkdir taskbuster_project

Эта папка будет главной папкой, содержащей Ваш проект Django, всю его
документацию, файлы развертывания, файлы управления версиями,
файлы sublime проекта, функциональные тесты и т.д.

Давайте создадим Sublime Text проект для управления всеми файлами внутри этой
папки (или с помощью своего редактора). Здесь, я буду считать, что вы активировали
`subl command`_ для открытия файлов с помощью терминала:

.. _subl command: http://www.marinamele.com/2014/03/install-and-configure-sublime-text-3.html

.. code-block:: bash

    $ subl taskbuster_project

Эта команда откроет все файлы в папке :red:`tastbuster_project`.
Затем перейдите в главное меню и выберите :red:`Project / Save Project As`
(Проект / Сохранить Проект Как), имя вашего проекта как
:red:`TaskBuster.sublime-project`, и сохраните файл в той же папке
:red:`taskbuster_project` (по умолчанию это главная папка).

Теперь, вы будете видеть два различных файла в папке :red:`taskbuster_project`:
:red:`TaskBuster.sublime-project` и :red:`TaskBuster.sublime-workspace`
(Примечание: последний не отображается в редакторе sublime, но он есть в терминале).

Повинуйтесь козе тестирования
-----------------------------

I am reading an incredible book:
`Test-Driven development with Python <http://www.obeythetestinggoat.com/>`_,
by Harry J.W. Percival. And it says that you must obey the :redbold:`Testing Goat` — a
little voice in your head that tells you to write a test before writing any
code. :bolditalic:`Test first, test first!`

.. figure:: _static/obey_the_testing_goat-300x290.jpg
       :alt: testing goat
       :align: center

       Obey the testing Goat
       Test first, test first!

And this is what we will do here, before creating any Django project…

We know that when we create successfully a Django project, we get the usual
:bolditalic:`It worked!` Django blue page when we go to the localhost :red:`http://127.0.0.1:8000`.
If we inspect this page, we will see that in its head section, the title tag
is ``<title>Welcome to Django</title>`` I know, you can’t see that because you
haven’t created any project yet! You will have to trust me for now |смайл|

.. figure:: _static/taskbuster_part1_it_worked.png
       :alt: it worked!
       :align: center

So let’s write a test that asserts than when we go to :red:`http://127.0.0.1:8000` we
get a page with :bolditalic:`Welcome to Django` in its title. Of course, this test will fail,
because we don’t have any project defined yet! But that’s what’s all about:
create your test first, and then the code.

First, we will create another virtual environment for testing, with Django 1.8 in it:

.. code-block:: bash

    $ mkvirtualenv --python=/usr/local/bin/python3 tb_test
    $ pip install Django==1.8

Next, in order to simulate a browser for our testing, we will use the :red:`Selenium`
package (before installing it you will need to have Firefox):

.. code-block:: bash

    $ pip install --upgrade selenium

Go inside the :red:`taskbuster_project` folder and create a folder for the **functional tests.**
This folder will contain all the files that **test the project functionality from the user point of view.**
Create also the file :red:`all_users.py` in it:

.. code-block:: bash

    $ cd taskbuster_project
    $ mkdir functional_tests
    $ cd functional_tests
    $ touch all_users.py

Edit this file with your editor and write:

.. code-block:: python
    :linenos:

    # -*- coding: utf-8 -*-
    from selenium import webdriver
    import unittest


    class NewVisitorTest(unittest.TestCase):

        def setUp(self):
            self.browser = webdriver.Firefox()
            self.browser.implicitly_wait(3)

        def tearDown(self):
            self.browser.quit()

        def test_it_worked(self):
            self.browser.get('http://localhost:8000')
            self.assertIn('Welcome to Django', self.browser.title)

    if __name__ == '__main__':
        unittest.main(warnings='ignore')

Let’s analyze the previous code step by step:

* The first line indicates the coding of the file
* Then, it imports :orange:`selenium` and :orange:`unittest`, a Python library for testing
* It creates a :orange:`TestCase` class, named :orange:`NewVisitorTest`, with:
    * a :orange:`setUp` method that initializes the test. It opens the browser and it waits for 3 seconds if needs to (if the page is not loaded).
    * a :orange:`tearDown` method that runs after each test. It closes the browser.
    * a method that starts with test and that asserts that the title of the webpage has :bolditalic:`Welcome to Django` in it.
* The :orange:`setUp` and :orange:`tearDown` methods run at the beginning and at the end of each test method (the ones that start with the word test).
* The final lines mean that only if Python runs the file directly (not imported) it will execute the function :orange:`unittest.main()`. This function launches the :red:`unittest Test runner`, that identifies the different tests defined by looking for methods that start with test.
* We call the :orange:`unittest.main()` function with the optional parameter :orange:`warnings=’ignore’` to avoid a ResourceWarning message.

Let’s run this script with

.. code-block:: bash

    $ python all_users.py

The resulting output shows how the test, obviously, fails. You will see something like

.. code-block:: bash

    FAIL: test_it_worked (__main__.NewVisitorTest)

and an :red:`AssertionError` with the message :bolditalic:`Welcome to Django` not found.

So let’s create a Django project and make this test pass!

Создаем проект Django
---------------------

Get inside the :red:`taskbuster_project` folder and type:

.. code-block:: bash

    $ django-admin.py startproject taskbuster

Note the dot at the end of the command, this will create the project
taskbuster without creating any additional folders. The current structure of
your top folder should be:

.. figure:: _static/taskbuster_part1_folder_structure.png
       :alt: folder_structure
       :align: center

Note: this picture shows the output of my terminal when using the tree command
(if you want to use it, you may have to install it first).

As you can see, we have created:

* the :red:`manage.py` file, used to manage the development server, database migrations, custom scripts, etc.
* the taskbuster folder, which contains:
    * :red:`__init__.py` file indicating that this folder is a Python package
    * :red:`settings.py` file, used to configure the project
    * :red:`urls.py`, used to relate urls with views
    * and the :red:`wsgi.py` file, used to configure Django’s deployment.

Later in this tutorial, you will see that the taskbuster folder will also
contain all our apps, templates, static files and other files related to our project.

Запуск сервера разработки
-------------------------

Once we have the project created, we can start the development server.
Open a tab in your terminal with the :red:`tb_dev` environment activated, go inside
the :red:`taskbuster_project` folder and run

.. code-block:: bash

    $ python manage.py runserver

You might see a warning about migrations, but don’t worry, we’ll get to that
latter in this tutorial. At the end of the output you might see something like

.. code-block:: bash

    Starting development server at http://127.0.0.1:8000/.

You can open a browser and check this url to see the :bolditalic:`It worked Django`
message, but I rather prefer to use the test we have created |smile|

Open another terminal tab (with ctrl+t or cmd+t) and activate the :red:`tb_test`
envrironment. Now, let’s run our test:

.. code-block:: bash

    $ python functional_tests/all_users.py

And now you should see the message:

.. code-block:: bash

    Ran 1 test in 0.05s
    OK

indicating that your test passed! |smile|

Now it would be a good moment to start a Git repository and start the version
control of your code. However, before going into that we need to talk about
Security. :redbold:`We want to keep the Secret Keys out of the version control`,
to maintain them… *Secret*!

And you will learn how to do that on the next part of the tutorial,
:doc:`Settings files and Version control </part_2>`

Don’t miss it! |смайл|

Please help me and share it to your friends, they might find it useful too! |смайл|
