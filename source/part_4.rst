Часть IV – Файлы веб-сайта и Тестирование с coverage (покрытие)
===============================================================

.. role:: red
.. role:: redbold
.. role:: bolditalic
.. role:: orange
.. |;)| image:: _static/1f609.png
.. |:)| image:: _static/1f642.png

.. |br| raw:: html

   <br />

Мы начнем этот раздел учебника Taskbuster, закончив наследование
шаблонов между базовой и домашней страницей.

Далее мы поговорим о том, как настроить файлы, которые используются в нормальных веб-сайтах.

Файл :red:`robots.txt` очень важен в начале проекта, потому что вы ведь не хотите,
чтобы Google индексировал любую из ваших страниц. Представьте, что будет,
если пользователь попадает на ваш сайт, ища что-нибудь в Google и находит ваш
сайт в разработке, с плохими стилями CSS и бредовым текстом!

Кроме того, вы узнаете, как реализовать :red:`Coverage` (Покрытие) в ваших тестах.
Как вам известно, этот инструмент дает вам показатель того, насколько код покрыт тестами.

Вот наброски этой части:

* :ref:`Template-Inheritance`
* :ref:`Robots.txt-and-humans.txt-files`
* :ref:`The-favico.ico-image`
* :ref:`Testing-with-Coverage`

Перед тем как начать с этого поста, вы проверили, что всем тестам мы написали прохождение? |;)|


.. _Template-Inheritance:

Наследование Шаблонов
---------------------

В последней части этого учебника, мы установили :red:`index.html` наследоваться от
:red:`base.html`. Однако единственное содержание, которое отличалось между
этими двумя файлами, было заголовком.

Давайте подготовим наш шаблон :red:`base.html` быть более гибким.
Вы можете найти окончательную версию этих файлов здесь:
`base.html <https://gist.github.com/mineta/5546c0f3f1953f8f38d1>`_ и
`index.html <https://gist.github.com/mineta/9deadb890b03e5aaec97>`_.

Мы добавим два блока в конце главного тега head в файле :red:`base.html`:

.. code-block:: django

    {% block head_css %}{% endblock %}
    {% block head_javascript %}{% endblock %}

чтобы иметь возможность включать больше :red:`css` или :red:`javascript` файлов
на наших шаблонах. Затем, мы добавим :red:`блок navbar`, который
оборачивает элемент :red:`navbar`:

.. code-block:: django

    {% block navbar %}
    <nav class="navbar navbar-inverse navbar-fixed-top" role="navigation">
    ...
    </nav>
    {% endblock %}

Таким образом, мы можем заменить содержимое навигационной панели navbar
в наших шаблонах, но если мы не указываем, эта панель навигации появится по умолчанию.

Основное содержание должно быть в отдельном блоке контента, названным :orange:`block content`.
Однако, поскольку нам не нужно никакое значение по умолчанию, мы сократим все содержимое внутри
блоков :red:`jumbotron` и :red:`container` и вставим его в файл :red:`index.html`.
Файл :red:`index.html` будет выглядеть так:

.. code-block:: django

    {% extends "base.html" %}

    {% block head_title %}TaskBuster Django Tutorial{% endblock %}

    {% block content %}
    <div class="jumbotron">
    ...
    </div>
    <div class="container">
    ...
    </div>
    {% endblock %}

А в :red:`base.html` вам просто нужно добавить это заменяющее недостающее содержание:

.. code-block:: django

    {% block content %}{% endblock %}

В некоторых версиях шаблона HTML-кода, элемент ``<footer>`` находится внутри ``<container>``.
Возможно, вы захотите сохранить этот элемент в файле :red:`base.html`.

И, наконец, иногда нам необходимо добавлять скрипты в конце html документа,
поэтому перед тегом ``</body>`` добавьте строку:

.. code-block:: django

    {% block footer_javascript %}{% endblock %}

Сохраните оба файла и запустите тесты. Домашняя страница должна выглядеть
точно так же, поэтому все тесты должны пройти!

.. _Robots.txt-and-humans.txt-files:

Файлы robots.txt и humans.txt
-----------------------------

Обычно поисковые системы ищут эти файлы:

* :red:`/robots.txt`
* :red:`/humans.txt`

Так что давайте сделаем их доступными в этих url адресах.  Перед этим,
как вы уже догадались, мы создадим функциональный тест для них |;)|

В том же :red:`functional_tests/test_all_users.py` определим следующий
метод класса ``HomeNewVisitorTest``:

.. code-block:: python

    def test_home_files(self):
        self.browser.get(self.live_server_url + "/robots.txt")
        self.assertNotIn("Not Found", self.browser.title)
        self.browser.get(self.live_server_url + "/humans.txt")
        self.assertNotIn("Not Found", self.browser.title)

