**Django Rest Framework**



`pip install djangorestframework`
затем регистрируем DRF в django
в файле settings
```
INSTALLED_APPS = [
  ...
    'rest_framework',
]
```
***
*Используемая модель*
```
from django.db import models
from django.contrib.auth.models import User

class UserProfile(models.Model):
    username = models.CharField(max_length=100)
    user = models.OneToOneField(User, on_delete=models.CASCADE, blank=True, null=True)  #on_delete=models.CASCADE означает, что при удалении пользователя будут удаляться все записи с этим пользователем
    discription = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    is_published = models.BooleanField(default=True)
    cate = models.ForeignKey('Category', on_delete=models.PROTECT, null=True)

    def __str__(self):
        return self.username

class Category(models.Model):
    title = models.CharField(max_length=100, db_index=True)

    def __str__(self):
        return self.title
```

***
**Настройки DRF**

файл settings
```
REST_FRAMEWORK = {      #настройки DRF
    'DEFAULT_RENDER_CLASSES': [
        'rest_framework.renderers.JSONRenderer',        #используемый сериализатор
        'rest_framework.renderers.BrowsableAPIRenderer',    #html шаблон, где отображаются данные по запросу, при его отключении данные представляются в виде строки формата JSON
    ] }
```
***

Создание представления (файл views):
```
from django.shortcuts import render
from rest_framework import generics

from .models import UserProfile
from .serializers import UserProfileSerializer
# Create your views here.


class UserProfileAPIView(generics.ListAPIView):    #формируем представление
    queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer
```
создание сериализатора(создаем файл serializers.py):
```
from rest_framework import serializers
from .models import UserProfile

class UserProfileSerializer(serializers.ModelSerializer):
    class Meta:
        model = UserProfile
        fields = ('username', 'cate', 'created_at', 'discription')  #согласно модели, что выводить пользователю
```
маршрутизация (файл urls проекта):
```
from drf_app.views import UserProfileAPIView

urlpatterns = [
    ...
    path('api/v1/userlist/', UserProfileAPIView.as_view()),   #обтявление api с его версией /v1, затем суть запроса /userlist
]
```
***
**метод APIView без автоматического сериализатора**

файл views:
```
from django.forms import model_to_dict  #преобразование данных в словарь
from rest_framework.views import APIView
from rest_framework.response import Response
from .models import UserProfile
# Create your views here.

class UserProfileAPIView(APIView):  #используя APIView возможно более тонко настроить обработку запросов от клиента серверу
    #необходимые проверки этих запросов происходят автоматически
    def get(self, request):  #метод для обработки GET запроса
        lst = UserProfile.objects.all().values()        # без метода .values наш queryset не сможет сериализоваться через JSON
        return Response({'posts' : list(lst) })    #возврат фиксированного значения без сериализации

    def post(self, request):
        post_new = UserProfile.objects.create(
            username = request.data['username'],
            discription = request.data['discription'],
            cate_id = request.data['cate_id'],
            user_id = request.data['user_id'],)
        return Response({'post' : model_to_dict(post_new)})     #преобразование данных в словарь. необъродимо импортировать из библиотеки Джанго
```
***
**Класс Serializer**
Сериализация - конвертирование языка пайтон в формат JSON

файл urls
```
urlpatterns = [
    ...
    path('api/v1/userlist/', UserProfileAPIView.as_view()),     #объявление api с его версией /v1, затем суть запроса /userlist
    path('api/v1/userlist/<int:pk>/', UserProfileAPIView.as_view()),
]
```

