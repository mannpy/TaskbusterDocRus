Часть V – Интернационализация и Локализация. Языки и временные зоны
===================================================================

.. role:: red
.. role:: redbold
.. role:: bolditalic
.. role:: orange
.. |;)| image:: _static/1f609.png
.. |:)| image:: _static/1f642.png

.. |br| raw:: html

   <br />

В этой части учебника TaskBuster Django мы поговорим об
:redbold:`Интернационализации`, :redbold:`Локализации` и :redbold:`Часовых поясах`.

Мы определим два различных языка для нашего веб-сайта, создадим url-адреса,
конкретные для каждого из них, и сделаем так, чтобы наш веб-сайт поддерживал эти два языка.

Кроме того, мы будем говорить о часовом поясе и о том, как мы можем
показать местное время на шаблоне.

Содержание этой части:

* :ref:`Internationalization-Settings`
* :ref:`Internationalization-Urls`
* :ref:`Internationalization-Templates`
* :ref:`Internationalization-Translation`
* :ref:`Localization`
* :ref:`Time-Zones`

Давайте начнем! |:)|

.. _Internationalization-Settings:

Интернационализация – Настройки
-------------------------------

Давайте поработаем над тем, как можно реализовать интернационализацию,
то есть, чтобы наша страница могла поддерживать несколько языков.

.. figure::  _static/taskbuster_internationalization-288x300.jpg
       :alt: taskbuster_internationalization
       :align: center

       Сделайте Django распространенным по всему миру говорящим на разных языках!

Вначале, мы создадим папку для сохранения всех файлов перевода.
Эти файлы будут создаваться автоматически с помощью Django, пишущего строки,
которые мы хотим перевести. Я покажу вам, как вы можете сказать Django,
какие строки вы хотите перевести позднее, но идея состоит в том, что вы редактируете файлы
и помещаете перевод для каждой строки вручную. Таким образом, Django будет выбирать
один язык или другой в зависимости от предпочтений пользователя.

Папка переводов будет находиться внутри папки :red:`taskbuster`:

.. code-block:: bash

    $ mkdir taskbuster/locale

Затем, откройте файл :red:`settings/base.py` и убедитесь, что у вас есть

.. code-block:: python

    USE_I18N = True

и шаблонный процессор контекста ``django.template.context_processors.i18n``
внутри настройки ``TEMPLATES['OPTIONS']['context_processors']``:

.. code-block:: python

    TEMPLATES = [
    {
        ...
        'OPTIONS': {
            'context_processors': [
                ...
                'django.template.context_processors.i18n',
            ],
        },
    },
    ]

Примечание: Вы также можете найти значение определенного параметра
с помощью оболочки Django. Например:

.. code-block:: pycon

    $ python manage.py shell
    >>> from django.conf import settings
    >>> settings.TEMPLATES

и он будет выводить текущее значение этой переменной.

Далее добавьте промежуточный слой локализации (Locale middleware) в правильном
положении, чтобы иметь возможность определить языковые предпочтения
пользователя через контекст запроса:

.. code-block:: python

    MIDDLEWARE_CLASSES = (
    ...
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.locale.LocaleMiddleware',
    'django.middleware.common.CommonMiddleware',
    ...
    )

Затем укажите языки, которые вы хотите использовать:

.. code-block:: python

    from django.utils.translation import ugettext_lazy as _
    LANGUAGES = (
        ('en', _('English')),
        ('ca', _('Catalan')),
    )

Мы будем использовать Английский и Каталанский (но не стесняйтесь ставить языки,
которые вы хотите. Вы можете найти свои коды
`здесь <http://msdn.microsoft.com/en-us/library/ms533052(v=vs.85).aspx>`_).
Функция :orange:`ugettext_lazy` используется для обозначения названия языков для перевода,
и обычно применяется ярлык функции :orange:`_`.

Примечание: есть еще одна функция, :orange:`ugettext`, используемая для перевода.
Разница между этими двумя функциями заключается в том, что :orange:`ugettext`
переводит строку сразу же, тогда как :orange:`ugettext_lazy` переводит
строку при отрисовке шаблона.

