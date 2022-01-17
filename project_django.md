1. Запустить VS Code от имени администратора, перейти в каталог проекта в PowerShell,
выполнить код ниже, появится папка env, содержащая файлы виртуального окружения
`python -m venv env`

2. Изменить политику, в PowerShell набрать
`Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`

3. Войти в папку окружения (env), выполнить команду
`env\Scripts\activate.ps1`

4. Впереди в PowerShell появится маркер окружения (env), но VS Code может о нем все еще ничего не знать.
Нажать Ctrl+Shift + P, набрать `Python: Select Interpreter`
Указать нужный путь к python.exe в папке окружения env, это отобразится внизу в панели состояния.
Профит! Теперь можно устанавливать модули только для конкретного проекта.

5. Если нужно будет выйти, то в PowerShell выполнить `deactivate`, в выборе интерпетатора вернуться на глобальный.

```
pip install Django
pip list
pip help
django-admin #список команд
django-admin startproject name
python manage.py runserver 1.2.3.4:3000
python manage.py startapp name
```
********
файл settings
```
INSTALLED_APPS = [
    ...		,
    'news.apps.NewsConfig',]    #приложение news, файл apps, имя класса конфигурации NewsConfig
```
********
приложение views #any another name

	from django.http import HttpResponse
	# Create your views here.
	def index(request):  #контроллер функции
    		print(request)
    		return HttpResponse('Hello, mr.Sidr!')
********
файл urls

	from news.views import index, tests  	#для того, чтобы не перечислять все функции -> 	from news.views import *
	#импорт по ссылке, начиная с папки books(так как корневая папка например env), импорт контроллера функции index
	#нужно, чтобы корневой папкой была папка проекта

	urlpatterns = [
    		... 	,
    		path('news/', index),]		#адрес и функция,кот. по ней вызывается

''' 	#для того, чтобы не указывать "from news.views import * " ->

	from django.urls import path, include
	urlpatterns = [ ...	,
    		path('news/', include('news.urls')),]	 #адрес и функция,кот. по ней вызывается
'''
********
создаем в нашем приложении(news) файл urls.py
	from django.urls import path
	from .views import *

	urlpatterns = [
    		path('', index),
    		path('test/', tests),]
********
___блок по SQLlite___
файл models.py
	from django.db import models
	from django.db.models.fields import DateField

	# Create your models here.
	class News(models.Model):
    		title = models.CharField(max_length= 150)
    		content = models.TextField(blank=True)      #текст поля новости, blant True означает необязательно к заполнению
    		created_at = models.DateTimeField(auto_now_add=True)    #поле даты и времени. auto_now - время будет автоматически устанавливаться, после изменения исправляется;
    		#auto_now_add - устанавливается один первый раз, можно изменить
    		update_at = models.DateTimeField(auto_now=True)  #удобно для отслеживания времени сохранения новости
    		photo = models.ImageField(upload_to= 'photos/%Y/%m/%d/')    #поле для сохранения фото. Filefield - любой файл. imagefield - только фото
    		#upload_to - путь сохранения; photos/%Y/%m/%d/ - сортировка фото по дате создания. можно указать логику загрузки файла
    		is_published = models.BooleanField(default= True)    #отметка одобрена или нет/ опубликована или нет
*******
подготовка к миграции модели в базу данных
pip install Pillow	#команда для установки библиотеки поля ImageField
python manage.py makemigrations		#для указания конкретной модели через пробел указывается имя класса модели (пр. News)

для того, чтобы посмотреть sql запрос для создания таблицы в базе данных:
python manage.py sqlmigrate news/модудь/ 0001/порядковый номер/

в файле settings проекта есть функция
	DATABASES = { 			#здесь указана база данных
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',		#SQLite
        'NAME': BASE_DIR / 'db.sqlite3', } }		#вчастности константа BASE_DIR

#С создания проекта существует 18 неосуществленных миграций.
#Для выполнения этих миграций необ. команда:
python manage.py migrate
********
Для сохранения изображений в структуре данных необходимо настроить константу MEDIA_ROOT
в файле settings проекта
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')    #В папку media будут загружаться фото
MEDIA_URL = '/media/'

Чтобы просматривать изображения на сайте, необходимо создать соответствующий маршрут, кот. будет указывать на соотв. файлы.
Этот маршрут необходим только для отладочного сервера, который используется на этой стадии разработки проекта.
когда сайт перейдет на реальный веб сервер, этот маршрут уже не будет нужен.
#О том, что проект находится в отладочном режиме говорит константа DEBUG=True в файле settings проекта.
Для создания маршрута в файле urls проекта:
	from books import settings
	from django.conf.urls.static import static

	if settings.DEBUG:      #когда сервер перейдет на реальный DEBUG=False, а значит эти строки не нужны
    		urlpatterns += static(settings.MEDIA_URL, document_root = settings.MEDIA_ROOT)
*********
	ORM - object relative mapping
	CRUD	part 1
вся работа с ORM ведется через terminal powerShell python,
по команде python manage.py shell
>>>
далее необходим импорт
>>>from news.models import News
>>>News(title='Новости 1', content='контент новости 1')
<News: News object (None)>

#если проверить в БД эти данные, окажется, что они их там нет
#для добавления данных необ осуществить запрос на выполнение
>>>news1 = _
<News: News object (None)>

#При этом свойства объекта в БД существуют
НАПРИМЕР 	>>>news1.title
>>>news1.save() 	#сохраняет в БД данные, создается Id
>>>news1.id
1

