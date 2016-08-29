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

Схема этой части:

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

Note: there is another function, :orange:`ugettext`, used for translation.
The difference between these two functions is that :orange:`ugettext` translates
the string immediately whereas :orange:`ugettext_lazy` translates the string
when rendering a template.

For our :red:`settings.py`, we need to use :orange:`ugettext_lazy` because the other function
would cause import loops. In general, you should use :orange:`ugettext_lazy`
in your :red:`model.py` and :red:`forms.py` files as well.

Moreover, the ``LANGUAGE_CODE`` setting defines the default language that Django
will use if no translation is found. I’ll leave the default:

.. code-block:: python

    LANGUAGE_CODE = 'en-us'

And finally, specify the locale folder that we created before:

.. code-block:: python

    LOCALE_PATHS = (
    os.path.join(BASE_DIR, 'locale'),
    )

Don’t forget the trailing coma.

.. _Internationalization-Urls:

Internationalization – Urls
---------------------------

Ok, now that we have configured the settings, we need to think about
how we want the app to behave with different languages. Here we will
take the following approach: we will include a language prefix on each url
that will tell Django which language to use. For the Home page
it will be something like:

* :red:`mysite.com/en`
* :red:`mysite.com/ca`

And for the rest of urls, like :red:`mysite.com/myapp`, it will be:

* :red:`mysite.com/en/myapp`
* :red:`mysite.com/ca/myapp`

This way the user may change from one language to another easily. However,
we don’t want that neither the :red:`robots.txt` nor the :red:`humans.txt` files follow
this structure (search engines will look at :red:`mysite.com/robots.txt`
and :red:`mysite.com/humans.txt` to find them).

One way to implement this is with the following :red:`urls.py` file:

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

Note that we left the :red:`robots.txt` and :red:`humans.txt` files with the same url,
and the ones that we want to be translated use the ``i18n_patterns`` function.

Run your local server and visit the home page, it should redirect to :red:`/en` or :red:`/ca`.
You can learn more about
`how Django discovers language preference <https://docs.djangoproject.com/en/1.8/topics/i18n/translation/#how-django-discovers-language-preference>`_
in the official documentation.

But how does the user change its language preferences? Well,
Django comes with a view that does this for you |;)|

This view expects a POST request with a language parameter.
However, we will take care of that view in another time in this tutorial,
when customizing the top navigation bar. The idea is to have a drop-down
menu with all the possible languages, and just select one to change it.

Before proceeding, let’s run our tests,

.. code-block:: console

    $ python mange.py test taskbuster.test

Oh…. one failed! Well actually both unittest in :red:`taskbuster/test.py` fail,
as the template rendered when trying to use ``reverse("home")`` is not found.
This is because we need to set an active language for the reverse to work
properly. First, write this at the top of the file:

.. code-block:: python

    from django.utils.translation import activate

and next, activate the selected language just after the test declaration. For example:

.. code-block:: python

    def test_uses_index_template(self):
        activate('en')
        response = self.client.get(reverse("home"))
        self.assertTemplateUsed(response, "taskbuster/index.html")

And the same for the other test: :red:`test_uses_base_template`.

Now, tests pass. You should do the same for the :red:`functional_tests/test_all_users.py`:
import the :orange:`activate` method at the beginning of the file and add
the :orange:`activate(‘en’)` as the last step on the :orange:`setUp` method.

.. _Internationalization-Templates:

Internationalization – Templates
--------------------------------

Let’s focus now on how we can translate the h1 Hello World title of
the Home Page. Open the :red:`index.html` template and look for
``<h1>Hello, world!</h1>``.

We will use two different template tags:

* trans is used to translate a single line – we will use it for the title
* blocktrans is used for extended content – we will use it for a paragraph

Change the h1 and p contents of the jumbotron container for the following code:

.. code-block:: django

    <div class="jumbotron">
    <div class="container">
        <h1>{% trans "Welcome to TaskBuster!"%}</h1>
        <p>{% blocktrans %}TaskBuster is a simple Task manager that helps you organize your daylife. </br> You can create todo lists, periodic tasks and more! </br></br> If you want to learn how it was created, or create it yourself!, check www.django-tutorial.com{% endblocktrans %}</p>
        <p><a class="btn btn-primary btn-lg" role="button">Learn more &raquo;</a></p>
      </div>
    </div>
    </div>

Moreover, to have access to the previous template tags, you will have
to write ``{% load i18n %}`` near the top of your template. In this case,
after the extends from :red:`base.html` tag.

.. _Internationalization-Translation:

Internationalization – Translation
----------------------------------

Finally, we are able to translate our strings!

Go to the terminal, inside the :red:`taskbuster_project` folder
(at the same level as the :red:`manage.py` file), and run:

.. code-block:: console

    $ python manage.py makemessages -l ca

This will create a message file for the language we want to translate.
As we will write all our code in english, there is no need to create
a message file for that language.