Для нашего :red:`settings.py` мы должны использовать :orange:`ugettext_lazy`,
потому что другая функция вызвала бы циклы импорта. В общем, вы должны использовать
:orange:`ugettext_lazy` так же в ваших файлах :red:`model.py` и :red:`forms.py`.

Кроме того, параметр ``LANGUAGE_CODE`` задает язык по умолчанию, который Django
будет использовать, если никакого перевода не будет найдено. Я оставлю по умолчанию:

.. code-block:: python

    LANGUAGE_CODE = 'en-us'

И наконец, указжите папку locale, которую мы создали ранее:

.. code-block:: python

    LOCALE_PATHS = (
    os.path.join(BASE_DIR, 'locale'),
    )

Не забывайте идущую следом запятую.

.. _Internationalization-Urls:

Интернационализация – URL-Адреса
--------------------------------

Хорошо, теперь, когда мы настроили параметры, мы должны подумать о том,
как мы хотим, чтобы приложение вело себя с разными языками. Здесь мы будем
использовать следующий подход: мы будем указывать префикс языка на каждый URL,
который укажет Django, какой язык использовать. Для домашней странице
это будет что-то вроде:

* :red:`mysite.com/en`
* :red:`mysite.com/ca`

И для остальной части URL-адресов, как :red:`mysite.com/myapp`, это будет:

* :red:`mysite.com/en/myapp`
* :red:`mysite.com/ca/myapp`

Таким образом, пользователь легко может переходить из одного языка на другой.
Однако, мы не хотим, чтобы ни :red:`robots.txt`, ни :red:`humans.txt` не
придерживались этой структуры (поисковые системы будут смотреть на :red:`mysite.com/robots.txt`
и :red:`mysite.com/humans.txt`, чтобы найти их).

Одним из способов реализовать это со следующим файлом :red:`urls.py`:

.. code-block:: python

    # -*- coding: utf-8 -*-
    from django.conf.urls import include, url
    from django.contrib import admin
    from django.conf.urls.i18n import i18n_patterns
    from .views import home, home_files

    urlpatterns = [
    url(r'^(?P<filename>(robots.txt)|(humans.txt))$',
        home_files, name='home-files'),
    ]

    urlpatterns += i18n_patterns(
    url(r'^$', home, name='home'),
    url(r'^admin/', include(admin.site.urls)),
    )

Обратите внимание, что мы оставили :red:`robots.txt` и :red:`humans.txt` с тем же URL,
а те URL, которые мы хотим иметь переведенными используют функцию ``i18n_patterns``.

Запустите локальный сервер и зайдите на главную страницу,
он должен перенаправить на :red:`/en` или :red:`/ca`.
Вы можете узнать больше о том,
`как Django обнаруживает предпочтение языка <https://docs.djangoproject.com/en/1.8/topics/i18n/translation/#how-django-discovers-language-preference>`_
в официальной документации.

Но как пользователю изменить свои языковые предпочтения? Ну,
Django идет вместе с представлением, который делает это для вас |;)|

Это представление ожидает POST запрос с параметром языка.
Однако, мы позаботимся об этом представлении в другое время в этом учебнике,
при настройке верхней панели навигации. Идея в том, чтобы иметь выпадающее
меню со всеми возможными языками, и просто выбирать один, чтобы изменить его.

Прежде чем продолжить, давайте запустим наши тесты,

.. code-block:: console

    $ python mange.py test taskbuster.test

Ой.... один провалился! Ну на самом деле оба unittest в :red:`taskbuster/test.py`
провалились, так как шаблон для рендеринга, при попытке использовать
``reverse("home")``, не найден. Это потому, что нам нужно установить активный
язык для функции reverse, чтобы она работала должным образом. Сначала, напишите это
в верхней части файла:

.. code-block:: python

    from django.utils.translation import activate

и затем, активируйте выбранный язык сразу после объявления теста. Например:

.. code-block:: python

    def test_uses_index_template(self):
        activate('en')
        response = self.client.get(reverse("home"))
        self.assertTemplateUsed(response, "taskbuster/index.html")