файл views
```
from urllib import response
from django.shortcuts import render
from django.forms import model_to_dict  #преобразование данных в словарь
from rest_framework import generics
from rest_framework.views import APIView
from rest_framework.response import Response
from django.contrib.auth.models import User

from .models import UserProfile
from .serializers import UserProfileSerializer, UserSerializer

class UserProfileAPIView(APIView):  #используя APIView возможно более тонко настроить обработку запросов от клиента серверу
    #необходимые проверки этих запросов происходят автоматически

    def get(self, request):  #метод для обработки GET запроса
        '''1 cпособ'''
        #lst = UserProfile.objects.all().values()        # без метода .values наш queryset не сможет сериализоваться через JSON
        #return Response({'posts' : list(lst) })    #возврат фиксированного значения без сериализации
        ''' 2 и 3 способ'''
        lst = UserProfile.objects.all()     #формируется список класса с пользователями
        lst_user = User.objects.all()
        #print(UserProfileSerializer(lst, many=True).data)
        return Response({'posts' : UserProfileSerializer(lst, many=True).data, 'lst_user' : UserSerializer(lst_user, many=True).data })    #возврат сериализованного списка. many=True означает, что сериализатор должен обработать список записей, а не один элемент
        # .data переводит результат сериализации в словарь

    def post(self, request):    #создание пользователя
        #print('запрос от клиента ', request.data)
        # I. блок проверки корректности ввода данных
        serializer = UserProfileSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)   #валидация данных от клиента с последующей генерации ошибки при необходимости

        # II. блок добавления пользователя
        ''' 2 способ
        post_new = UserProfile.objects.create(
            username = request.data['username'],
            discription = request.data['discription'],
            cate_id = request.data['cate_id'],
            user_id = request.data['user_id'],)             '''

        '''3 способ'''
        #вместо кода выше достаточно строки:
        serializer.save()   # он вызовет метод create

        # III. блок return
        '''1 способ'''
        #return Response({'post' : model_to_dict(post_new)})     #преобразование данных в словарь. необходимо импортировать из библиотеки Джанго

        '''2 способ'''
        #return Response({'post' : UserProfileSerializer(post_new).data })  # эта строка создавала словарь на основе сериализованных данных

        '''3 способ'''
        return Response({'post' : serializer.data })

    def put(self, request, *args, **kwargs):
        pk = kwargs.get("pk", None)
        if not pk:
            return Response({"error": "Method PUT not allowed" })
        try:
            instance = UserProfile.objects.get(pk=pk)
        except:        # генерация ошибки в случае отсутствия записи по данному pk
            return Response({"error": "Object does not exists"})

        serializer = UserProfileSerializer(data=request.data, instance=instance)
        print(request.data, instance)
        serializer.is_valid(raise_exception=True)
        serializer.save()       # этот метод автоматически вызовет метод update из UserSerializer файла serializers, это возможно благодаря передаче двух атрибутов(instance и data). при вызове метода create - там нужно передать только один атрибут
        return Response({"post" : serializer.data})

    def delete(self, request, *args, **kwargs):
        pk = kwargs.get("pk", None)
        if not pk:
            return Response({"error": "Method DELETE not allowed" })
        try:
            instance = UserProfile.objects.get(pk=pk)
            print(instance)
            instance.delete()
        except:
            return Response({"error": "Object does not exists"})
        return Response({"post" : "delete post " + str(pk)})
```

файл serializers
```
import io
from rest_framework import serializers
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
from .models import UserProfile

    # Сериализация пользователей:
class UserProfileSerializer(serializers.Serializer):    # класс ModelSerializer насследуется от базового класса Serializer
    #работа с моделью БД, имена должны совпадать с объявленными именами класса UserProfile
    username = serializers.CharField(max_length=255)      #представление данных в виде обычной строки. CharField вытсупает в роли валидатора сериализатора
    discription = serializers.CharField()
    created_at = serializers.DateTimeField(read_only=True)  # read_only=True означает, что эти поля будут только читаться, но не будут учавствовать в проверке данных при вводе клиентом
    updated_at = serializers.DateTimeField()
    is_published = serializers.BooleanField(default=True)
    cate_id = serializers.IntegerField()
    user_id = serializers.IntegerField()

    def create(self, validated_data):       #вызывается через метод save во views (serializer.save())
        return UserProfile.objects.create(**validated_data)

    def update(self, instance, validated_data):     # instance - ссылка на объект UserProfile, validated_date - словарь из проверенных данных, которые нужно обработать
        print('update done', instance, validated_data)
        instance.username = validated_data.get('username', instance.username)
        instance.discription = validated_data.get('discription', instance.discription)
        instance.updated_at = validated_data.get('updated_at', instance.updated_at)
        instance.is_published = validated_data.get('is_published', instance.is_published)
        instance.cate_id = validated_data.get('cate_id', instance.cate_id)
        instance.save()
        return instance
```


