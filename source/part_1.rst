Part I – Working environment and start a Django Project
=======================================================
Building a good working environment is very important for your productivity. Maybe
it takes a while to configure, but when it’s well done it’s worth the time!

For example, virtual environments help you have your packages organized and controlled,
and a good text editor helps you with variable names, package methods, and syntax highlighting.

Oh, and If you didn’t read the introduction, this Django tutorial is up-to-date.
We will use Django 1.8 and Python 3. This doesn’t mean you can’t follow the tutorial
if you are using an older version. It only means that you will have to work harder
in your debugging, to solve problems I can’t anticipate here.

.. role:: red

Building your Working environment
---------------------------------
To follow this Django tutorial you will need :red:`Python3`, the package manager :red:`pip3`
and the following packages: :red:`virtualenv` and :red:`virtualenvwrapper`.

Install Django 1.8
------------------

Working directory and Sublime Text
----------------------------------

Obey the Testing Goat
---------------------

Create a Django project
-----------------------

Start a development server
--------------------------

.. code-block:: python

	from django import template
	from rango.models import Category

	register = template.Library()

	@register.inclusion_tag('rango/cats.html')
	def get_category_list():
	    return {'cats': Category.objects.all()}

This is a paragraph that contains `a link`_.

.. _a link: http://example.com/

Списки могут быть маркированными:

 * Элемент Foo
 * Элемент Bar

Или же автоматически пронумерованными:

 #. Элемент 1
 #. Элемент 2

Внутренняя разметка
------------––––––-
Слова можно выделять *наклонным* или **полужирным** шрифтами.
Фрагменты кода (например, примеры команд) можно заключать в обратные кавычки, например:
команда ``sudo`` дает вам привилегии суперпользователя!