который проверяет, что при переходе к соответствующему url адресу
мы не видим страницу Not Found 404 - Не Найдено 404 (Если ваш браузер отображает
другое сообщение, когда страница не найдена, то необходимо изменить тестовый текст
в соответствии с этим).

Запустите тест и посмотрите на его провал. Затем, обновите свой файл
:red:`urls.py` так:

.. code-block:: python

    from .views import home, home_files

и

.. code-block:: python

    urlpatterns = patterns(
    ...
    url(r'^(?P<filename>(robots.txt)|(humans.txt))$',
        home_files, name='home-files'),
    ...
    )

это является регулярным выражением, которое принимает желаемые
url адреса и передает в качестве аргумента имя файла, т.е.
:red:`robots.txt` или :red:`humans.txt`.

Затем, отредактируйте :red:`views.py` и добавьте
соответствующее представление :orange:`home_files`:

.. code-block:: python

    def home_files(request, filename):
        return render(request, filename, {}, content_type="text/plain")

Снова запустите тесты, теперь они должны пройти! |;)|

Но, что должны содержать эти файлы?

Пока я занимаюсь разработкой веб-сайта, я не хочу, чтобы Google
индексировала любую из моих страниц. Поэтому я пишу следующее
на моем файле :red:`robots.txt`:

.. code-block:: text

    User-agent: *
    Disallow: /

С другой стороны, в :red:`humans.txt` вы должны написать информацию
о вашей команде. Например,

.. code-block:: text

    # humanstxt.org/
    # The humans responsible & technology colophon

    # TEAM
    Marina Mele -- Developer -- @marina_mele

    # THANKS
    Thanks to all my Blog readers, who encouraged me to write the
    TaskBuster Django Tutorial

    # TECHNOLOGY COLOPHON
    Django
    HTML5 Boilerplate
    Twitter Bootstrap

Когда ваша страница будет существовать, вы должны помнить о том, чтобы снова
изменить файл :red:`robots.txt`, и позволить Google индексировать ваши страницы.
Иначе люди не смогут найти ваш сайт, когда ищут что-то в поисковых системах!

.. _The-favico.ico-image:

Изображение favicon.ico
-----------------------

Если вы не знаете, что такое изображени favicon, обратите внимание на
сферический значок коровы данного сайта в верхней части окна браузера |:)|
(`здесь <http://www.marinamele.com/taskbuster-django-tutorial/template-inheritance-website-files-and-testing-with-coverage>`_)

Вы, наверное, видели, что даже у нас есть файл :red:`favicon.ico` внутри
папки :red:`taskbuster/static`, он не виден на главной странице нашего сайта.
Это потому, что мы забыли включить его в шаблон!

Итак, перейдите к шаблону :red:`base.html` и добавьте следующую
строку после тега заголовка:

.. code-block:: django

    <link rel="shortcut icon" href="{% static 'favicon.ico' %}"
    type="image/x-icon">

Перезагрузите свою страницу, и вы увидите значок на ярлыке страницы вашего браузера.

.. _Testing-with-Coverage:

Тестирование с помощью Coverage
-------------------------------

Coverage (покрытие) является полезным инструментом для тестирования. Он говорит вам,
какие части вашего кода покрыты неким тестом.  Очень важно, что перед запуском
в production (производство), ваш код имел большой процент покрытия.

Мы будем устанавливать его только в среде тестирования, поэтому активируйте
:red:`tb_test` и выполните команду:

.. code-block:: console

    $ pip install coverage

проверьте установленную версию с :orange:`pip freeze` и добавьте его в файл
:red:`requirements/testing.txt`

Затем, запустите свои тесты с использованием coverage:

.. code-block:: console

    $ coverage run --source='.' manage.py test

который выполнит и модульные тесты unittests, и функциональные тесты.
Вы можете увидеть отчет coverage набрав:

.. code-block:: console

    $ coverage report

Если вы хотите более приятное представление в html, используйте:

.. code-block:: console

    $ coverage html

И затем, вы сможете увидеть результаты в :red:`htmlcov/index.html`, где вы увидите,
какие строки были покрыты тестом в каждом файле! Приятно!

Однако эти команды создали файлы, которые мы не хотим сохранять
системе в управлении версиями. Поэтому введите:

.. code-block:: console

    $ echo ".coverage" >> .gitignore
    $ echo "htmlcov" >> .gitignore

Это все для этой части учебника!

В следующей части, мы поговорим об **Интернационализации**.
Мы настроим два разных языка и заставим наш сайт работать с обоими.

Перейдите к следующей части –
:doc:`Интернационализация и Локализация. Множество языков и часовые пояса. </part_5>`

Нравится ли вам учебник Taskbuster? Поделитесь ею с друзьями! |:)|