***
**Класс ModelSerializer и представления ListCreateAPIView**

Класс ModelSerializer заменяет строки кода представленные выше.

файл serializers
```
# Сериализация пользователей:
class UserProfileSerializer(serializers.ModelSerializer):    # класс ModelSerializer насследуется от базового класса Serializer
    class Meta:
        model = UserProfile
        #fields = ("username", "discription", "cate", "user")    # либо
        fields = "__all__"


class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ("username", "first_name", "last_name", "email", "password")    # либо fields = "__all__"
```

Для представления APIView также существует упрощение.
- CreateAPIView - создание данных по POST-запросу.
- ListAPIView - чтение списка данных по GET-запросу.
- RetrieveAPIView - чтение конкретных данных(записи) по GET-запросу
- DestroyAPIView - удаление данных(записи) по DELETE-запросу
- UpdateAPIView - изменение записи по PUT- или PATCH-запросу
- ListCreateAPIView - для чтения (по GET-запросу) и создания списка данных (по POST-запросу)
- RetrieveUpdateAPIView - чтение и изменение отдельной записи (GET- и POST-запрос)
- RetrieveDestroyAPIView - чтение (GET-запрос) и удаление (DELETE-запрос) отдельной записи
- RetrieveUpdateDestroyAPIView - чтение, изменение и добавление отдельной записи (GET-, PUT-, PATCH- и DELETE-запросы)

Документация Джанго:
`http://www.django-rest-framework.org/api-guide/generic-views/`

файл views
```
# пользователи, созданные дополнительно
class UserProfileAPIList(generics.ListCreateAPIView):   #запросы GET и POST
    queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer

class UserProfileAPIUpdate(generics.UpdateAPIView):     # запросы PUT и PATCH
    queryset = UserProfile.objects.all()        # такой запрос является 'ленивым', он выполниться по вызову и выдаст один результат по запросу, а не все
    serializer_class = UserProfileSerializer

class UserProfileAPIDetailView(generics.RetrieveUpdateDestroyAPIView):     # запросы GET-, PUT-, PATCH- и DELETE-запросы
    queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer


# пользователи из админки
class UserAPIList(generics.ListCreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

файл urls
```
from drf_app.views import *

urlpatterns = [
    ...
    path('api/v1/profilelist/', UserProfileAPIList.as_view()),     #объявление api с его версией /v1, затем суть запроса /userlist
    path('api/v1/profilelist/<int:pk>/', UserProfileAPIUpdate.as_view()),
    path('api/v1/profilelist-detail/<int:pk>/', UserProfileAPIDetailView.as_view()),

    path('api/v1/userlist/', UserAPIList.as_view()),
]
```

***
**Viewsets и ModelViewSet**

в файле views
```
from rest_framework import generics, viewsets
from .models import UserProfile
from .serializers import UserProfileSerializer

class UserProfileViewSet(viewsets.ModelViewSet):    # viewsets.ReadOnlyModelViewSet позволяет только читать записи, но не изменять
    queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer
```

В коде по типу
```
path('api/v1/profilelist/', UserProfileViewSet.as_view({ "get" : "list" })),
path('api/v1/profilelist/<int:pk>/', UserProfileViewSet.as_view({ "put" : "update" })),
```
есть повторяющиеся закономерности, при этом функционал GET и POST запросов разделен, поэтому для систематизации данного функционала используют Routers DRF (GET и POST запросы объеденены в одной страницы).

В документации DRF `http://www.django-rest-framework.org/api-guide/routers/` сущестувует два типа роутеров: SimpleRouter и DefaultRouter.

