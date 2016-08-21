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

После того, как мы установили нашу рабочую среду, мы можем
сосредоточиться на **создании домашней страницы** нашего сайта.

Однако, это не будет обычной пустой домашней страницей
с одной только надписью “привет мир” в ней.

Мы настроим как статические файлы, так и шаблоны,
внедрим шаблон HTML5 и фреймворк Twitter Bootstrap,
и создадим **намного лучшую** версию обычной домашней страницы
(да, она также будет иметь надпись "привет мир"...)

К тому же, мы будем подчиняться Козе Тестирования и следовать **Разработке Через Тестирование**
(РЧТ), чтобы создать домашнюю страницу. Здесь вы узнаете много интересного! |;)|

Вот то, что мы рассмотрим:

* :ref:`Static-Files-Settings`
* :ref:`Templates-Settings`
* :ref:`Initializr-HTML5-Boilerplate-and-Twitter-Bootstrap`
* :ref:`Home-Page-with-TDD-Tests-first`
* :ref:`Home-Page-with-TDD-Code-next`
* :ref:`Commit-again-to-your-local-repository-and-Bitbucket`

.. _Static-Files-Settings:

Настройки Статических Файлов
----------------------------

Откройте общий файл настроек (:red:`settings/base.py`) и найдите переменную
``INSTALLED_APPS``. Убедитесь, что она содержит приложение :redbold:`'django.contrib.staticfiles'`.

Далее перейдите в конец файла и найдите строку:

.. code-block:: python

    STATIC_URL = '/static/'

Эта строка указывает Django искать :orange:`статические` (static) файлы в папке
static внутри каждого из наших приложений.

Однако, некоторые статические файлы используются для всего проекта
и не должны быть внутри конкретного приложения. Перейдите в папку :red:`taskbuster`,
на том же уровне где находятся файлы настроек, и создайте папку с именем :orange:`static`.

.. code-block:: bash

    $ cd taskbuster
    $ mkdir static

Этот каталог будет содержать все статические файлы, которые являются
глобальными для проекта, такие как CSS или файлы JavaScript.

Если вы посмотрите на начало файла :red:`settings/base.py`, вы найдете строку

.. code-block:: python

    BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

которая указывает на папку, содержащую другую папку, которая содержит
фактический файл, т.е. папку :red:`taskbuster`.

.. note::
    если вы работаете с одним  файлом :red:`settings.py`, расположенным в папке
    :red:`taskbuster` (без дополнительной папки настроек :red:`settings`,
    которую мы создали здесь) вам придется переопределить :orange:`BASE_DIR`
    для того, чтобы указать папку :red:`taskbuster`. Это потому, что без
    дополнительной папки настроек в середине, предыдущий :orange:`BASE_DIR`
    будет указывать на верхнюю папку :red:`taskbuster_project`, вместо папки
    :red:`taskbuster`. Вы должны переопределить его так

.. code-block:: python

    BASE_DIR = os.path.dirname(os.path.abspath(__file__))

Чтобы указать Django искать статические файлы в папке :red:`taskbuster/static`,
которую мы только что создали, напишите следующее после объявления :orange:`STATIC_URL`:

.. code-block:: python

    STATICFILES_DIRS = (
    os.path.join(BASE_DIR, "static"),
    )

Не забудьте запятую в конце! При такой конфигурации Django будет искать
статические файлы в папке :red:`static` внутри каждого приложения и
в папке :red:`taskbuster/static`, которую мы только что создали.

.. _Templates-Settings:

Настройки Шаблонов
------------------

Подобная вещь происходит и с шаблонами.
По умолчанию, загрузчик шаблонов Django
ищет шаблоны в папке :red:`templates` внутри каждого приложения.

Но давайте создадим папку шаблонов :red:`templates` внутри :red:`taskbuster`,
чтобы содержать все шаблоны, которые будут использоваться всюду по всему
проекту, такие как :red:`base.html` или некоторые страницы ошибок.

.. code-block:: bash

    $ cd taskbuster
    $ mkdir templates

Затем обновите файлы настроек и отредактируйте ключ :red:`DIRS`
в переменной :red:`TEMPLATE` так:

.. code-block:: python

    # Templates files
    TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(BASE_DIR, "templates")],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                 'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
    ]

Как и со статическими файлами, Django будет искать шаблоны,
расположенные в папке, названной :red:`templates` в каждом приложении и в
папке :red:`taskbuster/templates`, которую мы только что создали.

.. _Initializr-HTML5-Boilerplate-and-Twitter-Bootstrap:

Initializr: шаблон HTML5 и Twitter Bootstrap
--------------------------------------------

В целях проверки, что конфигурация шаблонов и статических файлов работает,
и также, потому что мне нравится разрабатывать в Django, по крайней мере,
с некоторым CSS, мы собираемся включить
`Шаблон HTML5 <http://html5boilerplate.com/>`_
и `Bootstrap <http://getbootstrap.com/>`_.
Эти инструменты помогают создавать адаптивные шаблоны,
которые работают во многих браузерах.

Здесь мы будем использовать `Initializr <http://www.initializr.com/>`_,
смешанную версию, которая сочетает в себе Шаблон HTML5 и Bootstrap.
Перейдите на этот сайт, нажмите на "Получить Пользовательскую Сборку"
(Get Custom Build), и выберите приоритеты вашего проекта.
Конфигурация, которую я выбрала для этого проекта, показана на изображении
(за исключением Apple Touch Icons,
которые не используются в этом проекте):

.. figure:: _static/taskbuster_part2_initializr-1024x868.png
       :alt: initializr
       :align: center

После того как вы загрузите пакет, распакуйте и преобразуйте его содержимое:

* Переместите файлы :red:`index.html`, :red:`404.html`, :red:`humans.txt` и :red:`robots.txt` в папку :red:`taskbuster/templates`.
* Измените имя :red:`index.html` на :red:`base.html`. Индексный файл, как правило является шаблоном вашей Домашней страницы, но мы будем использовать его в качестве базового шаблона - все наши шаблоны сайта будут наследовать от этого базового шаблона.
* Переместите другие файлы и папки в папку :red:`taskbuster/static`
* Если у вас есть значок, который вы хотите использовать для вашего приложения, замените его на файл :red:`favicon.ico` (я рекомендую вам использовать то же имя).
* Я также удалила файлы :red:`apple-touch-icon.png`, :red:`browserconfig.xml`, :red:`tile-wide.png` и :red:`tile.png`.

.. _Home-Page-with-TDD-Tests-first:

Домашняя страница с РЧТ – Сначала тесты
---------------------------------------

Для того чтобы увидеть, загружаются ли статические файлы и шаблоны
правильно, нам будет нужен тест. Ну вы знаете… Повинуйтесь Козе тестирования!
:bolditalic:`Сначала тест, сначала тест!`

.. figure:: _static/obey_the_testing_goat-300x290.jpg
       :alt: testing goat
       :align: center

       Сначала тест, Сначала тест!

На самом деле, чтобы работать с Разработкой Через Тестирование (РЧТ)
- Test Driven Development, мы должны были написать тест даже прежде,
чем установить шаблоны и статические папки. Но я сперва хотела закончить
редактирование файлов настроек.

Вначале, мы преобразуем папку :red:`functional_tests` в пакет,
включив пустой файл с именем :red:`__init__.py` внутри.

.. code-block:: bash

    touch functional_tests/__init__.py

Таким образом, мы можем запустить наши функциональные тесты вот так:

.. code-block:: bash

    $ python manage.py test functional_tests

Однако, исполнитель тестов (test runner) находит только файлы, которые
начинаются со слова "тест", так что давайте переименуем
:red:`all_users.py` на :red:`test_all_users.py`.

Мы дадим git проконтролировать это, так что репозиторий обнаружит перемещение правильно:

.. code-block:: bash

    $ git mv functional_tests/all_users.py functional_tests/test_all_users.py

Запустите сервер разработки, используя среду :red:`tb_dev`, а затем выполните
функциональные тесты в среде :red:`tb_test`.
Все должно работать как и прежде, ничего не сломалось!