И то же самое для другого теста: :red:`test_uses_base_template`.

Теперь тесты проходят. Вы должны сделать то же самое для функциональных тестов
:red:`functional_tests/test_all_users.py`: импортируйте метод :orange:`activate`
в начале файла и добавьте :orange:`activate(‘en’)` в качестве последнего шага
для метода :orange:`setUp`.

.. _Internationalization-Templates:

Интернационализация – Шаблоны
-----------------------------

Давайте сосредоточимся теперь на том, как мы можем перевести h1 Hello World
(Здравствуй мир) - название домашней страницы. Откройте шаблон :red:`index.html`
и ищите ``<h1>Hello, world!</h1>``.

Мы будем использовать два различных шаблонных тега:

* trans используется, чтобы перевести одну строку – мы будем использовать ее для заголовка
* blocktrans используется для расширенного содержания – мы будем использовать его для абзаца

Измените содержимое h1 и p контейнера jumbotron для следующего кода:

.. code-block:: django

    <div class="jumbotron">
    <div class="container">
        <h1>{% trans "Welcome to TaskBuster!"%}</h1>
        <p>{% blocktrans %}TaskBuster is a simple Task manager that helps you organize
        your daylife. </br> You can create todo lists, periodic tasks and more! </br></br>
        If you want to learn how it was created, or create it yourself!,
        check www.django-tutorial.com{% endblocktrans %}</p>
        <p><a class="btn btn-primary btn-lg" role="button">Learn more &raquo;</a></p>
      </div>
    </div>
    </div>

Кроме того, чтобы иметь доступ к предыдущим шаблонным тегам, вам необходимо
написать ``{% load i18n %}`` в верхней части шаблона.  В этом случае, после
тега наследования (extends) от :red:`base.html`.

.. _Internationalization-Translation:

Интернационализация – Перевод
-----------------------------

Наконец, мы можем перевести наши строки!

Перейдите в терминал, внутри папки :red:`taskbuster_project`
(на том же уровне, что и файл :red:`manage.py`), и выполните команду:

.. code-block:: console

    $ python manage.py makemessages -l ca

Это создаст файл сообщений для языка, который мы хотим перевести.
Поскольку мы запишем весь наш код на английском языке, нет необходимости
создавать файл сообщений для этого языка.

Но охх!! мы получаем уродливую ошибку, которая говорит, что у нас не установлена
библиотека :redbold:`GNU gettext` (если у вас не возникает сообщение об ошибке,
это хорошо для вас! тогда пропустите эту часть установки!). Перейдите на
`главную страницу GNU gettext <https://www.gnu.org/software/gettext/>`_
и скачайте последнюю версию. Внутри zip архива вы найдете инструкцию по установке
в файле с именем :red:`INSTALL`.

В принципе, вы должны перейти в папку пакета (после распаковки) и ввести:

.. code-block:: console

    $ ./configure

чтобы настроить установку для вашей системы. Далее, введите

.. code-block:: console

    $ make

для компиляции пакета. Я всегда удивляюсь, почему некоторые установки
печатают весь этот ужасный код в вашем терминале!

Если вы хотите, используйте

.. code-block:: console

    $ make check

для запуска тестов пакета перед установкой, чтобы убедиться, что все работает.
И, наконец, запустите

.. code-block:: console

    $ make install

чтобы установить пакет.

Хорошо! Давайте вернемся к нашей среде разработки,
и попытаемся создать наш файл сообщений!

.. code-block:: console

    $ python manage.py makemessages -l ca

Да! Это сработало!

Теперь перейдите в папку :red:`taskbuster/locale`, чтобы посмотреть, что там.

.. code-block:: console

    $ cd taskbuster/locale
    $ ls

Тут есть папка с именем ca (или язык, который вы выбрали для перевода) с папкой
под названием :red:`LC_MESSAGES` внутри. Если вы зайдете внутрь, то найдете
еще один файл с именем :red:`django.po`. Проверьте этот файл с помощью текстового редактора.