But ohh!! we get an ugly error that says that we don’t have
the :redbold:`GNU gettext` installed (if you don’t get the error, good
for you! skip this installation part then!). Go to
the `GNU gettext home page <https://www.gnu.org/software/gettext/>`_
and download the last version.
Inside the zip file you’ll find the installation
instructions on a file named :red:`INSTALL`.

Basically, you should go inside the package folder (once unzipped) and type:

.. code-block:: console

    $ ./configure

to configure the installation for your system. Next, type

.. code-block:: console

    $ make

to compile the package. I always wonder why some installations
print all that awful code on your terminal!

If you want, use

.. code-block:: console

    $ make check

to run package tests before installing them, to see that everything works.
Finally, run

.. code-block:: console

    $ make install

to install the package.

Okey! Let’s go back to our developing environment,
and try to generate our message file!

.. code-block:: console

    $ python manage.py makemessages -l ca

Yes! It worked!

Now go to the :red:`taskbuster/locale` folder to see what’s in there.

.. code-block:: console

    $ cd taskbuster/locale
    $ ls

There is a folder named ca (or the language you chose to translate) with
a folder named :red:`LC_MESSAGES` inside. If you go inside it, you’ll find another
file named :red:`django.po`. Inspect that file with your editor.

There is some metadata at the beginning of the file, but after that
you’ll see the strings we marked for translation:

* The language’s names :red:`“English”` and :red:`“Catalan”` in the :red:`base.py` settings file
* The :red:`Welcome to TaskBuster!` title on the :red:`index.html` file
* The paragraph after the title on the :red:`index.html` file

Each of these sentences appear in a line beginning with :orange:`msgid`.
You have to put your translation in the next line,
the one that starts with :orange:`msgstr`.
Translating the title is simple:

.. code-block:: po

    msgid "Welcome to TaskBuster!"
    msgstr "Benvingut a TaskBuster!"

And with a paragraph, you have to be careful to start and end each line with ``""``:

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

Also, note the final space at the end of the line. If you don’t include that space, the words at the end of the line and at the beginning of the next line will concatenate.

Once you have all the translations set, you must compile them with:

.. code-block:: console

    $ python manage.py compilemessages -l ca

You can run your local server and see the effect by going to the home page,
but I prefer writing a test first! |:)|

In the :red:`functional_tests/test_all_users.py` add the following tests:

.. code-block:: python

    def test_internationalization(self):
        for lang, h1_text in [('en', 'Welcome to TaskBuster!'),
                                    ('ca', 'Benvingut a TaskBuster!')]:
            activate(lang)
            self.browser.get(self.get_full_url("home"))
            h1 = self.browser.find_element_by_tag_name("h1")
            self.assertEqual(h1.text, h1_text)

Remember to change the Benvingut a ``TaskBuster``!
sentence and the ``activate('ca')`` if you’re using another language!

I hope all of your tests passed! |:)|

.. _Localization:

Localization
------------

Django is capable to render times and dates in the template using the
format specified for the current **locale**. This means that two people
with different locales may see a different date format on the template.

To enable Localization, edit the :red:`base.py` settings file and make sure that:

.. code-block:: python

    USE_L10N = True

This way, when including a value in a template, Django will try to render
it using the **locale**‘s format. However, we also need a way to deactivate this
automatic formatting. For example, when using javascript, we need that the
value we use has a uniform format for all locals.

Let’s write a test for it! In the Home Page, we will show today’s date
and time using both local and non-local formats.

Open the :red:`functional_tests/test_all_users.py` and write these imports at the top:

.. code-block:: python

    from datetime import date
    from django.utils import formats

and next, add this test method inside the ``HomeNewVisitorTest`` class:

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

Next, run the test to see what should we implement first.
You can run just this test using the command:

.. code-block:: console

    $ python manage.py test functional_tests.test_all_users.HomeNewVisitorTest.test_localization

The test complains because it can’t find the element with a :orange:`local-date` id.
Let’s create it!

Open again :red:`taskbuster/index.html` template and load the localization
template tags at the beginning of the file (for example, after loading
the internationalization template tags):

.. code-block:: django

    {% load l10n %}

Note: that it’s a lowercase :redbold:`L10N`, not a 110N!!

Next, look for the :red:`container` div. We will edit the first two columns
to display the date on their h2 headers:

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

Note the unlocalize filter on the second header. This disables the localization
format while rendering the :orange:`today` variable. Moreover, there is another
template tag to disable large blocks:

.. code-block:: django

    {% localize off %} code without localization {% endlocalize %}

You can learn more about this in the
`Django documentation <https://docs.djangoproject.com/en/1.8/topics/i18n/formatting/#template-tags>`_.

Save and run the test again. It fails again, because we didn’t pass
the :orange:`today` variable in the home view. Let’s do that!

Open the views.py and edit the :orange:`home` view:

.. code-block:: python

    import datetime

    def home(request):
        today = datetime.date.today()
        return render(request, "taskbuster/index.html", {'today': today})

Rerun the test! Yeah! It passed! |:)|

.. _Time-Zones:

Time Zones
----------

If your project has to handle dates and times from all over the world,
it’s better to work in UTC (coordinated universal time).
This way all your dates and times have a uniform convention
in your database, so you can compare them no matter the time zone of the user.

And don’t worry, Django translates them automatically to
the desired time zone in forms and templates.

To enable the time zone support, open your :red:`base.py` settings
file and make sure you have

.. code-block:: python

    USE_TZ = True

Django also recommends to install `pytz <http://pytz.sourceforge.net/>`_,
a Python package that allows
for timezones calculations. Moreover, it’s also a package needed
to use `Celery <http://www.celeryproject.org/>`_, a task queue manager that we will use latter in this
tutorial. Let’s install it!

Activate your development environment and type:

.. code-block:: console

    $ pip install pytz

and add it into the :red:`base.txt` requirements file (you can see the version
installed with :orange:`pip freeze`). Remember to install it too in your testing environment.

There are two different kind of datetime objects, the ones that are
aware of the time zone and the ones that are not. You can use the datetime
methods ``is_aware()`` and ``is_naive()`` to determine which one it is.

If we enable time zone support, all the datetime instances will be aware.
Therefore, in order to interact with some saved datetime,
we should create an aware datetime instance.

For example:

.. code-block:: python

    import datetime
    from django.utils.timezone import utc

    now_naive = datetime.datetime.now()
    now_aware = datetime.datetime.utcnow().replace(tzinfo=utc)

Moreover, if you have time zone support enabled, i.e. ``USE_TZ=True``,
there is a shortcut to obtain the current aware time:

.. code-block:: python

    from django.utils.timezone import now
    now_aware = now()

However, as explained in the Django documentation, there is no way to determine
the user time zone preferences through the HTTP header. What we will do is to
ask the preferred time zone to the user and save it in his user profile.
But that will come latter in this tutorial, sorry |:)|

Meanwhile, we will define a default time zone with the setting variable:

.. code-block:: python

    TIME_ZONE = 'Europe/Madrid'

which is the time zone here in Barcelona |:)| You can pick
`your time zone from  here <http://en.wikipedia.org/wiki/List_of_tz_database_time_zones>`_.

So let’s see how to display the current time here in Barcelona
(using the default time zone), the UTC time, and the time in New York.

Open your :red:`functional_tests/test_all_users.py` and write the following test:

.. code-block:: python

    def test_time_zone(self):
        self.browser.get(self.get_full_url("home"))
        tz = self.browser.find_element_by_id("time-tz").text
        utc = self.browser.find_element_by_id("time-utc").text
        ny = self.browser.find_element_by_id("time-ny").text
        self.assertNotEqual(tz, utc)
        self.assertNotIn(ny, [tz, utc])

we will check that the Barcelona, UTC and New York times are different.

Run this test and see it fail because the element with an id of
:orange:`time-tz` is missing. Open the :red:`index.html` template
and edit the third column:

.. code-block:: html

    <div class="col-md-4">
    <h2>Time Zones</h2>
    <p> Barcelona: <span id="time-tz">{{now|time:"H:i"}}</span></p>
    <p> UTC: <span id="time-utc">{{now|utc|time:"H:i"}}</span></p>
    <p> New York: <span id="time-ny">
           {{now|timezone:"America/New_York"|time:"H:i"}}</span></p>
    <p><a class="btn btn-default" href="#" role="button">View details &raquo;</a></p>
    </div>

Where I used the time filter to display only the time. Run the test again,
and it will fail because it can’t find the utc  filter. In your :red:`index.html`
file add at the top of the file:

.. code-block:: django

    {% load tz %}

Run your tests again and this time they will fail because the views didn’t pass
any :orange:`now` variable. Open the :red:`views.py` and add this import
at the beginning of the file:

.. code-block:: python

    from django.utils.timezone import now

and add the now variable into the home view:

.. code-block:: python

    def home(request):
        today = datetime.date.today()
        return render(request, "taskbuster/index.html",
                           {'today': today, 'now': now()})

Run your tests again, and they should pass!

Ok, this is the end of this part of the tutorial.
Don’t forget to commit your changes:

.. code-block:: console

    $ git add .
    $ git status
    $ git commit -m "Internationalization and localization"
    $ git push origin master

The last command only if you want to push it in your cloud
repository, like Bitbucket or GitHub.

In the next part of the tutorial, we’ll cover
:doc:`Documentation </part_6>`.
Yes, it may sound a boring subject to you,
but I’m sure that you really appreciate a package with a
understandable and well structured documentation, right?

Don’t miss it!

Do you like this tutorial? Don’t forget to share it with your friends! |:)|
Thanks!