Реализация:

в файле urls
```
from django.urls import path, include
from drf_app.views import *
from rest_framework import routers

router = routers.SimpleRouter() #создали объект класса routers
router.register(r'userprofile', UserProfileAPIDetailView)  # в этом объекте нужно зарегистрировать класс Viewset
print(router.urls)      #коллекция маршрутов

urlpatterns = [
    ...
    path('api/v1/', include(router.urls)), ]    #/127.0.0.1:8000/api/v1/userprofile/
```
при этом если перейти по адресу с указанием конкретной записи (pk) `пример http://127.0.0.1:8000/api/v1/userprofile/5/`, то будет отображена одна запись, в которой добавляются методы PUT и DELETE.

***Справочно*** если перейти в модуль класса ModelViewSet, то он будет представлен несколькими миксинами:
```
class ModelViewSet(mixins.CreateModelMixin,
                   mixins.RetrieveModelMixin,
                   mixins.UpdateModelMixin,
                   mixins.DestroyModelMixin,
                   mixins.ListModelMixin,
                   GenericViewSet):
    """
    A viewset that provides default `create()`, `retrieve()`, `update()`,
    `partial_update()`, `destroy()` and `list()` actions.
    """
    pass
```
заменив насследование `class UserProfileViewSet(viewsets.ModelViewSet)` на `class UserProfileViewSet(mixins.CreateModelMixin, mixins.RetrieveModelMixin, mixins.UpdateModelMixin, mixins.DestroyModelMixin, mixins.ListModelMixin, GenericViewSet)` и удаляя конкретные миксины в списке можно управлять функционалом форматирвоания данных. Например удалив миксин `mixins.DestroyModelMixin` мы удаляем возможность удалять запись со стороны клиента.

***
**Роутеры: SimpleRouter и DefaultRouter.**

router.urls - коллекция/группа маршрутов.

при отсутствии QuerySet в маршрут необходимо добавить basename.
`register(r'userprofile', UserProfileAPIDetailView, basename = usr)`

Для формирования своего уникального маршрута используется специальный декоратор @action.

```
from rest_framework.response import Response
from .models import UserProfile, Category
from .serializers import *
from rest_framework.decorators import action

class UserProfileViewSet(viewsets.ModelViewSet):
    queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer

    #добавление функционала просмотра списка категорий через декоратор @action
    @action(methods=['get'], detail=False)     #detail=False выводит список категорий, а True одну запись
    def category(self, request):        # по имени нашего метода (имя произвольное) через роутер будет формироваться маршрут
        cats = Category.objects.all()
        return Response({ 'cats': [c.title for c in cats]})
```
Новый созданный уникальный маршрут - /api/v1/userprofile/category/.

Для доступа к конкретной записи - /api/v1/userprofile/1/category/, но с измененным методом декоратора @action
```
@action(methods=['get'], detail=True)
def category(self, request, pk=None):
    cats = Category.objects.get(pk=pk)
    return Response({ 'cats': cats.title })
```

Переопределение формата вывода QuerySet. файл views
```
class UserProfileViewSet(viewsets.ModelViewSet):
    # queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer

    def get_queryset(self):
      pk = self.kwargs.get("pk")

      if not pk:
        return UserProfile.objects.all()[:3]

      return UserProfile.objects.filter(pk=pk)
```

Существует возможность создать свой кастомный роутер. Пример есть в документации.
***
**Ограничения доступа (permissions)**

- AllowAny - полный доступ
- IsAuthenticated - только для авторизованных пользователей
- IsAdminUser - только для администратора
- IsAuthenticatedOrReadOnly - только дял атворизованных или всем, но для чтения

Документация Джанго:
`http://www.django-rest-framework.org/api-guide/permissions/`