В начале файла есть некоторые метаданные, но после этого вы увидите строки,
которые мы отметили для перевода:

* Наименования языка :red:`“English”` (Английский) и :red:`“Catalan”` (Каталонский) в файле настроек :red:`base.py`
* Заголовок :red:`Welcome to TaskBuster!` в файле :red:`index.html`
* Абзац после заголовка в файле :red:`index.html`

Каждое из этих предложений появляется в строке, начинающейся с :orange:`msgid`.
Вы должны поместить ваш перевод в следующей строке,
которая начинается с :orange:`msgstr`. Перевод заголовка прост:

.. code-block:: po

    msgid "Welcome to TaskBuster!"
    msgstr "Benvingut a TaskBuster!"

И с абзацем, вы должны быть осторожны, нужно чтобы каждая строка начиналась и заканчивалась с ``""``:

.. code-block:: po

    msgid ""
    "TaskBuster is a simple Task manager that helps you organize your daylife. </"
    "br> You can create todo lists, periodic tasks and more! </br></br> If you "
    "want to learn how it was created, or create it yourself!, check www.django-"
    "tutorial.com"
    msgstr ""
    "TaskBuster és un administrador senzill de tasques que t'ajuda a administrar "
    "el teu dia a dia. </br> Pots crear llistes de coses pendents, tasques "
    "periòdiques i molt més! </br></br> Si vols apendre com s'ha creat, o"
    "crear-lo tu mateix!, visita la pàgina <a href='http://www.marinamele.com/taskbuster-django-tutorial' target=_'blank'>Taskbuster Django Tutorial</a>."

Кроме того, обратите внимание на заключительный пробел в конце. Если вы не включите этот пробел, слова в конце строки и в начале следующей строки будут объединены.

Как только у вас есть все переводы, вы должны скомпилировать их используя:

.. code-block:: console

    $ python manage.py compilemessages -l ca

Вы можете запустить свой локальный сервер и увидеть эффект, перейдя на
главную страницу, но я предпочитаю писать сначала тест! |:)|

В :red:`functional_tests/test_all_users.py`  добавьте следующие тесты:

.. code-block:: python

    def test_internationalization(self):
        for lang, h1_text in [('en', 'Welcome to TaskBuster!'),
                                    ('ca', 'Benvingut a TaskBuster!')]:
            activate(lang)
            self.browser.get(self.get_full_url("home"))
            h1 = self.browser.find_element_by_tag_name("h1")
            self.assertEqual(h1.text, h1_text)

Не забудьте изменить предложение ``Benvingut a TaskBuster!``
и ``activate('ca')``, если вы используете другой язык!

Я надеюсь, что все ваши тесты прошли! |:)|

.. _Localization:

Локализация
-----------

Django способен представить времена и даты в шаблоне, используя формат,
указанный для текущей **локали**. Это означает, что два человека
с различными локалями могут видеть различный формат даты в шаблоне.

Чтобы включить Локализацию, откройте файл настроек :red:`base.py` и убедитесь, что:

.. code-block:: python

    USE_L10N = True

Таким образом, при включении этого значения в шаблоне, Django будет пытаться
выполнить его используя формат **локали**.  Тем не менее, нам также нужен способ,
чтобы отключить это автоматическое форматирование. Например, при использовании JavaScript,
нам нужно, чтобы значение, которое мы используем имело единый формат для всех локалей.

Давайте напишем для него тест! На главной странице мы покажем сегодняшнюю дату и время,
используя как местные, так и неместные форматы.

Откройте :red:`functional_tests/test_all_users.py` и запишите эти импорты наверху:

.. code-block:: python

    from datetime import date
    from django.utils import formats

и затем добавьте этот метод тестирования внутри класса ``HomeNewVisitorTest``:

.. code-block:: python

    def test_localization(self):
        today = date.today()
        for lang in ['en', 'ca']:
            activate(lang)
            self.browser.get(self.get_full_url("home"))
            local_date = self.browser.find_element_by_id("local-date")
            non_local_date = self.browser.find_element_by_id("non-local-date")
            self.assertEqual(formats.date_format(today, use_l10n=True),
                                  local_date.text)
            self.assertEqual(today.strftime('%Y-%m-%d'), non_local_date.text)