Но я уверена, что вам не нравится эта игра с двумя средами, не так ли?
Необходимость запускать сервер, используя :red:`tb_dev` и тестировать с :red:`tb_test`.
**Почему сам тест не может создавать сервер?**

**Кроме того, изменения, сделанные этими функциональными тестами являются постоянными.**
Представьте себе, что во время одного теста,
мы создаем экземпляр модели (например, нового пользователя).
Мы хотим, чтобы после выполнения теста, этот экземпляр (пользователь)
исчез из нашей базы данных, верно?
Но с функциональными тестами, мы просто запускаем сервер разработки
и играем с базой данных разработки, так что эти изменения сохраняются
даже после завершения теста.

Но не волнуйтесь, есть :orange:`LiveServerTestCase`, чтобы сделать
нашу жизнь намного проще! |;)|

Как мы увидим, экземпляры этого класса создают сервер с **базой данных тестирования**,
так же как и когда мы запускали **unittests**.

Теперь, давайте отредактируем файл :red:`functional_tests/test_all_users.py`
и проверим, что обе шаблоны и статические папки работают, как и ожидалось.
Например, мы можем протестировать эти две разные вещи.

* Заголовок в домашней странице - “TaskBuster”
* Цвет текста заголовка h1 в домашней странице :red:`rgba(200, 50, 255, 1)` ~ розовый цвет.

Итак, давайте создадим этот тест! |br|
Примечание: файл :red:`test_all_users.py` содержит ``NewVisitorTest``,
созданный в части I. Вы должны заменить тот тест на этот:

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

Давай пройдемся через этот код шаг за шагом:

* Сначала, мы определяем вспомогательную функцию с именем ``get_full_url``, которая принимает один аргумент, :orange:`пространство имен`
    * :orange:`Пространство имен` является идентификатором для url. Это хорошая вещь в Django: когда вы работаете с идентификаторами, вы можете изменить url-адрес, на то, что вы хотите, а *код просто работает* как и прежде.
    * ``self.live_server_url`` дает вам локальный url-адрес узла. Мы используем этот метод потому, что тестовый сервер использует другой url (обычно ``http://127.0.0.1:8021``), и этот метод является более гибким.
    * reverse дает **относительный** url-адрес из заданного пространства имен, здесь ``/``
    * Полученная функция дает вам абсолютный адрес этого пространства имен (сумма двух предыдущих), ``http://127.0.0.1:8021/``.
* Метод ``test_home_title`` проверяет, что заголовок домашней страницы содержит слово :red:`TaskBuster`. Мы создадим шаблон для этого, так что если заголовок существует, то это означает, что шаблон был загружен правильно.
* Метод ``test_h1_css`` проверяет, что текст h1 имеет нужный цвет. Правило CSS для цвета текста будет в файле CSS, который означает, что если тест пройден, то статические файлы загружаются правильно.
* Наконец, обратите внимание на то, что мы удалили, выражение ``if __name__ == '__main__'``, так как :red:`functional_tests` является теперь пакетом, который будет работать с исполнителем тестов Django.

Как только мы создали наш тест, РЧТ говорит нам следовать за циклом:

* запускаем тест и смотрим, что он терпит неудачу
* пишем некоторый код так, чтобы он исправлял тестовое сообщение об ошибке (написать только код, который исправляет сообщение об ошибке, показанное на тестовом провале, не ожидайте других возможных ошибок)

Мы должны следовать этому циклу до полного прохождения теста.
Это будет более ясно в следующем разделе.

.. _Home-Page-with-TDD-Code-next:

Домашняя страница с РЧТ – Далее код
-----------------------------------

Теперь, когда у нас есть функциональный тест для нашей домашней страницы,
мы можем запустить его и посмотреть, как он терпит неудачу.
В наше тестовой среде :red:`tb_test`:

.. code-block:: bash

    $ python manage.py test functional_tests

Мы видим, что первая ошибка заключается в том, что пространство имен
:redbold:`"home"` не определено. Откройте файл :red:`taskbuster/urls.py`
и импортируйте представление ``home`` из :red:`views.py` (в зависимости от
вашей версии Django, вам, возможно, потребуется создать этот файл сначала:
:red:`taskbuster/views.py`):

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