файл views
```
from rest_framework.permissions import IsAuthenticatedOrReadOnly, IsAdminUser

# пользователи, созданные дополнительно
class UserProfileAPIList(generics.ListCreateAPIView):   #запросы GET и POST
    queryset = UserProfile.objects.all()
    serializer_class = UserProfileSerializer
    permission_classes = (IsAuthenticatedOrReadOnly, )      # проверка на авторизованность пользователя, в случае неавторизованного убирает форму POST

class UserProfileAPIUpdate(generics.RetrieveUpdateAPIView):     # запросы  GET, PUT, PATCH
    queryset = UserProfile.objects.all()        # такой запрос является 'ленивым', он выполниться по вызову и выдаст один результат по запросу, а не все
    serializer_class = UserProfileSerializer
    permission_classes = (IsAdminUser, )

class UserProfileAPIDelete(generics.RetrieveDestroyAPIView):     # запросы GET, DELETE
    queryset = UserProfile.objects.all()        # такой запрос является 'ленивым', он выполниться по вызову и выдаст один результат по запросу, а не все
    serializer_class = UserProfileSerializer
    permission_classes = (IsAdminUser, )    #доступ только для авторизованных

```

файл urls
```
urlpatterns = [
    ...
    path('api/v1/profilelist/', UserProfileAPIList.as_view()),     #объявление api с его версией /v1, затем суть запроса /profilelist
    path('api/v1/profilelist/<int:pk>/', UserProfileAPIUpdate.as_view()),
    path('api/v1/profiledelete/<int:pk>/', UserProfileAPIDelete.as_view()),   ]
```

Для скрытия поля user в форме POST, автоматическое заполнение авторизованным пользователем

файл serializers
```
from rest_framework import serializers

class UserProfileSerializer(serializers.ModelSerializer):    # класс ModelSerializer насследуется от базового класса Serializer
    user = serializers.HiddenField(default=serializers.CurrentUserDefault())    #автоматически заполняет форму POST авторизованным пользователем

    class Meta:
        model = UserProfile
        #fields = ("username", "discription", "cate", "user")    # либо
        fields = "__all__"
```

**Кастомные разрешения / Custom permissions**

2 метода
```
class BasePermission(metaclass=BasePermissionMetaclass)
  def has_permission(self, request, view):    # позволяет настраивать права доступа на уровне всего доступа от клиента
    return true

  def has_object_permission(self, request, view, obj):    # права доступа на уровне отдельного объекта данных
    return True
```
Создадим отдельный файл для доступа permissions.py
```
from rest_framework import permissions

class IsAdminOrReadOnly(permissions.BasePermission):
    def has_permission(self, request, view):    #разрешение на уровне списка записей
        if request.method in permissions.SAFE_METHODS:      #если метод принадлежи коллекции SAFE_METHODS (GET, HEAF, OPTIONS - запросы только на чтение данных, а не изменение) безопасный, то даем доступ
            return True #True - доступ предоставлен
        return bool(request.user and request.user.is_staff)     # а если методы небезопасные, то даем доступ только администратору

#проверка на принадлежность пользвателя как автора записи, например для доступа к редактированию
class IsOwnerOrReadOnly(permissions.BasePermission):    #класс взят из примеров в документации DRF по CustomPermission
    def has_object_permission(self, request, view, obj):    #разрешение на уровне одной записи
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.user == request.user
```
Затем в файле views
```
from .permissions import IsAdminOrReadOnly, IsOwnerOrReadOnly

class UserProfileAPIUpdate(generics.RetrieveUpdateAPIView):     # запросы PUT и PATCH
    queryset = UserProfile.objects.all()        # такой запрос является 'ленивым', он выполниться по вызову и выдаст один результат по запросу, а не все
    serializer_class = UserProfileSerializer
    permission_classes = (IsOwnerOrReadOnly, )      # разрешение редактировать запись только тому пользователю, кто является автором

class UserProfileAPIDelete(generics.RetrieveDestroyAPIView):     # запросы PUT и PATCH
    queryset = UserProfile.objects.all()        # такой запрос является 'ленивым', он выполниться по вызову и выдаст один результат по запросу, а не все
    serializer_class = UserProfileSerializer
    permission_classes = (IsAdminOrReadOnly, )    #доступ только для авторизованных
```
Таким образом неавторизованным пользователям существует доступ для просмотра, но отказано в редактировании.