Запустите тест, чтобы увидеть то, что мы должны реализовать в первую очередь.
Вы можете сделать это просто используя команду:

.. code-block:: console

    $ python manage.py test functional_tests.test_all_users.HomeNewVisitorTest.test_localization

Тест выражает недовольство, потому что не может найти элемент с идентифкатором
(id) :orange:`local-date`. Давайте создадим его!

Откройте снова шаблон :red:`taskbuster/index.html` и загрузите шаблонный тег
локализации в начале файла (например, после загрузки шаблонного тега
интернационализации):

.. code-block:: django

    {% load l10n %}

Примечание: это нижний регистр слова :redbold:`L10N`, а не 110N!!

Далее, найдите блок div :red:`container`. Мы отредактируем первые две колонки,
чтобы показать дату  на их h2 заголовках.

.. code-block:: html

    <div class="col-md-4">
      <h2 id="local-date">{{today}}</h2>
      <p>This is the time using your local information </p>
      <p><a class="btn btn-default" href="#" role="button">View details &raquo;</a></p>
    </div>
    <div class="col-md-4">
      <h2 id="non-local-date">{{today|unlocalize}}</h2>
      <p>This is the default time format.</p>
      <p><a class="btn btn-default" href="#" role="button">View details &raquo;</a></p>
    </div>

Обратите внимание на фильтр unlocalize во втором заголовке. Это отключает формат
локализации при выполнении переменной :orange:`today`.  Кроме этого, существует
еще один шаблонный тег для отключения больших блоков:

.. code-block:: django

    {% localize off %} code without localization {% endlocalize %}

Вы можете узнать больше об этом в
`документации Django <https://docs.djangoproject.com/en/1.8/topics/i18n/formatting/#template-tags>`_.

Сохраните и заново запустите тест. Он снова терпит неудачу, потому что мы
не передали переменную :orange:`today` в представлении home.
Давайте сделаем это!

Откройте views.py и отредактируйте представление :orange:`home`:

.. code-block:: python

    import datetime

    def home(request):
        today = datetime.date.today()
        return render(request, "taskbuster/index.html", {'today': today})

Перезапустите тест! Да! Он прошел! |:)|

.. _Time-Zones:

Часовые пояса
-------------

Если ваш проект должен обрабатывать даты и времени со всего мира,
то лучше работать в формате UTC (всемирное координированное время).
Таким образом, все ваши даты и времени будут иметь единое соглашение
в вашей базе данных, так что вы сможете сравнить их независимо от
часового пояса пользователя.

И не волнуйтесь, Django переводит их автоматически к нужному
часовому поясу в формах и шаблонах.

Чтобы включить поддержку часовых поясов, откройте свой файл настроек
:red:`base.py` и убедитесь, что у вас имеется:

.. code-block:: python

    USE_TZ = True

Django также рекомендует установить python-пакет `pytz <http://pytz.sourceforge.net/>`_,
который позволяет делать вычисления с временными зонами.
Кроме того, необходимо так же использовать пакет
`Celery <http://www.celeryproject.org/>`_,
менеджер очереди задач, который мы используем позже в этом учебнике.
Давайте установим его!

Активируйте вашу среду разработки и введите:

.. code-block:: console

    $ pip install pytz

и добавьте его в файл требований :red:`base.txt` (вы можете увидеть версию
установленного пакета используя :orange:`pip freeze`). Не забудьте установить
его также в вашей среде тестирования.

Существует два различных вида объектов даты и времени (datetime),
те, которые осведомлены о часовом поясе и те, которые нет.
Вы можете использовать методы datetime ``is_aware()`` и ``is_naive()``,
чтобы определить, кто из них кто.

Если мы включим поддержку часовых поясов, все экземпляры
datetime будут знать об этом. Поэтому, чтобы взаимодействовать с некоторым
сохраненным объектом datetime, мы должны создать осведомленный (an aware)
экземпляр класса datetime.

Например:

.. code-block:: python

    import datetime
    from django.utils.timezone import utc

    now_naive = datetime.datetime.now()
    now_aware = datetime.datetime.utcnow().replace(tzinfo=utc)

Кроме того, если у вас включена поддержка часовых поясов, т.е. ``USE_TZ=True``,
то имеется кратчайший путь, чтобы получить текущее осведомленное (aware) время :

.. code-block:: python

    from django.utils.timezone import now
    now_aware = now()

Однако, как описано в документации Django, нет никакого способа
определить предпочтение пользователя на временные зоны через заголовок HTTP.
Что мы и будем делать, чтобы запросить предпочтительный часовой пояс у пользователя
и сохранить его в его пользовательском профиле. Но это прибудет позже в этом
учебнике, извините |:)|

Между тем,  мы определим часовой пояс по умолчанию настройкой переменной:

.. code-block:: python

    TIME_ZONE = 'Europe/Madrid'

который является часовым поясом здесь в Барселоне |:)| Вы можете выбрать
`свой часовой пояс отсюда <http://en.wikipedia.org/wiki/List_of_tz_database_time_zones>`_.

Итак, давайте посмотрим, как отобразить текущее время здесь, в Барселоне
(используя часовой пояс по умолчанию), время UTC, и время в Нью-Йорке.

Откройте свой :red:`functional_tests/test_all_users.py` и напишите следующий тест:

.. code-block:: python

    def test_time_zone(self):
        self.browser.get(self.get_full_url("home"))
        tz = self.browser.find_element_by_id("time-tz").text
        utc = self.browser.find_element_by_id("time-utc").text
        ny = self.browser.find_element_by_id("time-ny").text
        self.assertNotEqual(tz, utc)
        self.assertNotIn(ny, [tz, utc])

мы проверим, что время в Барселоне, UTC и Нью-Йорке различны.

Запустите этот тест и вы увидите, что он терпит неудачу, потому что
элемент с идентификатором (id) :orange:`time-tz` отсутствует.
Откройте шаблон :red:`index.html` и отредактируйте третью колонку:

.. code-block:: html

    <div class="col-md-4">
    <h2>Time Zones</h2>
    <p> Barcelona: <span id="time-tz">{{now|time:"H:i"}}</span></p>
    <p> UTC: <span id="time-utc">{{now|utc|time:"H:i"}}</span></p>
    <p> New York: <span id="time-ny">
           {{now|timezone:"America/New_York"|time:"H:i"}}</span></p>
    <p><a class="btn btn-default" href="#" role="button">View details &raquo;</a></p>
    </div>

Где я использовала фильтр времени (time), чтобы показать только время.
Снова запустите тест, и он потерпит неудачу, потому что не может найти фильтр UTC.
В файле :red:`index.html` добавьте в верхней части файла:

.. code-block:: django

    {% load tz %}

Запустите ваши тесты опять и на этот раз они так же потерпят неудачу, потому что
представления (views) не передали ни одной переменной :orange:`now`.
Откройте :red:`views.py` и добавьте этот импорт в начале файла:

.. code-block:: python

    from django.utils.timezone import now

и добавьте переменную :orange:`now` в главное представление (home view):

.. code-block:: python

    def home(request):
        today = datetime.date.today()
        return render(request, "taskbuster/index.html",
                           {'today': today, 'now': now()})

Запустите ваши тесты снова, и они должны пройти!

Хорошо, это конец данной части учебника.
Не забудьте зафиксировать ваши изменения:

.. code-block:: console

    $ git add .
    $ git status
    $ git commit -m "Internationalization and localization"
    $ git push origin master

Последняя команда, только если вы хотите отправить изменения в облачное
хранилище, такие как Bitbucket или GitHub.

В следующей части учебника мы рассмотрим
:doc:`Документацию </part_6>`.
Да, это может показаться скучным предметом для вас, но я уверена,
что вы действительно цените пакет с понятной и хорошо
структурированной документацией, верно?

Не пропустите это!

Вам нравится этот учебник? Не забудьте поделиться им с друзьями! |:)|
Спасибо!