#чтобы посмотреть список запросов к БД:
>>> from django.db import connection
>>> connection.queries
[{'sql': 'INSERT INTO "news_news" ("title", "content", "created_at", "update_at", "photo", "is_published")
VALUES (\'Новости 1\', \'контент новости 1\', \'2021-11-25 16:11:12.327471\', \'2021-11-25 16:11:12.327471\', \'\', 1)',
'time': '0.016'}]

|Рекомендация| primary key
>>>news1.pk
1

Пример заполнения контентом
пример 1
>>>news2 = News(title='Новости 2', content='контент новости 2')
>>> news2.save()

пример 2:
news3 = News()
news3.title = 'Новость 3'
news3.content = 'Контент Новости 3'
news3.save()

пример 3:
news4 = News.objects.create(title = 'Новость 4', content = 'Контент новости 4')

пример 4:
News.objects.create(title = 'News 5', content = 'News content  5')

******
	CRUD	part 2
from news.models import News
News.objects.all()
news = _ 	#присвоить предыдущее значение
for item in news:
...     print(item.title, item.is_published)

#поиск по заголовку title
News.objects.filter(title= 'News 5')

метод GET - возвращает конкретную запись например по pk или id
News.objects.get(pk=3)
News.objects.get(id=3)

исправлять содержание title
News.objects.get(id=3)
news = _
news.title = 'Исправленная новость 3'
news.save()

Удаление строки в БД:
news6 = News.objects.get(id=4)
news6.delete()

Сортировка по title:
News.objects.order_by('title') 		#алфавитный порядковый
News.objects.order_by('-title') 		#обратный алфавитный порядковый

Поиск с исклчениями:
News.objects.exclude(title='Новости 1')

#В случае объявления переменной в терминале запрос не будет выполнен пока переменная не была бы использована
пример
News.objects.exclude(title='Новости 1')		#результат запроса сразу
news_3 = News.objects.exclude(title='Новости 1')		#результат после использования переменной

******
В шаблонизаторе можно использовть теги, директивы и фильтры.
По умолчанию Джанго ищет шаблоны в папке Templates пакета приложения. Изначально такой папки нет. Django !рекомендует!
в папке templates создать доп. папку с именем приложения (news).
шаблон - это html файл. Для того чтобы подкл шаблон необходимо использовать 'from django.!shortcuts! import render'
	def index(request):  #контроллер функции, еще можно назвать ее экшеном - действием
    	news = News.objects.order_by('-created_at')		#параметр order_by('-created_at') позволяет сортировать новости по дате
    	context = {
        'news': news,
        'title': 'Список новостей'}
    return render(request, template_name='news/index.html', context=context ) 		#в шаблон index передается два ключа news и title

В самом шаблон файле index вставляем:
				<!DOCTYPE html>
				<html lang= "en">
				<head>
						<meta charset="UTF-8">
						<title>{{ title }}</title>
				</head>
				<body>
				<h1>{{ title }}</h1>
				{% for item in news %}
					<div>
						<p>{{ item.title }}</p>
						<p>{{ item.content }}</p>
						<p>{{ item.created_at | date:"Y-m H:i:s" }}</p>			# | date:"Y-m H:i:s такая запись является фильтром, в данном случае фильтром даты и времени
					</div>
					<hr>
				{% endfor %}
				</body>
				</html>

******************
Админка Django
******************
python manage.py createsuperuser

в приложении news файл admin.py
	from .models import News
	admin.site.register(News)   #регистрация приложения в проекте

в файле models в классе News:
class Meta:     #заменяет название приложения(News) на имя "Новость" в админке проекта
        verbose_name = 'Новость'
        verbose_name_plural = 'Новости'     #атрибут во множественном числе
        ordering = ['-created_at', '-title']    #сортировка по дате и времени, в случае совпадения времени сортировка по названию

в файле app.py добавляем
#class NewsConfig(AppConfig):
    #default_auto_field = 'django.db.models.BigAutoField'
    #name = 'news'
    !verbose_name = 'Новости'!	#меянет заголовок в админке с News на "Новости"

Далее в админке с отображением таблицы необходимо настроить отображение id, времени и др. параметров
from django.contrib import admin
from .models import News
class NewsAdmin(admin.ModelAdmin):
    list_display = ('id', 'title', 'created_at', 'update_at', 'is_published')
	list_display_links = ('id', 'title')
    search_fields = ('title', 'content')
    list_editable = ('is_published',)    #отвечает за возможность редактирования поля прямо в таблице админки
    list_filter = ('is_published',)      #фильтр по "опубликовано"
admin.site.register(News, NewsAdmin)   #порядокразрешения моделей, он важен      #регистрация приложения в проекте

для отображение корректных названий атрибутов в таблице (на русском) необходимо в файле models в классе News выставить параметр verbose_name=
class News(models.Model):
    title = models.CharField(max_length= 150, verbose_name= 'Наименование')
    content = models.TextField(blank=True, verbose_name= 'Контент')
    created_at = models.DateTimeField(auto_now_add=True, verbose_name= 'Опубликовано')
    update_at = models.DateTimeField(auto_now=True, verbose_name= 'Обновлено')
    photo = models.ImageField(upload_to= 'photos/%Y/%m/%d/', verbose_name= 'Фото')
    is_published = models.BooleanField(default= True, verbose_name= 'Опубликовано?')

для того, чтобы в таблице администрирования изменить гиперссылку на публикацию с id на title в файле admin +атрибут
~class NewsAdmin(admin.ModelAdmin):
    ~list_display = ('id', 'title', 'created_at', 'update_at', 'is_published')
    !list_display_links = ('id', 'title')!
	!search_fields = ('title', 'content')!		#создание Searchbar в таблице админки

!Валидация!
для того чтобы сделать поле необязательным необходимо добавить параметр blank=True.
photo = models.ImageField(upload_to= 'photos/%Y/%m/%d/', verbose_name= 'Фото', blank= True)
затем необходимо выпонить миграцию!
	#python manage.py makemigrations
	#python manage.py migrate

End Админка Django
******************


	Связи моделей
******************
в документации Джанго: "модели" - "поля отношений".
Нормализация данных, принцип нормализации. Данные должны быть структурированы в нескольких таблицах для удобства редактирования и структурирования.
	поля отношений
ForeignKey - наиболее популярный. Один ко многим. Многие к одному.
Пример: категория одна, относится к многим новостям.
Модель на которую ссылаются она первичная, а те кто ссылается - вторичная. в ForeignKey есть два аргумента(пр. on_delete).
В файле models указываем:
	class Category(models.Model):
		title = models.CharField(max_length=150, db_index=True, verbose_name= 'Наименование категории')
		def __str__(self):
			return self.title   #строковое представление
		class Meta:     #заменяет название приложения на имя "категория" в админке проекта
			verbose_name = 'Категория'
			verbose_name_plural = 'Категории'     #атрибут во множественном числе
			ordering = ['title']    #сортировка по названию
Эта модель Category является первичной, на нее будет ссылаться модель News, значит она должна быть вначале файла models.

Для связки двух моделей (двух таблиц) используем class ForeignKey. В моделе News(class News) указываем новый атрибут.
	class News(models.Model):
		...
		category = models.ForeignKey('Category', on_delete=models.PROTECT, null=True, ~default=1~, verbose_name= 'Категория')  # атрибут со ссылкой на модель Category. null=True позволит добавлять новые связи между моделями.
		# blank= True - поле будет необязательным для заполнения

Затем миграция.
Но при этом в ходе выполнения миграции возникнет ошибка
IntegrityError: The row in table 'news_news' with primary key '1' has an invalid foreign key
причина: в моделе category еще нет никаких данных, в то время как мы по умолчанию заполняем модель News категорией 1.
*Все миграции входят в папку приложения News/migrations, поэтому в случае удаления дефолтного параметра "категория 1" миграция выдаст ошибку, т.к. в ней сохранена предыдущая миграция.
*этот пример говорит о необхоимости проектирования моделей заранее. Способ устранения ошибки - удаление предыдущих миграций, содержащих ошибку и повторная исправленная миграция.

Настройка внешнего вида админки
	файл Admin
from .models import News, !Category!
class CategoryAdmin(admin.ModelAdmin):
    list_display = ('id', 'title')
    list_display_links = ('id', 'title')
    search_fields = ('title',)      #запятая необходима, т.к. "title" находится в кортеже, если запятой не будет, кортеж станет обычной строкой
admin.site.register(Category, CategoryAdmin)   #регистрация модели Категории

******************
Шаблоны. Внешний вид.

для тестового шаблона - https://getbootstrap.com/
в html:
	директива или переменная {{ title }}
	фильтр прописывается в директиве 		| date:"Y-m-d H:i:s"
	теги {% for item in news %}		{% endfor %}

пример файла-шаблона Index.html:
'''
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
    <title>{{ title }}</title>
  </head>
  <body>
    <nav class="navbar navbar-expand-lg navbar-light bg-light">
        <div class="container-fluid">
          <a class="navbar-brand" href="/">Segnarock</a>
          <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
          </button>
          <div class="collapse navbar-collapse" id="navbarSupportedContent">
            <ul class="navbar-nav me-auto mb-2 mb-lg-0">
              <li class="nav-item"><a class="nav-link active" aria-current="page" href="/">Главная</a>
              <li class="nav-item"><a class="nav-link active" aria-current="page" href="/">Добавить новость</a></li></li></ul></div></div>
    </nav>
<div class="container mt-3">
    <h1>{{ title }}</h1>
    <div class="row">
      <div class = "col-md-12">
        {% for item in news %}
        <div class="card mb-3">
            <div class="card-header">Категория: {{ item.category.id }} {{ item.category }}</div>
            <div class="card-body">
              <h5 class="card-title">{{ item.title }}</h5>
              <p class="card-text">{{ item.content }}</p>
              <a href="#" class="btn btn-primary">Read more...</a>
            </div>
            <div class="card-footer text-muted">{{ item.created_at | date:"Y-m-d H:i:s" }}</div>
        	</div>{% endfor %}</div>
		</div></div>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>
  </body>
</html>
'''
******************
Директивы, теги и фильтры
******************
в документации Джанго: 	the template layer - built-in template
в случае если в html вписать несуществующую переменную, то на страница по умолчанию будет пустая строка.
теги {%			%}
Наиболее популярные теги for, if
file model - class News
    def my_func(self):
        return 'Hello from model'

В случае если в БД попадет текст html кода он будет отображться строчно как есть. Для того, чтобы такой код работал
внутри БД необходимо этот код экранировать при помощи тега, пример:
  {% autoescape off %}
  <p class="card-text">		{{ item.content }}		</p>
  {% endautoescape %}

Для комментирования кода html используется тег comment:
	{% comment "переменная контента" %}
	<p class="card-text">		{{ item.content }}		</p>
	{% endcomment %}

Не всем тегам нужны {% end... %}, пр. cycle

Вывод изображения пример:
{%  if item.photo  %}
    <img src="{{ item.photo.url }}" alt="" width="350" class="mr-3">
{%  else %}
	<img src="https://picsum.photos/id/1060/350/235/?blur=3" alt="" class="mr-3">
{%  endif  %}

Фильтры позволяют редактировать вывод переменных {{	}}.

******************
Параметры в URL-запросах

Для того, чтобы работали ссылки по категориям:
файл views
from .models import News, !Category!
def index(request):  #контроллер функции, еще можно назвать ее экшеном - действием
    ...
    categories = Category.objects.all()
    ~context = {
        !'categories': categories!,	}
def get_category(request, category_id):
    news = News.objects.filter(category_id=category_id)
    categories = Category.objects.all()
    category = Category.objects.get(pk=category_id)
    return render(request, 'news/category.html', {'news': news, 'categories': categories, 'category': category} )

файл urls
urlpatterns = [
    ~path('', index),
    !path('category/<int:category_id>/', get_category),]     # int:category_id   объявление - целое число, пробелы в такой записи недопустимы

******************
Имена маршрутов

в файле urls добавляем имена, для того чтобы использовать в html шаблонах эти имена как ссылки. Имена используются благодаря тегу {% url 'Имя' %}
urlpatterns = [
    path('', index, !name='home'! ),
    path('category/<int:category_id>/', get_category, !name='category'! ),]     # int:category_id   объявление - целое число, пробелы в такой записи недопустимы. чтобы изменить ссылку менять только 'category/<int:category_id>/'
пример:
<a class="navbar-brand" href="{%  url 'home' %}">Segnarock</a>			#простая
<a href="{%  url 'category' item.category.pk %}">{{ item.category }}</a></div>		#в цикле for, где item итерация объекта news.

******************
Наследование шаблонов

Необходимые теги: block, extends, include
block. Существует возможность создания базового/родительского шаблона, который остальные шаблоны будут наследовать.
Род. шаблоны принято называть base.html. Более подробно в документации.
наследование осуществляется при помощи тега extend.
Директива базового шаблона должна распологаться в папке всего проекта. Books/templates/base.html
*Чем больше тегов block в шаблоне, тем лучше.

include. Этот тег позволяет разбить шаблон на части, на несколько файлов.
Имя такого файла шаблона принято называть наичная с нижнего подчеркивания _nav.html,
он должен хранится в Books/templates/include/_nav.html
{% include 'inc/_nav.html' %} вместо куска кода, который теперь хранится в _nav.html

Алгоритм
	1. Копируем полностью файk index в base.html
	2. Затем убираем блоки, которые остануться в index.
	3. В файле base вместо них указываем:
{%   block sidebar   %}SIDEBAR{%    endblock    %}
{%   block content   %}CONTENT{%    endblock    %}
	4. файл index теперь должен выглядеть так:

{% extends 'base.html' %}

{%   block title   %}   {{ title }} :: {{ block.super }}
{%    endblock    %}

{%   block sidebar   %}
<div class="col-md-3">
  <div class="list-group">
    {% for item_category in categories %}
    <a href="{%  url 'category' item_category.pk %}" class="list-group-item list-group-item-action">{{ item_category.title }}</a>
    {% endfor %}</div></div>
{%    endblock    %}

{%   block content   %}
<div class="col-md-9">
  {% for item in news %}
  <div class="card mb-3">
      <div class="card-header">Категория: <a href="{%  url 'category' item.category.pk %}">{{ item.category }}</a></div>
      <div class="card-body">
        <div class = "media">
            {%  if item.photo  %}
              <img src="{{ item.photo.url }}" alt="" width="350" class="mr-3">
            {%  else %}
              <img src="https://picsum.photos/id/1060/350/235/?blur=3" alt="" class="mr-3">
            {%  endif  %}
            <div class="media-body">
                    <h5 class="card-title  {%  cycle 'text-danger' 'text-success' %}">{{ item.title }}</h5>
                    {% autoescape off %}
                    <p class="card-text">{{ item.content|safe|linebreaks|truncatewords:50 }}</p>
                    {% endautoescape %}
                    <a href="#" class="btn btn-primary">Read more...</a></div></div></div>
      <div class="card-footer text-muted">{{ item.created_at }}</div>
  </div>{% endfor %}</div>
{%    endblock    %}

	5. Чтобы Django смог определить расположение base.html нужно в файле settings проекта указать директорию templates:
TEMPLATES = ...
        'DIRS': [	!os.path.join(BASE_DIR, 'templates')!	],


******************
Пользовательские теги шаблона

Django позвоялет создавать свои теги и фильтры.
Single tags
Inclusion tags
Для создания своих тегов создаем папку templatetags в папке приложения news, там создаем __init__.py
Далее templatetags/news_tags.py:
from django import template
from news.models import Category
register = template.Library()
@register.simple_tag()    #декоратор, возможность объединить поведение функции
def get_categories():
    return Category.objects.all()

В существующий шаблон импорт нового тега:	{%   load news_tags   %}
Данные, кот выводит этот тег передаем переменной categories:	{% get_categories as categories %}
Таким образом благодаря этому кастомному тегу и передаче данных в переменную categories можно избавиться
от этой переменной в функциях котроллеров файла views.

***************************
коммент
{#  Чтобы закомментировать строку  #}

{% comment %}
чтобы закомментировать блок кода
{% endcomment %}
***************************
Обратное разрешение адресов

алгоритм построения ссылок:
1. в файле urls создаем ссылку и даем ей имя, которое будем использовать в шаблоне
2. в шаблоне используем тег url, чтобы вызвать это имя
при этом в Джанго существует еще один способ построения ссылок, он рекомендуем Джанго

в файле models создаем новую модель класса Category
from django.urls import reverse
  def get_absolute_url(self):
    #необхоидим модуль из Django.urls reverse
    #в некоторых случаях джанго не сможет построить ссылку и необходимо использовать reverse_lazy
    #чтобы не путаться в вариантах можно использовать reverse_lazy всегда
    return reverse('category', kwargs= {"category_id": self.pk})    #в качестве первого праметра воспользуемся названием ссылки из файла urls,
      #вторым - словарь kwargs= {"category_id": self.pk} (из файла urls и views функции def get_category(request, category_id))
      #self.pk является необходимым параметром идентификации,по которому будет построена будущая ссылка
      #функция reverse является аналогией тега url. url используется в шаблонах, а reverse в python файлах.
затем я использую вместо тега
<a href="{% url 'category' item_category.pk %}"...>
следующий метод
<a href="{{ item_category.get_absolute_url}}" class="list-group-item list-group-item-action">{{ item_category.title }}</a>
где я обращаюсь к элементу item_category в списке categories/используя цикл/, а затем вызываю его метод item.get_absolute_url

Для создания ссылки:
1. прописываем маршрут в файле url
  path('news/<int:news_id>/', view_news, name='view_news'),   #маршрут ссылки
2. затем в файле views создаем функцию views_news
  def view_news(request, news_id):
    news_item = News.objects.get(pk=news_id)       #по первичному ключу pk достаем в БД необхожимый id
    return render(request, 'news/views_news.html', {"news_item":news_item})     #рендер шаблона
3. в папке приложения templates/news создаем файл views_news.html

{% extends 'base.html' %}
{%   block title   %}{{ news_item.title }} :: {{ block.super }}{%    endblock    %}
{%   block sidebar   %}{% include 'inc/_sidebar.html' %}{%    endblock    %}
{%   block content   %}
  <div class="card mb-3">
  <div class="card-header">
    Категория: <a href="{{ news_item.category.get_absolute_url}}">{{ news_item.category }}</a>
  </div>
  <div class="card-body">
    <div class = "media">
      {%  if news_item.photo  %}
      <img src="{{ news_item.photo.url }}" alt="" width="350" class="mr-3">
      {%  else %}
      <img src="https://picsum.photos/id/1060/350/235/?blur=3" alt="" class="mr-3">
      {%  endif  %}
      <div class="card-body">
        <div class="media-body">
          <h5 class="card-title  {%  cycle 'text-danger' 'text-success' %}">{{ news_item.title }}</h5>
          {% autoescape off %}
          <p class="card-text">{{ news_item.content|safe|linebreaks }}</p>
          {% endautoescape %}
        </div></div></div></div>
  <div class="card-footer text-muted">{{ news_item.created_at }}</div></div>
{%    endblock    %}

4. в файле models в классе News прописываем доп метод
  class News(models.Model):
    ...
    def get_absolute_url(self):
          return reverse('view_news', kwargs= {"news_id": self.pk})
5. ссылка создана, остается прописать эту сслыку в соответствующих шаблонах
  <a href="{{ item.get_absolute_url }}" class="btn btn-primary">Read more...</a>
  #кнопка Read more теперь ссылается на метод класса News

В случаях, когда ссылка не найдена. например такой новости не существует, ошибка будет выглядеть как
DoesNotExist at /news/8/
Для корректного отображения ошибки в виде 404 в файлк views
from django.shortcuts import get_object_or_404
def view_news(request, news_id):
    вместо этого #  news_item = News.objects.get(pk=news_id)       #по первичному ключу pk достаем в БД необхожимый id
    ! news_item = get_object_or_404(News, pk=news_id) !
    return render(request, 'news/views_news.html', {"news_item":news_item})     #рендер шаблона


***************************
Статические файлы

Джанго ищет все файлы в папке static пакета приложения. Для этой настройки в settings существует специальная константа STATIC_URL = '/static/', там же находится ссылка на документацию.
Отладочный сервер Джанго умеет обрабатывать статику в проекте, сторонний сервер такое не умеет, ему необходимо, чтобы все было в папке проекта
Для работы с одной папкой static в проекте, создаем папку static в папке с файлом настройки проекта
константа STATIC_URL будет являться префиксом к запрашиваем файлам

Необходимо добавить новые константы:
STATIC_ROOT = os.path.join(BASE_DIR, 'static')    #суть этой константы собрать всю статику из приложений в одно место
STATICFILES_DIRS = [    os.path.join(BASE_DIR, 'books/static'),  ] #перечисленные пути для файлов статики
Для выполнения переноса файлов необходима команда
python manage.py collectstatic
После выполнения команды в корне проекта будет создана папка static, в ней будут собраны файлы по определнным директориям
static/admin, /bootstrap, /css #директории созданы на основании той директории, что мы создали рядом с файлом settings
Далее необходимо указать, что мы хотим подключать файлы не с SDN, а локально. Дял этого нужно подключить спец тег static
в base.html прописываем:
{% load static %}
 <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
!   <link rel="stylesheet" href="{%  static 'bootstrap/css/bootstrap.min.css' %}">
!   <link rel="stylesheet" href="{%  static 'css/style.css' %}">
    <title>{%   block title   %} Новости со всего мира {%    endblock    %}</title>
</head>

***************************
Формы

Лучше использовать документацию Джанго.
Формы бывают связанными с моделью и не связанные.
формы связанные с моделью требует меньше кода.

ФОРМЫ НЕ СВЯЗАННЫЕ С МОДЕЛЬЮ/ START BLOCK
Для создания формы переходим в файл urls
urlpatterns = [
   ...
    path('news/add-news/', add_news, name='add_news'),  ]

создаем новый файл forms.py в директории приложения news, в котором практически дублируем все то, что прописывали в модели
from django import forms
from .models import Category
class NewsForm(forms.Form):
    title = forms.CharField(max_length=150, label='Название', widget=forms.TextInput(attrs={"class": "form-control"})) #
    content = forms.CharField(label='Текст', required=False, widget=forms.Textarea(attrs={
      "class": "form-control",
      "rows": 5
      }))  #required - обязательно ли заполнение формы, прохождение валидации
    is_published = forms.BooleanField(label='Опубликовано?', initial=True)     #булевый выбор, initial по умолчанию чек бокс будет отмеччен
    category = forms.ModelChoiceField(empty_label='Выберите категорию', queryset=Category.objects.all(), label='Категория:', widget=forms.Select(attrs={"class": "form-control"}))  # select

файл views
from .forms import NewsForm
+from django.shortcuts import ..., redirect
def add_news(request):
    if request.method == 'POST':
        form = NewsForm(request.POST)
        if form.is_valid():     #прошла ли форма валидацию
            #print(form.cleaned_data)        #словарь cleaned_data сохраняет данные о валидации
            News.objects.create(**form.cleaned_data)    #благодаря ** python распакует словарь для корректной отправки запроса в БД
            return redirect(news)  #библиотека Джанго redirect позволяет отправить на необх страницу (на страницу созданной новости)
            #return redirect('home')    #библиотека Джанго redirect позволяет отправить на необх страницу (имя страницы взято из urls приложения)
    else:
        form = NewsForm()
    return render(request, 'news/add_news.html', {'form': form})

существует три вида автоматического рендера формы в Джанго
as_p каждое поле будет заключено в параграф. основной
as_ul каждое поле будет элементом таблицы
as_table каждое поле будет элементом таблицы

в файл add_news.html в блоке content
<h1>Добавление новости</h1>
<form action="{% url 'add_news' %}" method="post">
    {{ form.as_p }}   #автоматический рендер формы
    <button type="submit" class="btn btn-primary btn-block">Добавить новость</button>
</form>

Если затем проверить исходный код страницы, то можно заметить, что
каждый элемент формы помещен в отдельный параграф. в каждом поле сущестувует атрибут required, для того, чтобы
поле было обязательно к заполнению.
!!!без этого формы рабоать не будет!!!Для того, чтобы формы могла внести изменения на сервер необходимо указать тег
{% csrf_token %}  #также благодаря этому токену проверяется поддлинность формы

Пробуем написать ручной рендер формы. /в файл add_news.html в блоке content
<div class="form-group">
    <label for="{{ form.title.id_for_label }}" class="form-label">Название: </label>
    {{ form.title }}
    <div class="invalid-feedback">  #вывод ошибок валидации Джанго после формы
      {{ form.title.errors }}
    </div>
Затем точно также для кнопки опубликовано и категории.


структура файла add_news.html выглядит следующим образом:
{% extends 'base.html' %}
{%   block title   %} Добавление новости :: {{ block.super }} {%    endblock    %}
{%   block sidebar   %}{% include 'inc/_sidebar.html' %}{%    endblock    %}
{%   block content   %}
<h1>Добавление новости</h1>
<form action="{% url 'add_news' %}" method="post">
    {% csrf_token %}
    {{ form.as_p }}   #один из способов вывода формы
<button type="submit" class="btn btn-primary btn-block">Добавить новость</button>
</form>
{%    endblock    %}


Для внесения опреденных атрибутов в поля финальной html страницы в файле forms.py в атрибуты полей указываем виджет с необходимыми настройками
widget=forms.TextInput(attrs={"class": "form-control"})

Три способа формирования формы
1: {{ form.as_p }}  #полностью автоматический

2:  #ручной
    перед формами указываем
    {{ form.non_field.errors }}   #Ошибки валидации не связанные с видимыми полями)

    <div class="form-group">
        <label for="{{ form.title.id_for_label }}">Название: </label>
        {{ form.title }}
        <div class="invalid-feedback">
            {{ form.title.errors }}   </div></div>

    <div class="form-group">
        <label for="{{ form.content.id_for_label }}">Текст: </label>
        {{ form.content }}
        <div class="invalid-feedback">
            {{ form.content.errors }}   </div></div>

    <div class="form-group">
        <label for="{{ form.is_published.id_for_label }}">Опубликовано? </label>
        {{ form.is_published }}
        <div class="invalid-feedback">
            {{ form.is_published.errors }}  </div></div>

    <div class="form-group">
        <label for="{{ form.category.id_for_label }}">Выберите категорию </label>
        {{ form.category }}
        <div class="invalid-feedback">
            {{ form.category.errors }}  </div></div>

3:   {% for field in form %}    #полуавтоматический
    <div class="form-group">
        {{ field.label_tag }}
        {{ field }}
        <div class="invalid-feedback">
            {{ field.errors }}
        </div></div>
    {% endfor %}

ФОРМЫ НЕ СВЯЗАННЫЕ С МОДЕЛЬЮ/ FINISH BLOCK

***************************
ФОРМЫ СВЯЗАННЫЕ С МОДЕЛЬЮ/ START BLOCK
На практике используются чаще

в файле forms.py
from django.forms import fields
from .models import News
#чтобы создать форму
class NewsForm(forms.ModelForm):
    class Meta:
        model = News    #атрибут model указывает с какой конкретно моделью связана наша форма
        #fields = '__all__'
        #атрибут fields указывает какие поля мы хотим видеть в нашей форме. '__all__' - означает весь перечень полей
        # #полей is_published и update_at в отображении не будет, т.к.они заполняются автоматически
        fields = ['title', 'content', 'is_published', 'category' ]  #лучше самостоятельно перечислить поля
        widgets = {
            'title': forms.TextInput(attrs={"class": "form-control"}),
            'content': forms.Textarea(attrs={"class": "form-control", 'rows': 5}),
            'category': forms.Select(attrs={"class": "form-control"})   }

в файле views.py
from django.shortcuts import render, get_object_or_404, redirect
from .models import News, Category
from .forms import NewsForm
def add_news(request):
    if request.method == 'POST':
        form = NewsForm(request.POST)
        if form.is_valid():     #прошла ли форма валидацию
            news = form.save()      #метод модели save позвоялет сохранить данные одной строкой
            return redirect(news)  #библиотека Джанго redirect позволяет отправить на необх страницу (на страницу созданной новости)
            #return redirect('home')    #библиотека Джанго redirect позволяет отправить на необх страницу (имя страницы взято из urls приложения)
    else:
        form = NewsForm()   #пустая форма не связанная с данными
    return render(request, 'news/add_news.html', {'form': form})

в файле models.py
в пункте поля category убирем атрибут null=True, его мы помещали разово, чтобы новость получала заполненную графу "категория"
далее выполним миграцию. во время миграции джанго самостоятельно предложит заполнить это значение по умолчанию,
а затем предложит ввести это значение. (вводи любую цифру, обозначающую категорию по id, например 1)

теперь данные валидируются на основе полей модели. т.е. формы свяазаны с моделью.
ФОРМЫ СВЯЗАННЫЕ С МОДЕЛЬЮ/ FINISH BLOCK

***************************
Кастомные валидаторы
Бывают случаи, когда валидации полей недостаточно, тогда необходимо валидировать дополнительно пользовательским валидатором
Валидатор - это метод класса формы вида "clean_" и "название поля" (Для title - clean_title).
Кастомная/пользовательская валидация проходит после основной.

в файле forms.py
import re
from django.core.exceptions import ValidationError
class NewsForm(forms.ModelForm):
  ...
  def clean_title(self):      #кастомный/пользовательский валидатор, self - объект данного класса
        title = self.cleaned_data['title']      #словарь с данными, где ключ 'title'
        if re.match(r'\d', title):   #поиск в строке title цифр. Используется бибилиотека regular expression
            raise ValidationError('Название не должно начинаться с цифры')       #возбуждение ошибки из библиотеки django.core.exception
        return title

*******************************************************
КЛАССЫ МОДУЛЯ GENERIC:
ListView, DetailView, CreateView
*******************************************************

Класс ListView
контроллеры класса
класс CBV - class based views в документации Джанго
До этого реализация проекта основывалась на контроллерах функций (def get_category, def add_news и др)

в файле views
from django.views.generic import ListView
class HomeNews(ListView):
    model = News    #аналог строки news = News.objects.all()

в файле urls
urlpatterns = [
    # path('', index, name='home'),
    path('', HomeNews.as_view(), name='home'),      #метод as_view
#после этого страница будет искать путь, который формирует из "названия приложения + названия модели в нижнем регистре" = news/news_list.html
т.к. такого пути нет, будет ошибка
если создать файл с таким названием в папке templates приложения, то страница будет загружена
При этом данные из БД передаются в объект по умолчанию object_list, который уже можно итерировать и заполнять страницу

Чтобы использовать свой шаблон, со своим названием, в файле views:
```
class HomeNews(ListView):
    ...
    !template_name = 'news/home_news_list.html'!  #название нового шаблона - home_news_list.html
    !context_object_name = 'news'!    #вместо объекта по умолчанию, в который записываются данные из БД, называем свой объект news = object_list
```
Необходимо создать этот новый файл home_news_list.html, а итерировать данные для отображения новостей следует из объекта news.

Для передачи в шаблоны своих переменных (используем атрибут extra_context, это словарь, либо def get_context_data) в файле views:
class HomeNews(ListView):
  ...
  #extra_context = {'title': 'Главная'}  # используется только для статичных данных
  def get_context_data(self, **kwargs):    #для передачи динамических переменных
    context = super().get_context_data(**kwargs)    # данные по умолчанию - словарь
    context['title'] = 'Главная страница'   # добавляем данные по умолчанию своей парой ключ-значение
    return context

Фильтрация данных из БД
в файле views:
class HomeNews(ListView):
  ...
  def get_queryset(self):
    return News.objects.filter(is_published=True)

Для категорий вместо def get_category(request, category_id) в файле views:
class NewsByCategory(ListView):    #способ передавать данные из БД (конкретная категория)
    model = News
    template_name = 'news/home_news_list.html'
    context_object_name = 'news'
    allow_empty = False     #атрибут который выводит ошибку 404, если такой страницы не найдено. без него выводится 505 ошибка

    def get_context_data(self, **kwargs):    #для передачи динамических переменных
        context = super().get_context_data(**kwargs)    # данные по умолчанию - словарь
        context['title'] = Category.objects.get(pk=self.kwargs['category_id'])   # добавляем данные по умолчанию своей парой ключ-значение. просиходит поиск названия категории через pk
        return context

    def get_queryset(self):     #для фильтрации данных из БД
        return News.objects.filter(category_id=self.kwargs['category_id'], is_published=True)   #вывод опубликованных новостей конкретной категории

в файле urls:
urlpatterns = [
    ...
    # path('category/<int:category_id>/', get_category, name='category'),     # int:category_id   объявление - целое число, пробелы в такой записи недопустимы. чтобы изменить ссылку менять только 'category/<int:category_id>/'
    path('category/<int:category_id>/', NewsByCategory.as_view(), name='category'),     # метод as_view. В него можно передать переменныеб например: as_view(extra_context={'title':'Какой-то тайтл'})

***************************
Класс DetailView

необходимо импортировать DetailView в файле views:
from django.views.generic import ..., DetailView
class ViewNews(DetailView):  #замена def view_news(request, news_id)
  model = News
  # template_name = 'news/news_detail.html'   #чтобы изменить имя шаблона по умолчанию
  # pk_url_kwarg = 'news_id'   #для перенаправления pk в news_id, который используется в файле urls при формировании ссылки

в файле urls:
urlpatterns = [
  ...
  # path('news/<int:news_id>/', view_news, name='view_news'),
  path('news/<int:news_id>/', ViewNews.as_view(), name='view_news'),  ]

Django поддерживает по умолчанию два  атрибута для связи pk и slug.
slug - способ генерации url строки
pk переходит от БД вв виде news_id.
Если загружать страницу джанго будет ссылаться на шаблон news_detail.html
код для этого шаблона представлен в шаблоне views_news.html

Для упрощения кода в ссылке обращаемся напрямую к pk. В файле urls:
urlpatterns = [
  ...
  path('news/<int:pk>/', ViewNews.as_view(), name='view_news'),   ]

после обращения к pk напрямую  в файлле models меняем kwargs={!"news_id"!: self.pk}:
class News(models.Model):
  ...
  def get_absolute_url(self):
    return reverse('view_news', kwargs= {"pk": self.pk})   #

***************************
Класс CreateView
необходимо импортировать CreateView в файле views:
from django.views.generic import ..., CreateView
class CreateNews(CreateView):
    form_class = NewsForm   #связывает данный класс CreateNews с классом формы NewsForm
    template_name = 'news/add_news.html'    #указываем существующий template
    # success_url = reverse_lazy('home')      #происходит редирект на главную страницу после добавления новости
    # reverse_lazy в отличии от reverse вызывается после того, как Джанго узнает о существовании маршрута 'home', поэтому не генерит ошибок

в классе News файла models существует метод def get_absolute_url(self), который уже вывполянет функцию reverse_lazy

в файле urls:
terns = [
  ...
  # path('news/add-news/', add_news, name='add_news'),
  path('news/add-news/', CreateNews.as_view(), name='add_news'),  ]  #для вызова конструторов класса, а не конструкторов функций

***************************
Добавление MySql
Заметка:
Джанго позвоялет менять БД
Для работы БД (пр. MySql) необходимо использование драйвера mysqlclient, возможны ошибки на некоторых машинах, будет необхожимо установить его вручную.
версия mysqlclient должна соответствовать версии пайтона
в файле settings:
DATABASES = {   'default': {
    'ENGINE': 'django.db.backends.mysql',
    'NAME': 'DjangoSQL1',
    'USER': 'root',
    'PASSWORD': '0000',
    'HOST': 'localhost'    } }

MySql command:
  use DjangoSQL1;
  SELECT * FROM таблица [FROM db_name]; - показать таблицу
  SHOW DATABASES; - список баз данных
  SHOW TABLES [FROM db_name]; -  список таблиц в базе
  SHOW COLUMNS FROM таблица [FROM db_name]; - список столбцов в таблице
  SHOW CREATE TABLE table_name; - показать структуру таблицы в формате "CREATE TABLE"
  SHOW INDEX FROM tbl_name; - список индексов
  SHOW GRANTS FOR user [FROM db_name]; - привилегии для пользователя.

  SHOW VARIABLES; - значения системных переменных
  SHOW [FULL] PROCESSLIST; - статистика по mysqld процессам
  SHOW STATUS; - общая статистика
  SHOW TABLE STATUS [FROM db_name]; - статистика по всем таблицам в базе

*На новых версиях MySql используется другой механизм хеирования, с версии 8.0
*а именно используется caching_sha2_password, т.к. Джанго не умеет рабоать с таким алгоритмом, вместо него лучше использовать алгоритм mysql_native_password

***************************
ORM (querysets)
Django ORM (Object Relational Mapping) является одной из самых мощных особенностей Django.
Это позволяет нам взаимодействовать с базой данных, используя код Python, а не SQL.

python manage.py shell    -запуск shell для работы с Джанго
from news.models import News, Category  -импорт моделей из приложения news и файла models

News.objects.order_by('pk')
News.objects.all().reverse  - обратный порядок
News.objects.get(pk=1) - для получения одной записи
News.objects.get(title='News 5') - для получения одной записи
news5 = _     присвоение последнего результата выборки переменной news5
news5.is_published -использование метода, описанного в models
news5.category    -т.к. у объекта есть строковый метод __str__ нам будет возвращен не id, а строкове представление объекта (title)
news5.category.pk
news5.category.title
!Напоминане! Существует понятие первичной и вторичной моделей, Category явл первичной, т.е. у News есть ссылка на Category.

cat4 = Category.objects.get(pk=4)
<имя связанной модели>_set    -получение данных от вторичной модели через первичную, путем связывания
пример: cat4.news_set.all(), мы получаем один объект с данными об этой категории, затем запускаем цикл для итерации этих новостей в этой категории
Если в атрибут Category класса News прописать related name = 'get_news':
  category = models.ForeignKey('Category', on_delete=models.PROTECT, verbose_name= 'Категория', related name = 'get_news')
то теперь необходимо заново импортировать измененный файл models и команда  <имя связанной модели>_set будет иметь вид:
  cat4 = Category.objects.get(pk=4)
  cat4.get_news.all()

*exit()  -выход из shell
***************************
ORM (querysets) 2 часть
Расширенная работа с запросами и фильтрами

операторы фильтрации
<имя поля>__<фильтр>
News.objects.filter(pk__gt=12)  -фильтр, который позвоялет сделать выборку по pk, все pk что больше 12
News.objects.filter(pk__gte=12)   - больше 12 либо равно
News.objects.filter(pk__lt=12)    меньше 12
News.objects.filter(pk__lte=12)   < либо равно 12
News.objects.filter(title__contains='new')    -поиск слова среди title, при использовании латиницы регистр не важен, при киррилице - важен
News.objects.filter(title__icontains='new')
News.objects.filter(pk__in=[1, 2, 5])   -поиск pk в определленом диапазоне, представленный в списке. pk=1, pk=2, pk=5
***************************
ORM (querysets) 3 часть

News.objects.first()    запись воввзращает первую запись набора
News.objects.order_by('-pk').first()

cat1 = Category.objects.get(pk=1) -данные по категории 1
news = cat1.news_set.filter(pk__gt=1)
news = cat1.news_set.last(pk__gt=1)

News.objects.earliest('updated_at')

cats = Category.objects.filter(pk__in=[1, 3])   получаем данные по двум категориям
cats    команда вывода
>>> QuerySet [<Category: Культура>, <Category: Политика>]   -те самые две категории
News.objects.filter(category__in=cats)    все новости в этих категориях

Как получить количество записей в наборе (например проверка категории на наличие записей)
>>>cat1 = Category.objects.get(pk=1)
>>>cat1 = Category.objects.get(pk=5)
>>>cat1.news_set.exists()    # True
>>>cat5.news_set.exists()    # False
>>>cat1.news_set.count()     # 5
>>>cat5.news_set.count()     # 5

kai = News.objects.get(pk=5)
kai.get_next_by_created_at()
kai.get_previous_by_created_at()
kai.get_previous_by_created_at(pk__gt=10, title__contains='новость')
***************************
ORM (querysets) 4 часть

фильтрация полей связанных по записи
<имя поля внешнего ключа>__<имя поля первичной модели>
News.objects.filter(category__title='Политика')   поиск данных по значению, а не по id

Category.objects.filter(news__title__contains='новости из')   #имя вторичной модели__title__фильтр containes=..

Класс Q
from Djangodb.models import Q
| логическое or
& логическое and
~ логическое not

т.е. запись News.objects.filter(pk__in=[5, 6], title__contains='2')   #категории 5 и 6,где в названии новости будет цифра 2
в такой записи возможно только логическое 'и'
при Q запись будет иметь вид: News.objects.filter(Q(pk__in=[5, 6]) | Q(title__contains='2'))

News.objects.filter(Q(pk__in=[5, 6]) | Q(title__contains='2') & ~ Q(pk__lt=4))

***************************
ORM (querysets) 5 часть
Агрегатные функции (агрегации) — это функции, которые вычисляются от группы значений и объединяют их в одно результирующее.
Если в поле с измерением добавить агрегацию, то поле становится показателем.

в файлк models
class News(models.Model):
  ...
  views = models.IntegerField(default=0, verbose_name= 'Просмотры')  #счетчик просмотров новостей

после этого необходимо осуществить миграцию
News.objects.all()[:3]    # получаем вывод записей по срезу, первые три
News.objects.all()[10:]    # или с 10

```
>>>from django.db.models import *
>>>News.objects.aggregate(Min('views'), Max('views'))
{'views__min': 0, 'view__max': 1000}

>>>News.objects.aggregate(min_views=Min('views'), max_views=Max('views'))
{'min_views': 0, 'max_views': 1000}

>>>News.objects.aggregate(diff = Min('views') - max_views=Max('views'))		#diff разница минимального и максимального значения
{'diff': -1000}

>>>News.objects.aggregate(Sum('views'))		# Sum сумма значений
{'views_sum': 3600}

>>>News.objects.aggregate(Avg('views')		# Avg среднее значение
{'views_avg': 240.3333333334}

>>>News.objects.aggregate(Count('views')		# количество значений
{'views_count': 15}
```
***************************
ORM (querysets) 6 часть
Аннотация по группе записей
Annotate

filter() и exclude()
Фильтры могут использоваться вместе с агрегацией. Любой filter() (или exclude()) повлияет на выборку объектов, используемых для агрегации.

from django.db.models import Count, Avg
cats = Category.objects.annotate(Count('news'))		#получаем список категорий
Для того, чтобы посмотреть количество новостей в каждой категории необходимо проитерировать объект cats.
```
>>>for item in cats:
...	print(item.title, item.news__count)
Культура 3
Политика 2
Наука 4
Спорт 0
```
```
# в случае
cats = Category.objects.annotate(cnt=Count('news'))
>>>for item in cats:
...	print(item.title, item.cnt)
```
Чтобы использовтаь фильтрацию для отображений списка категорий (в случае, когда новостей в определенной категории нет - нет необходимости эту категорию выводить в список) в файле news_tags.py:
```
from django.db.models import Count			#для работы с annotate

@register.inclusion_tag('news/list_categories.html')
def show_categories(arg1 = 'Hello', arg2 = 'Sold'):  #кастомный тег для рендера категорий
    #categories = Category.objects.all()
    !categories = Category.objects.annotate(cnt= Count('news')).filter(cnt__gt=0)!		#фильтр для пустых категорий
    return {"categories": categories, 'arg1': arg1, 'arg2': arg2 }
```
***************************
ORM (querysets) 7 часть
values()
values( * поля , ** выражения )
Возвращает те поля, которые указываются в выборке, остальные недоступны
```
>>>new1 = News.objects.values('title', 'views').get(pk=1)
>>>new1['title']
'Новость 1'
>>>new1['views']
'99'
>>>new1['id']
		KeyError
```
```
from django.db.models import connection
>>>news = News.objects.values('title', 'views', 'category__title')
>>>connection.queries		# список выполненных запросов в базу данных
результат в виде списка достаточно длинный
>>>from django.db import reset_queries		#библиотека по очистке этого списка
>>>reset_queries()		#вызов функции очистки
>>>connection.queries		#повторное отображение
[]		#пустой список
```
проверим какое количество выполнит Джанго SQl запросов
```
>>>for item in news:
...		print(item['title'], item['category__title'])
<название новости> + <ее категория>
<название новости> + <ее категория>
<название новости> + <ее категория>
```
В случае работы с БД обычным способом было бы выполнено около 16 запросов, при использовании values() всего один.
`connection.queries		#повторное отображение`
***************************
class F
`from django.db.models import F`
Иногда существует необходимость сравнить одно поле записи с другим полем, эту задачу и решает класс F.
```
#вариант подсчета количества просмотров
news = News.objects.get(pk=1)
news.views = F('views') + 1
```
Задача - поиск названия статьи в самих текстах новостей:
`News.objects.filter(content__icontains=F('title'))`
***************************
class Cast
В джанго существует возможность перенести вычисления с пайтона на СУБД.
***************************
ORM (querysets) 8 часть
Метод raw SQL выполение чистых SQL запросов
```
from news.models import *		# импортирование модели
News.objects.raw("SELECT * FROM news_news")		# по имени приложения и по имени модели, SELECT * FROM news_news является чистым SQL запросом
news = _		# перевод в переменную news данные полученные строкой выше
for item in news:
...	 	print(item.title, item.pk, item.is_published)
```
!Чтобы использование raw проходило корректно необходимо после SELECT указывать id (* означает выбор всех полей), без id/pk генерируется ошибка.
Запись вида:
```
news = News.objects.raw("SELECT title FROM news_news")
for item in news:
		print(item.title)
ОШИБКА
```
Корректная запись:
```
news = News.objects.raw("SELECT id, title FROM news_news")
for item in news:
		print(item.title)
Новость 3
Новость 4
и т.д.
```
Отоложенная загрузка полей - это тот случай, когда какое то поле не выгружено ддя предоставления, но обращаясь к нему СУБД загружает это поле. Главным полем все еще является id/pk.
Но при таком способе загрузки вместо одного сформированного запроса в БД, будет сгенировано количество для каждой новости в таблице отдельно.
Для формирования запроса с условием:
```
news = News.objects.raw("SELECT * FROM news_news WHERE title = 'News 5'")		#запрос с условием
for item in news:
		print(item.title)
```
Запись выше к сожалению уязвима для БД от различных атак. Чтобы исключить преднамеренные атаки или непреднамеренные следует использовать "параметры":
`news = News.objects.raw("SELECT * FROM news_news WHERE title = %s ", ['News 5'])`

***************************
Django Debug Toolbar - специальный пакет, который необходим на этапе разработки, чтобы отслеживать корректность работы приложения и оптимальность запросов в БД.
Существует документация

Установка
`pip install django-debug-toolbar`

далее в файле settings
```
INSTALLED_APPS = [
		...
		!'debug_toolbar',!
		#'news.apps.NewsConfig', ]   #приложение news, файл apps, имя класса конфигурации NewsConfig
```
```
MIDDLEWARE = [
    # ...
    "debug_toolbar.middleware.DebugToolbarMiddleware",
    # ...]
```
В конец файла settings
`INTERNAL_IPS = ["127.0.0.1"]`

В файле urls
```
if settings.DEBUG:      #когда сервер перейдет на реальный DEBUG = False, а значит эти строки не нужны
    ! urlpatterns = [
    path('__debug__/', include('debug_toolbar.urls')),
]   + urlpatterns   !
    urlpatterns += static(settings.MEDIA_URL, document_root = settings.MEDIA_ROOT)
```
на этом установка дебаг тулзы законечна, она будет отображаться на странице самого сайта проекта в браузере.

select_related и prefetch_related в Django
Большая проблема сайта это количество лишних запросов, сущесвует метод в Джанго, позволяющий загрузить связанные данные не отложенно, а сжато(загрузка одним запросом). метод select_related подходит для связи ForeignKey
Для связи many to many(многие ко многим) подойдет второй метод объединения запросов prefetch_related.

В файле views
```
class HomeNews(ListView):      #способ передавать данные из БД (все новости)
  ...
  def get_queryset(self):     #для фильтрации данных из БД
      return News.objects.filter(is_published=True).select_related('category')    #метод Джанго select_related необходим для загрузки связанных данных не отложенно, а сжато
```
Теперь SQL запросов в разы меньше

Такую же операцию выполним и на формировании страницы категории
```
class NewsByCategory(ListView):    #способ передавать данные из БД (конкретная категория)
  ...
  def get_queryset(self):     #для фильтрации данных из БД
      return News.objects.filter(category_id=self.kwargs['category_id'], is_published=True).select_related('category')    #вывод опубликованных новостей конкретной категории
      #метод Джанго select_related необходим для загрузки связанных данных не отложенно, а сжато
```
***************************
Кастомизация админки
Admin, больше в документации Джанго

Благодаря Debug Toolbar можем просмотреть из каких шаблонов состоит Админка. В папке проекта templates создаем папку admin, в которой дублируем файл шаблона base_site.html. Теперь проект основывается на этот шаблон, который мы можем настраивать по своему усмотрениюю

в файл base_site.html помещаем блок
```
{% load static %}
{% block extrastyle %}
    <link rel="stylesheet" href="{% static 'css/admin.css' %}">
{% endblock %}
```
в котором указываем css файл со своим стилем. рядом с файлом settings В папке проекта static помещаем файл admin.css

т.е. например мы просматриваем код админки, находим интересующийся нас элемент и указываем его в своем стиле css:
```
#header{                      #цвет бэкграунда сайдбара
    background: #90a4ae; }
#branding h1, #branding h1 a:link, #branding h1 a:visited {       #цвет текста сайдбара
    color: #fff;  }
```
Для того, чтобы добавить возможность увидеть картинку в списке новостей в Джанго существует поле fieldsets
в файле admin
```
from django.utils.safestring import mark_safe

class NewsAdmin(admin.ModelAdmin):
  list_display = ('id', 'title', 'category', 'created_at', 'update_at', 'is_published', 'views', 'get_photo')   #get_photo вызывает метод
  # в случае если вместо метода get_photo написать атрибут photo, в админке он будет представлен просто ссылкой
  ...

  def get_photo(self, obj):
    if obj.photo:
      return mark_safe(f'<img src="{obj.photo.url}" width="75">')   #чтобы передавать в новости html код картинки для отображения
    else:
      return 'Фото не установлено'

  get_photo.short_description = 'Миниатюра'   #для переименования названия get_photo в таблице новостей
```
При это стоит помнить, что не все новости содержат картинки, вместо пустующих вставляются фото по умолчанию.

Для того, чтобы фото выводилось в странице создания/редактирования новости:
```
class NewsAdmin(admin.ModelAdmin):
    ...
    fields = ('title', 'category', 'content', 'photo', 'get_photo', 'is_published', 'views', 'created_at', 'update_at')
    readonly_fields = ('get_photo', 'views', 'created_at', 'update_at')     #указывает какие поля нельзя будет редактировать, т.к. некоторые вообще создаются автоматически
    save_on_top = True  #для удобства дублирует панель сохранения в вверх
```
***************************
Миксины(примеси) - возможность наследовать одним классом другой класс
*пайтон в отличии от php подерживает множественное наследование.
в папке приложения news создаем файл utils.py
*Для примера реализуем простую операцию перевода строки в верхний регистор
```
class MyMixin(object):
    mixin_prop = ''     #свойство, которое позже будем менять

    def get_prop(self):
        return self.mixin_prop.upper()

    def get_upper(self, s):
        if isinstance(s, str):
            return s.upper()
        else:
            return s.title.upper()
```
Mixin пригодится только дял классов, если используются функции, то уже понадобятся декораторы.
Например: Чтобы доступ к странице добавления новостей имел только админ необходимо использовать login_requuired (документация Джанго).
Для выполнения такой задачи для миксина потребуется LoginRequiredMixin.
Сперва уберем элементы на сайте для неавторизованных пользователей
```
{% if request.user.is_authenticated %}
  #выводим кнопку "Добавить новость"
  <li class="nav-item"><a class="nav-link active" aria-current="page" href="{%  url 'add_news' %}">Добавить новость</a>
{%  endif %}
```
Но при этом доступ к ссылке через адресную строку будет доступен, таск выполняется на уровне views:
`from django.contrib.auth.mixins import LoginRequiredMixin`
Существует два варианта действия для неавторизованных пользователей:
```
class CreateNews(LoginRequiredMixin, CreateView):
    ...
    login_url = '/admin'    # 1. для неавторизованных пользователей ресурс перенаправит на страницу авторизования
    #raise_exception = True     # 2. для неавторизованных пользователей выдаст ошибку "403 Forbidden"
```
***************************
Pagination(Пагинация) - разбивка Quereset, постраничная разбивка контента, добавление элементов взаимодействия
Часть 1

Django предоставляет методы высокого и низкого уровня, которые помогают разбивать данные на страницы, то есть данные, которые разделены на несколько страниц, с помощью ссылок «Предыдущий / Следующий».
Класс Paginator
Под капотом все методы разбиения на страницы используют этот класс Paginator. Он выполняет всю работу по разделению объекта QuerySet на объекты Page.
```
from django.core.paginator import Paginator
objects = ['john', 'paul', 'george', 'ringo']
p = Paginator(objects, 2)
```
Пагинацию можно использовать в классе наследнике ListView
в файле views
```
from django.core.paginator import Paginator
def test(request):
    objects = ['john1', 'paul2', 'george3', 'ringo4', 'john5', 'paul6', 'george7', 'ringo8']
    paginator = Paginator(objects, 3)
    page_num = request.GET.get('page', 1)   # номер текущей страницы из массива request, если номера страницы нет в масииве по умолчанию - 1
    page_objects = paginator.get_page(page_num)
    return render(request, 'news/test.html', {'page_obj': page_objects})
```
в файле urls
```
urlpatterns = [
  path('test/', test, name='test'),   #пагинация контента
  ...
```
в файл основного шаблона base.html добавялем кнопки из бутстрапа (папка templates)
```
{% if page_obj.has_other_pages %}   #треубует ли контент разбивки на страницы?
       <nav aria-label="...">
         <ul class="pagination">

           {%  if page_obj.has_previous %}  #если сущестувует предыдщуая страница
           <li class="page-item">
             <a class="page-link" href="?page={{  page_obj.previous_page_number  }}">Назад</a>
           </li>
           {% endif %}

           {%   for p in page_obj.paginator.page_range %}   #для всего контента
           {%  if page_obj.number == p %}
             <li class="page-item active" aria-current="page">
               <a class="page-link" href="?page={{  p  }}">{{  p  }}</a>
             </li>
           {%  elif p > page_obj.number|add:-3 and p < page_obj.number|add:3  %}
             <li class="page-item">
               <a class="page-link" href="?page={{  p  }}">{{  p  }}</a>
             </li>
           {%   endif  %}
           {%  endfor %}

           {%  if page_obj.has_next %}  #если сущестувует следующая страница
           <li class="page-item">
             <a class="page-link" href="?page={{  page_obj.next_page_number  }}">Вперед</a>
           </li>
           {% endif %}

         </ul>
       </nav>
       {%  endif %}
```
Реализация пагинации в виде класса.
Часть 2
Просто вводи в класс вывода контента объект пагинации по умолчанию
Файл views
```
class HomeNews(ListView):
    ...
    paginate_by = 2     #пагинация Джанго "из коробки"

```
***************************
Регистрация Джанго. Часть 1

Как правило ддя регистрациии создается отдельное приложение, но в целях обучения все выполним в текущем.
в файле views
```
def register(request):
    return render(request, 'news/register.html')
def login(request):
    return render(request, 'news/login.html')
```
в файле urls
```
urlpatterns = [
    path('register/', register, name='register'),
    path('login/', login, name='login'),
```
в папке templates/news создаем два файла login.html и register.html


в файле views
from django.contrib.auth.forms import UserCreationForm  #авторизация

в файле register.html
```
{% extends 'base.html' %}

{%   block title   %}
Regiser :: {{ block.super }}
{%    endblock    %}

{%   block sidebar   %}
{% include 'inc/_sidebar.html' %}
{%    endblock    %}

{%   block content   %}
<h1>Регистрация</h1>

<form method="post">
    {% csrf_token %}    #благодаря этому токену дял безопасности джанго будет проверять данные, получаемые из формы
    {{ form.as_p }}   элементы формы будут отдельно в параграфе
</form>

{%    endblock    %}
```
Для обратной связи с пользователями в Джанго существует компонент messages. Благодаря нему мы сможем вывести сообщение об успешной регистрации.
как работает:
сперва импорт
`from django.contrib import messages #вывод сообщений`
пример: Затем использовать метод add_message либо его урощенную версию
`messages.success(request, 'Вы успешно прошли регистрацию')`

Строки кода в шаблоне base.html
```
<div class="col-md-9">
         {%  if messages %}
         {%  for message in messages %}
           {%  if message.tags == "error" %}
           <br>
           <div class="alert alert-danger" role="alert">{{ message }}</div>
           {%  else  %}
           <div class="alert alert-{{ message.tags }}" role="alert">{{ message }}</div>
           {%  endif %}
         {%  endfor  %}
         {%  endif  %}
       </div>
```

Регистрация Джанго. Часть 2
В джанго существует стандартная модель User, которой зачастую хватает. Ее и будем использовать.

Переносим UserCreationForm из views в forms, создаем новый класс формы UserRegisterForm, который унаследуем от UserCreationForm
```
from django.contrib.auth.forms import UserCreationForm  #авторизация
from django.contrib.auth.models import User

class UserRegisterForm(UserCreationForm):
    email = forms.EmailField()
    class Meta:
        model = User    #связывает нашу модель с моделью User (from django.contrib.auth.models)
        fields = ('username', 'email', 'password1', 'password2')
```
в файл views импортируем из файла forms класс UserRegisterForm
`from .forms import NewsForm, !UserRegisterForm!`
и заменяем им UserCreationForm в классе register файла views.
в файле forms дополняем класс UserRegisterForm, где настраиваем формы:
```
class UserRegisterForm(UserCreationForm):
    email = forms.EmailField()
    #настройка в следующим виде работает некорректно
    '''
    widgets = {
        'username': forms.TextInput(attrs={"class": "form-control"}),
        'email': forms.EmailInput(attrs={"class": "form-control"}),
        'password1': forms.PasswordInput(attrs={"class": "form-control"}),
        'password2': forms.PasswordInput(attrs={"class": "form-control"})
    }'''
    #тонкую настройку каждой формы можно осуществить следующим образом:
    username = forms.CharField(label='Имя пользователя', widget=forms.TextInput(attrs={"class": "form-control"}))
    password1 = forms.CharField(label='Пароль', widget=forms.PasswordInput(attrs={"class": "form-control"}))
    password1 = forms.CharField(label='Подтверждение пароля', widget=forms.PasswordInput(attrs={"class": "form-control"}))
    email = forms.EmailField(label='E-mail', widget=forms.EmailInput(attrs={"class": "form-control"}))

    class Meta:
        model = User    #связывает нашу модель с моделью User (from django.contrib.auth.models)
        fields = ('username', 'email', 'password1', 'password2')
```
В поле Имя пользователя атрибут "автофокус" не убирается, возможно этот баг устранится с обновлением Джанго. т.е. курсор автоматически стоит в первом поле.
Help text после ручной настройки не отображается. Для его отображения необходимо ввести атрибут "help_text ='Подсказка бла-бла'"
`username = forms.CharField(label='Имя пользователя', widget=forms.TextInput(attrs={"class": "form-control"}), help_text ='Подсказка бла-бла')`
в файле views
```
def register(request):
    if request.method == 'POST':    #проверка - пришел ли запрос страницы методом POST
        form = UserRegisterForm(request.POST)   #связываем с формой
        if form.is_valid():     #валидация формы, проверка на подлинность
            form.save()     #сохраняем пользователя
            messages.success(request, 'Вы успешно прошли регистрацию')
            return redirect('login')
        else:
            messages.error(request, 'Ошибка регистрации')
    else:
        form = UserRegisterForm()   #если страница запрошена методом GET,то создаем экземпляр формы, не связанный с данными
    return render(request, 'news/register.html', {'form': form})    # {'form': form} передаем контекст в переменную form

def login(request):
    return render(request, 'news/login.html')
```
***************************
Авторизация
Процесс разбит на несколько этапов, таких как аутентификация(проверка есть ли такой пользователь с таким паролем) и авторизация(вход доступа к ресурсу).

в файле forms
```
from django.contrib.auth.forms import UserCreationForm, AuthenticationForm

lass UserLoginForm(AuthenticationForm):
    username = forms.CharField(label='Имя пользователя', widget=forms.TextInput(attrs={"class": "form-control"}))
    password = forms.CharField(label='Пароль', widget=forms.PasswordInput(attrs={"class": "form-control"}))
```
в файле views изменим класс login
```
from .forms import NewsForm, UserRegisterForm, !UserLoginForm!
from django.contrib.auth import login, logout

def user_login(request):
    if request.method == 'POST':    #проверка - пришел ли запрос страницы методом POST
        form = UserLoginForm(data=request.POST)   #в данном случае просто передать данные не выйдет, необходимо назначить переменную, в которую поместим эти данные
        if form.is_valid():     #валидация формы, проверка на подлинность
            user = form.get_user()
            login(request, user)
            messages.success(request, 'Вы успешно авторизовались')
            return redirect('home')
    else:
        form = UserLoginForm() #если страница запрошена методом GET,то создаем экземпляр формы, не связанный с данными
    return render(request, 'news/login.html', {'form': form})   # {'form': form} передаем контекст в переменную form
```
в файле urls изменим название пути
```
urlpatterns = [
    ...
    path('login/', !user_login!, name='login'),
```
в отличии от класса UserCreationForm в классе AuthenticationForm нет наследования к классу Meta, таким образом в предыдущем примере мы доплняли класс UserRegisterForm классом Meta, то класс UserLoginForm в этом не нуждается.
в файле login.html
```
{% extends 'base.html' %}

{%   block title   %}
Login :: {{ block.super }}
{%    endblock    %}

{%   block sidebar   %}
{% include 'inc/_sidebar.html' %}
{%    endblock    %}

{%   block content   %}
<h1>Авторизация</h1>
<form method="post">
    {% csrf_token %}  #благодаря этому токену дял безопасности джанго будет проверять данные, получаемые из формы
    {{ form.as_p }}   элементы формы будут отдельно в параграфе
    <button type="submit" class="btn btn-info btn-block">Login</button>
</form>
{%    endblock    %}
```
Чтобы после регистрации пользователю не вводить данные заново в целях авторизации, можно авторизовать в момент регистрации.
```
def register(request):
    if request.method == 'POST':    #проверка - пришел ли запрос страницы методом POST
        form = UserRegisterForm(request.POST)   #связываем с формой
        if form.is_valid():     #валидация формы, проверка на подлинность
!           user = form.save()     #сохраняем пользователя и передаем в переменную user
!           login(request, user)    #авторизуем, передавая request и данные пользователя
...
```
Добавим возиожность выйти из аккаунта и добавим информацию об авторизованном пользователе на страницу.
в файле views
```
def user_logout(request):
    logout(request)
    return redirect('login')
```
в файле urls
```
urlpatterns = [
    ...
    path('logout/', user_logout, name='logout'),
```
в шаблоне _nav.html
```
<span class="navbar-text">
  {% if request.user.is_authenticated %}
    Добро пожаловать, {{ user.username }} | <a href="{%  url 'logout' %}">Выход</a>
  {%  else %}
    <a href="{%  url 'register' %}">Регистрация</a> | <a href="{%  url 'login' %}">Авторизация</a>
  {%  endif %}
</span>
```
***************************
Визуальный редактор Django ckeditor - для редактирования и форматирования текста внутри сайта

1. pip install django-ckeditor
2. Add ckeditor to your INSTALLED_APPS setting.
3. Run the `collectstatic` management command: python manage.py collectstatic.
This will copy static CKEditor required media resources into the directory given by the STATIC_ROOT setting. /использовать параметр `STATIC_ROOT = os.path.join(BASE_DIR, 'static')` #суть этой константы собрать всю статику из приложений в одно место/
4. константа `CKEDITOR_UPLOAD_PATH = "uploads/"` куда CKEditor будет загружать изображение  - в файл settings
Add ckeditor_uploader to your INSTALLED_APPS setting.
5. добавить CKEditor URL в файл urls.py проекта:
`path('ckeditor/', include('ckeditor_uploader.urls')),`
6. Встраивание виджета ckeditor
в файл admin
```
from django import forms
from ckeditor_uploader.widgets import CKEditorUploadingWidget

class NewsAdminForm(forms.ModelForm):   #редактирование контента пользвателю из сайта
    content = forms.CharField(widget=CKEditorUploadingWidget()) #переопределение поля контента из файла models(контент новости)

    class Meta:
        model = News
        fields = '__all__'
```
7. также класс формы, который указан в пунке 6. нужно указать в классе приложения NewsAdmin файла Admin
```
class NewsAdmin(admin.ModelAdmin):
    form = NewsAdminForm
    ...
```
8. Конфигурация к существующему набору рредактирования через админа, просто добавляем в файл settings
```
CKEDITOR_CONFIGS = {
    'default': {
        'skin': 'moono',
        # 'skin': 'office2013',
        'toolbar_Basic': [
            ['Source', '-', 'Bold', 'Italic']
        ],
        'toolbar_YourCustomToolbarConfig': [
            {'name': 'document', 'items': ['Source', '-', 'Save', 'NewPage', 'Preview', 'Print', '-', 'Templates']},
            {'name': 'clipboard', 'items': ['Cut', 'Copy', 'Paste', 'PasteText', 'PasteFromWord', '-', 'Undo', 'Redo']},
            {'name': 'editing', 'items': ['Find', 'Replace', '-', 'SelectAll']},
            {'name': 'forms',
             'items': ['Form', 'Checkbox', 'Radio', 'TextField', 'Textarea', 'Select', 'Button', 'ImageButton',
                       'HiddenField']},
            '/',
            {'name': 'basicstyles',
             'items': ['Bold', 'Italic', 'Underline', 'Strike', 'Subscript', 'Superscript', '-', 'RemoveFormat']},
            {'name': 'paragraph',
             'items': ['NumberedList', 'BulletedList', '-', 'Outdent', 'Indent', '-', 'Blockquote', 'CreateDiv', '-',
                       'JustifyLeft', 'JustifyCenter', 'JustifyRight', 'JustifyBlock', '-', 'BidiLtr', 'BidiRtl',
                       'Language']},
            {'name': 'links', 'items': ['Link', 'Unlink', 'Anchor']},
            {'name': 'insert',
             'items': ['Image', 'Flash', 'Table', 'HorizontalRule', 'Smiley', 'SpecialChar', 'PageBreak', 'Iframe']},
            '/',
            {'name': 'styles', 'items': ['Styles', 'Format', 'Font', 'FontSize']},
            {'name': 'colors', 'items': ['TextColor', 'BGColor']},
            {'name': 'tools', 'items': ['Maximize', 'ShowBlocks']},
            {'name': 'about', 'items': ['About']},
            '/',  # put this to force next toolbar on new line
            {'name': 'yourcustomtools', 'items': [
                # put the name of your editor.ui.addButton here
                'Preview',
                'Maximize',

            ]},
        ],
        'toolbar': 'YourCustomToolbarConfig',  # put selected toolbar config here
        # 'toolbarGroups': [{ 'name': 'document', 'groups': [ 'mode', 'document', 'doctools' ] }],
        # 'height': 291,
        # 'width': '100%',
        # 'filebrowserWindowHeight': 725,
        # 'filebrowserWindowWidth': 940,
        # 'toolbarCanCollapse': True,
        # 'mathJaxLib': '//cdn.mathjax.org/mathjax/2.2-latest/MathJax.js?config=TeX-AMS_HTML',
        'tabSpaces': 4,
        'extraPlugins': ','.join([
            'uploadimage', # the upload image feature
            # your extra plugins here
            'div',
            'autolink',
            'autoembed',
            'embedsemantic',
            'autogrow',
            # 'devtools',
            'widget',
            'lineutils',
            'clipboard',
            'dialog',
            'dialogui',
            'elementspath'    ]),   }     }
```
Можно менят скины редакторы в сроке `'skin': 'moono',` например из установленных по умолчанию mono-lisa
если кастомный, то заменяем строку на `'config.skin': 'bootstrapck',`

***************************
Капча
Форма обратной связи

pip install django-simple-captcha

в файле settings
```
INSTALLED_APPS = [
    ...
    'captcha',
```
python manage.py migrate

Добавить urls.py проекта
```
urlpatterns += [
    path('captcha/', include('captcha.urls')), ]
```
Затем в файл forms
```
from django import forms
from captcha.fields import CaptchaField
```
А затем добавляем форму капча в нужные страницы, например страницы авторизации
```
class UserLoginForm(AuthenticationForm):
    username = forms.CharField(label='Имя пользователя', widget=forms.TextInput(attrs={"class": "form-control"}))
    password = forms.CharField(label='Пароль', widget=forms.PasswordInput(attrs={"class": "form-control"}))
    !captcha = CaptchaField(widget=CaptchaTextInput)!
```
***************************
Кэширование

Кэширование страницы:
В папке проекта создаем папку для кэша
django_cache

в файле settings
```
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.filebased.FileBasedCache',
        'LOCATION': os.path.join(BASE_DIR, 'django_cache'),    }   }
```
в файле urls добавляем возможность кэширования главной страницы на 20 сек
```
from django.views.decorators.cache import cache_page
  ...
  path('', cache_page(20) (HomeNews.as_view()), name='home'),   #20 - секунд до обновления
```

2 способ - Кэширование фрагментов шаблона:
Для этого нужно импортировать соответствующий тег
```
{% load cache %}
{% cache 500 sidebar %}   #500 - время в сек
    .. sidebar ..
{% endcache %}
```

3 способ - API низкого уровня для кэширования:
Допустим, ваш сайт содержит представление, итог работы которого зависит от нескольких ресурсоёмких запросов, результаты которых периодически изменяются. В этом случае, кэширование всей страницы будет только вредить, но вот кэширование редко изменяемых фрагментов было бы хорошим подходом.
Предлжен ряд функций для работы с кэшем, основные: set и get
set устанвливает кэш
`cache.set('my_key', 'hello, world!', 30)`
my_key - название кэша
hello, world! - что помещаем в кэш
30 - время кэширования в сек
Методом get передаем ключ, по которому можем забрать ранее заложенный кэш.
```
cache.get('my_key')
'hello, world!'
```
Пример в файле news_tags.py
```
from django.core.cache import cache

@register.inclusion_tag('news/list_categories.html')
def show_categories(arg1 = 'Hello', arg2 = 'Sold'):  #кастомный тег для рендера категорий
    categories = cache.get('categories')    #пытаемся получить данные из кэша
    if not categories:  #если их там нет
        #categories = Category.objects.all()
        categories = Category.objects.annotate(cnt= Count('news', filter=F('news__is_published'))).filter(cnt__gt=0)    #фильтр пустой категории, filter=F('news__is_published') - это фильтр для не опубликованных новостей в листе категорий
        cache.set('categories', categories, 30)     #создаем кэш из БД
    return {"categories": categories, 'arg1': arg1, 'arg2': arg2 }
```






.