**Настройки DRF**
Также в файле settings можно указать дополнительную проверку и настройку для DRF
```
REST_FRAMEWORK = {      #настройки DRF
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',     #доступ к данным только авторизованным пользователям, либо .AllowAny - доступ ко всем
    ]
}
```
В случае если в файле views в классах представления не стоит атрибут permission_classes, то будет работать настройка по умолчанию указанная выше.
***
**Авторизация и аутентификация.**

- Session-based authentication - аутентификация на основе сессий и cookies
- Token-based authentication - аутентификация на основе токенов
- JSON Web Token (JWT) authentication - аутентификация на основе JWT-токенов
- Django REST Framework OAuth - авторизация через социальные сети

Документация Джанго:
`http://www.django-rest-framework.org/api-guide/authentication/`


**Session-based authentication - аутентификация на основе сессий и cookies**

в файле urls указываем маршрут для авторизации
```
urlpatterns = [
    ...
    path('api/v1/drf-auth/', include('rest_framework.urls')),   #аутентификация на основе Session-based
]
```
При этом создается дополнительно два маршрута `api/v1/drf-auth/login` и `api/v1/drf-auth/logout`.


**Token-based authentication - аутентификация на основе токенов**

Существует два популярных подхода реализации аутентификации на токенах(сторонние библиотеки):
- обычная аутентификация токенами (библиотека Djoser)
- JWT-токены (библиотека Simple JWT)

Djoser
`https://djoser.readthedocs.io/en/latest/index.html`

`pip install -U djangorestframework_simplejwt`
```
INSTALLED_APPS = [
    ...
    'rest_framework.authtoken',   #авторизация по токенам
    'djoser',
]
```
`python manage.py migrate`

```
from django.urls import ..., re_path

urlpatterns = [
    ...
    path('api/v1/auth/', include('djoser.urls')),
    re_path(r'^auth/', include('djoser.urls.authtoken')),
]
```
**Настройки DRF**

файл settings
```
REST_FRAMEWORK = {      #настройки DRF
    ...
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ),
}
```
По адресу в приложение `api/v1/auth/` станет доступна страница с пользователями, для настройки взаимодействия с ними (добавление, удаления) в документации Djoser необходимо перейти Base Endpoints `https://djoser.readthedocs.io/en/latest/base_endpoints.html`.    *для регистрации методом POST нового пользователя используется адрес `api/v1/auth/users/`

Для авторизации пользователей используется Token Endpoints `https://djoser.readthedocs.io/en/latest/token_endpoints.html`.   *для авторизации методом POST нового пользователя используется адрес `api/v1/auth/token/login/`, после отправик логина и пароля сервер возвращает токен. Затем для работы с ресурсом требуется в отправляемый заголовок Header добавить поле с полученнмы токеном.

при выходе из системы используется адрес `api/v1/auth/token/logout/`, но при  этом нужно указать какой пользователь выходит из системы, т.е. указать  в заголовке токен клиента, полученный при входе.

Для автоматического добавления токена классу представления достаточно указать атрибут authentication_classes
```
from rest_framework.authentication import TokenAuthentication

class UserProfileAPIUpdate(generics.RetrieveUpdateAPIView):     # запросы PUT и PATCH
    queryset = UserProfile.objects.all()        # такой запрос является 'ленивым', он выполниться по вызову и выдаст один результат по запросу, а не все
    serializer_class = UserProfileSerializer
    permission_classes = (IsOwnerOrReadOnly, )      # разрешение редактировать запись только тому пользователю, кто является автором
    authentication_classes = (TokenAuthentication, )  #доступ будет выдаваться не по сессиямм, а по токенам
```










.
