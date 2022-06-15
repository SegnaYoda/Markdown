#Для тестирования используется библиотека PyTest#
ссылка на GitHub с примерами кода `https://github.com/canyoupleasecreateanaccount/pytestLessonsCode`

#Запуск и настройка теста#
pytest -s -v -k "not development" tests/first_test.py
-v отвечает за подробный иди детальный вывод результата
-s отвечает за вывод всех print в коде тестов
--duration=3  отображение времени прохождения теста, setup время, teardown время, call время. 3 в секундах.
-vv для более подробного duration


*Валидация объектов и работа с jsonschema*
создаем директорию над корнем проекта tests, в ней создаем файл с указанием вначале, либо в конце имени _test.py. В нем будут написаны наши тесты.
```
import requests
from configuration import SERVICE_URL
from src.enums.global_enums import GlobalErrorMessages

def test_getting_cards():
    response = requests.get(url=SERVICE_URL)
    print(response.json())
    recieved_cards = response.json()
    assert response.status_code == 200, GlobalErrorMessages.WRONG_STATUS_CODE.value # код ошибок вынесен в отдельный файл
    # assert len(recieved_cards) == 5, GlobalErrorMessages.WRONG_ELEMENT_COUNT.value
```
Создаем файл конфигурации - configuration.py
```
SERVICE_URL = 'http://127.0.0.1:8000/api/v1/userprofile/'
```
Также создаем директорию для хранения ответов по результатам тестов: src/enums/global_enums.py
```
from enum import Enum

class GlobalErrorMessages(Enum):
    WRONG_STATUS_CODE = "Received status code is not equal expected."
    WRONG_ELEMENT_COUNT = "Number of cards is ot eual to expected."
```
Для запуска первого теста запускаем сервер и вводим `pytest -s -v tests/first_test.py`.
Для валидации Json файлов /по контракту между фронтендом и бекендом/ используем библиотеку `pip install jsonschema`.

В папке src создаем папку schemas, где будут храниться схемы для валидации данных. Пример файл схемы userprofile.py
```
USERPROFILE_SCHEMA = {
   "type": "object",
   "properties": {
       "id": {"type":"number"},
       "username": {"type":"string"},
       "first_name": {"type":"string"},        # в случае если поле должно быть обязательно к примеру строкой POST, то "first_name": {"type":"string", "enum": ["POST"]},
       "last_name": {"type":"string"},        
       "email": {"type":"string"},        

   },
   "required": ["id"]      # обязательные параметры
   }

#  {'id': 3, 'username': 'Jeltorolik', 'first_name': 'ivan', 'last_name': '', 'email': ''}
```
Для вывода ответа и валидации данных мы можем вынести повтоярющийся код в отдельный блок
папка src/baseclasses файл response.py
```
from jsonschema import validate
from src.enums.global_enums import GlobalErrorMessages

class Response:

    def __init__(self, response):
        self.response = response
        self.response_json = response.json()
        self.response_status = response.status_code

    def validate(self, schema):
        # в случае если приходит для проверки не список, то мы валидируем его без цикла
        if isinstance(self.response_json, list):
            for item in self.response_json:   # так как приходит массив, то нужно валидировать по очереди
                validate(item, schema)
        else:
            validate(self.response_json, schema)

    def assert_status_code(self, status_code):
        if isinstance(status_code, list):
            assert self.response_status in status_code, GlobalErrorMessages.WRONG_STATUS_CODE.value # код ошибок вынесен в отдельный файл
        else:
            assert self.response_status == status_code, GlobalErrorMessages.WRONG_STATUS_CODE.value
        return self
```
При этом оснвной код теста имеет вид. first_test.py
```
import requests
from configuration import SERVICE_URL, SERVICE_URL_AUTH
from src.schemas.userprofile import USERPROFILE_SCHEMA
from src.baseclasses.response import Response

def test_getting_userprofile():
    r = requests.get(url=SERVICE_URL)
    response = Response(r)
    response.assert_status_code(200).validate(USERPROFILE_SCHEMA)
```
***
**Урок для начинающих по PyTest #3.2 | Используем pydantic для валидации данных в тестах**

Альтернатива для jsonschema - pydantic_schemas
`pip install pydantic`
Путь src/pydantic_schemas файл userprofile_pydantic.py
Первое отличие от json_schema - простота такой схемы:
```
from pydantic import BaseModel, validator

class UserProfile(BaseModel):
    id: int
    username: str
    first_name: str
    last_name: str
    email: str

#  {'id': 3, 'username': 'Jeltorolik', 'first_name': 'ivan', 'last_name': '', 'email': ''}
```
второе преимущество - свои валидаторы /1 вариант/, userprofile_pydantic.py:
```
# определяет, что количество элементов не больше семи

from jsonschema import ValidationError

@validator("id")
   def check_that_id_is_less_than_seven(cls, v):
       if v > 7:
           raise ValidationError("Id is not less than seven")
       else:
           return v
```
/2 вариант валидатора/
```
# используя Field
from pydantic import BaseModel, validator, Field

class UserProfileSchema(BaseModel):
    id: int = Field(le=2)
    username: str
    first_name: str
    last_name: str
    email: str
```
при этом меняется код Response
```
# from jsonschema import validate
from src.enums.global_enums import GlobalErrorMessages

class Response:

    def __init__(self, response):
        self.response = response
        self.response_json = response.json()
        self.response_status = response.status_code

    def validate(self, schema):
        # в случае если приходит для проверки не список, то мы валидируем его без цикла
        if isinstance(self.response_json, list):
            for item in self.response_json:   # так как приходит массив, то нужно валидировать по очереди
                schema.parse_obj(item)
                # validate(item, schema) # --jsonschema
        else:
            schema.parse_obj(self.response_json)
            # validate(self.response_json, schema) # --jsonschema
        return self

    def assert_status_code(self, status_code):
        if isinstance(status_code, list):
            assert self.response_status in status_code, GlobalErrorMessages.WRONG_STATUS_CODE.value # код ошибок вынесен в отдельный файл
        else:
            assert self.response_status == status_code, GlobalErrorMessages.WRONG_STATUS_CODE.value
        return self
```
***
**Урок для начинающих по PyTest #3.3 | Пишем тесты близкие к боевым условиям и бустим AssertError log**
Example

***
**Урок для начинающих по PyTest #4.1 | Fixtures, conftest. Зачем они и как с ними работать.**
В директории tests создаем файл conftest.py (название файла важно для PyTest)
где и будут указаны наши будущие фикстуры.
```
import pytest

@pytest.fixture
def say_hello():
    print('hello')
```
Затем в коде основного теста указываем нашу фикстуру:
```
...
def test_getting_userprofile(say_hello):    # say_hello указываем фикстуры
    r = requests.get(url=SERVICE_URL)
    response = Response(r)
    response.assert_status_code(200).validate(UserProfileSchema)
```
Фикстура будет запущена раньше кода теста, она может выполнять какой-либо доп функционал внутри себя.
фикстура выполнит свой внутренний код и передаст нашему тесту в качестве аргумента результат выполнения.

Параметры для фикстуры.
Если нажать на `@pytest.fixture`, то откроется библиотека фикстуры с документацией параметров.
scope='function' - выполянется каждый раз при вызове, стоит по умолчанию
scope='session' - код выполняется единожды, а затем кешируется
autouse=True - автоматическое выполнение фикстуры
```
import pytest

@pytest.fixture(scope='session')  # пример указания параметра
def say_hello():
    print('hello')
```
Благодаря фисктуре можно отправлять respone по url.

***
**Урок для начинающих по PyTest #4.2 | Fixtures и conftest интересные фичи которые стоит знать**
Место хранения conftest.py влияет на то, какая директория будет будет попадать под действие фикстуры и будет ли она доступна.

для передачи в тест функции через фикстуру, необходимо создать функцию, например "_calculate", затем создать фикстуру, на выход которой поместим объект функции _calculate.
```
def _calculate(a, b):   # функция, которую мы передадим как объект через фикстуру
  return a + b

@pytest.fixture
def calculate():
    return _calculate
```
По итогу в тест будет помещен объект функции, который можно использовать как обычную функцию.

***
**Конструкция yield**
Если в ходе выполнения фикстуры указать итератор yield, то выполение фикстуры будет приостановлено, будет выполняться тест, после его выполнения заново запуститься выполнение фикстуры. Также yield работает в качестве return.
Это больше поможет для получения временных данных, для подключения к БД и другое.
***
**Урок для начинающих по PyTest #5 | Декораторы для тестов. Parametrize, skip, duration, custom params**

Маркирвока теста, чтобы его пропустить.
```
import pytest

@pytest.mark.skip   # команда чтобы пропустить тест
def test_another():
  assert 1 == 1
```
Можно передать "пропуску" причину `@pytest.mark.skip('[ISSUE-23414] Issue with network connection')`.
***
Для выполнения теста с различными входными параметрами используется `@pytest.mark.parametrize`
т.е. вместо
```
def test_calculation_both_positive(calculate):
    print(calculate(1, 2))


def test_calculation_one_negative(calculate):
    print(calculate(-1, 2))


def test_calculation_both_negative(calculate):
    print(calculate(-1, -2))


def test_calculation_one_char(calculate):
    print(calculate('b', -2))


def test_calculation_both_char(calculate):
    print(calculate('b', 'a'))
```
и фикстуры к нему для проверки
```
def _calculate(a, b):       # функция, которую мы передадим как объект через фикстуру
    if isinstance(a, int) and isinstance(b, int):
        return a + b
    else:
        return None

@pytest.fixture
def calculate():
    print('calculating')
    return _calculate
```
можно использовать это
```
@pytest.mark.parametrize('first_value, second_value, result', [
    (1, 2, 3),
    (-1, -2, -3),
    (-1, 2, 1),
    ('b', -2, None),
    ('b', 'a', None),
    ])
def test_calculator(first_value, second_value, result, calculate):  # calculate здесь это фисктура
    assert calculate(first_value, second_value) == result
```
тесты при этом будут отображаться как не один, а несколько, в зависимости от количества входных данных.
***
#Маркеры#
Для того чтобы отметить какие тесты необходимо запустить можно воспользоваться маркерами.
В глобальной директории создаем файл pytest.ini, где и указываем все необходимые маркеры:
```
[pytest]
markers=
    development: marker for runnig test only on dev env
    production: marker for run test on prod
```
Затем маркируем сами тесты. Причем маркировать тест можно несколькими маркерами.
```
!@pytest.mark.development!        # маркировка теста - development
@pytest.mark.parametrize('first_value, second_value, result', [
    (1, 2, 3),
    (-1, -2, -3),
    (-1, 2, 1),
    ('b', -2, None),
    ('b', 'a', None),
    ])
def test_calculator(first_value, second_value, result, calculate):
    assert calculate(first_value, second_value) == result
```
Для запуска тестов только определенного маркера в командной строке прописываем дополнительно -k development, где development это название нашего маркера. `pytest -s -v -k development tests/first_test.py`
Для того чтобы запустить все остальные тесты кроме конретно помеченного определенным маркером - `pytest -s -v -k "not development" tests/first_test.py`
***
**Урок для начинающих по PyTest #6 | Создаём красивый allure report для результатов тестов**
linux: `sudo apt-get install allure`
windows: `scoop install allure`
Но сперва необходимо установить Scoop.
Устанавливанм в проект allure `pip install allure-pytest`.

Теперь команда для запуска `pytest -s -v -k development tests/first_test.py --alluredir=allureress`
После этой команды будут созданы директории allureress, куда будут помещены подробные отчеты о тестах в формате json.
! не забыть поставить в gitignore !
Далее чтобы сгенирировать html report страницу с подробныи отчетом - `allure serve allureress`, где allureress наша директория с файлами json.
! Важно установить Java и перезапустить интерпритатор !

***
**Урок для начинающих по PyTest #7.1 | простенький и элегантный билдер для генерации данных**
Генераторы позволят генерировать новые входные данные для тестов.

В директории src создаем папку generators, в ней будет находиться файл с генератором входных данных, например userprofile_generator.py
У нас есть пример данных, на основе которых нужно будет генерировать новые:
```
# Пример данных с котороми будет работать билдер Userprofile
user = {
    "account_status": "ACTIVE",
    "balance": 10,
    "localize": {
        "en": {"nickname": "SolweMe"},
        "ru": {"nickname": "СолвеМи"},
    },
    "avatar": "https://google.com/"
    }
```
В файле нашего билдера
```
from src.enums.global_enums import Statuses
from src.generators.userprofile_localize import UserprofileLocalisation

class Userprofile:

    def __init__(self):
        self.result = {}
        self.reset()

    def set_status(self, status=Statuses.ACTIVE.value):
        self.result['account_status'] = status
        return self

    def set_balance(self, balance=0):
        self.result['balance'] = balance
        return self

    def set_avatar(self, avatar="https://google.com/"):
        self.result['avatar'] = avatar

    def reset(self):
        self.set_status()
        self.set_avatar()
        self.set_balance()
        self.result["localize"] = {
                "en": UserprofileLocalisation('en_US').build(),
                "ru": UserprofileLocalisation('ru_RU').build()
        }
        return self

    def update_inner_generator(self, key, generator):
        self.result[key] = {"en": generator.build()}
        return self

    def build(self):
        return self.result
```
!Справочно!
Для from src.enums.global_enums import Statuses создадим статусы:
```
from enum import Enum

class Statuses(Enum):
    INACTIVE = "INACTIVE"
    ACTIVE = "ACTIVE"
```
Для генерации пункта "localize" нужен отдельный билдер, так как здесь существует вложенность.
Затем установим так называемый фейкер `pip install faker`, который умеет генерировать фейковые имена, адреса, ссылки.

Создаем его в этой же директории /src/userprofile_localize.py
```
from faker import Faker

class UserprofileLocalisation:

    def __init__(self, lang):
        self.fake = Faker(lang)
        self.result = {
            "nickname": self.fake.first_name()
        }

    def build(self):
        return self.result
```
Теперь мы можем создать объект класса, который будет генерировать нам новых пользователей с необходимыми нам параметрами.
```
c = Userprofile().set_balance(20).set_status('asaas').build()
print(c)
```
*Генератор написан!*
Для работы с основным тестом необходимо создать фикстуру в основном файле conftest.py директории /tests
```
from src.generators.userprofile_generator import Userprofile

@pytest.fixture
def get_userprofile_generator():
    return Userprofile()
```
Затем в основных тестах tests/something/test_something.py
```
# Напоминание: фикстуры не нужно импортить, они определяются автоматически
import pytest


@pytest.mark.parametrize("status", [
    "ACTIVE",
    "BANNED",
    "DELETED",
    "INACTIVE",
    ])
def test_something_status(status, get_userprofile_generator):       # фикстура, возвращающая генератор
    # print(get_userprofile_generator.build())     
    print(get_userprofile_generator.set_status(status).build())     # принт сгенирируемого нового пользователя


@pytest.mark.parametrize("balance_value", [
    "100",
    "0",
    "-15",
    "asdf",
    ])
def test_something_balance_value(balance_value, get_userprofile_generator):       # фикстура, возвращающая генератор
    print(get_userprofile_generator.set_balance(balance_value).build())     # принт сгенирируемого нового пользователя


@pytest.mark.parametrize("dekete_key", [
    "account_status",
    "balance",
    "localize",
    "avatar",
    ])
def test_something_dekete_key(dekete_key, get_userprofile_generator):       # фикстура, возвращающая генератор
    object_to_send = get_userprofile_generator.build()
    del object_to_send[dekete_key]
    print(object_to_send)


def test_something(get_userprofile_generator):
    object_to_send = get_userprofile_generator.update_inner_generator(
        'localize', UserprofileLocalisation('fr_FR')      # смена локализации на французский
        ).build()
    print(object_to_send)
```
И запускаем наш тест `pytest -s -v tests/something/test_something.py`
***
**Урок для начинающих по PyTest #7.2 | Работаем с билдерами и данными на любом уровне вложенности**
Код пропущен, при необходимости в видео.
*https://www.youtube.com/watch?v=RBa06b-m8C4&list=PLB2iiSfKWtvykq9s0plSVI_Du60i0iphU&index=15*
***
**Урок для начинающих по PyTest #8 | Детальный разбор pydantic и способы работы с ним в автотестах**

Pydantic необходим для валидации/проверки данных.
Он может заменить json_schema
Пример данных для проверки
```
computer = {
    "id": 21,
    "status": "ACTIVE",
    "activated_at": "2013-06-01",
    "expiration_at": "2040-06-01",
    "host_v4": "91.192.222.17",
    "host_v6": "2001:0db8:85a3:0000:0000:8a2e:0370:7334",
    "detailed_info": {
        "physical": {
            "color": 'green',
            "photo": 'https://images.unsplash.com/photo-1587831990711-23ca6441447b?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxzZWFyY2h8MXx8ZGVza3RvcCUyMGNvbXB1dGVyfGVufDB8fDB8fA%3D%3D&w=1000&q=80',
            "uuid": "73860f46-5606-4912-95d3-4abaa6e1fd2c"
        },
        "owners": [{
            "name": "Stephan Nollan",
            "card_number": "4000000000000002",
            "email": "shtephan.nollan@gmail.com",
        } ] } }
```
В директории src/pydantic_schemas/computer_pydantic.py
```
from pydantic import UUID4, BaseModel, HttpUrl, PaymentCardNumber, EmailStr
from pydantic.types import PastDate, FutureDate, List # для проверки дат
from pydantic.networks import IPv4Address, IPv6Address
from pydantic.color import Color
from src.enums.global_enums import Statuses


class Physical(BaseModel):
    color: Color
    photo: HttpUrl
    uuid: UUID4

class Owners(BaseModel):
    name: str
    card_number: PaymentCardNumber
    email: EmailStr

class DetailedInfo(BaseModel):
    physical: Physical
    owners: List[Owners]

class Computer(BaseModel):
    status: Statuses
    activated_at: PastDate
    expiration_at: FutureDate
    host_v4: IPv4Address
    host_v6: IPv6Address
    detailed_info: DetailedInfo

computer = {
    "id": 21,
    "status": "ACTIVE",
    "activated_at": "2013-06-01",
    "expiration_at": "2040-06-01",
    "host_v4": "91.192.222.17",
    "host_v6": "2001:0db8:85a3:0000:0000:8a2e:0370:7334",
    "detailed_info": {
        "physical": {
            "color": 'green',
            "photo": 'https://images.unsplash.com/photo-1587831990711-23ca6441447b?ixlib=rb-1.2.1&ixid=MnwxMjA3fDB8MHxzZWFyY2h8MXx8ZGVza3RvcCUyMGNvbXB1dGVyfGVufDB8fDB8fA%3D%3D&w=1000&q=80',
            "uuid": "73860f46-5606-4912-95d3-4abaa6e1fd2c"
        },
        "owners": [{
            "name": "Stephan Nollan",
            "card_number": "4000000000000002",
            "email": "shtephan.nollan@gmail.com",
        }
    ]
    }
}

comp = Computer.parse_obj(computer)
print(comp)
```
Файл src/baseclass/response.py будет выглядеть следующим образом
```
# from jsonschema import validate
from src.enums.global_enums import GlobalErrorMessages


class Response:

    def __init__(self, response):
        self.response = response
        self.response_url = response.url
        self.response_json = response.json()
        self.response_status = response.status_code
        self.parsed_object = None

    def validate(self, schema):
        # в случае если приходит для проверки не список, то мы валидируем его без цикла
        if isinstance(self.response_json, list):
            for item in self.response_json:   # так как приходит массив, то нужно валидировать по очереди
                parsed_object = schema.parse_obj(item)
                # validate(item, schema) # --jsonschema
                self.parsed_object = parsed_object
        else:
            schema.parse_obj(self.response_json)
            # validate(self.response_json, schema) # --jsonschema
        return self

    def assert_status_code(self, status_code):
        if isinstance(status_code, list):
            assert self.response_status in status_code, GlobalErrorMessages.WRONG_STATUS_CODE.value # код ошибок вынесен в отдельный файл
        else:
            assert self.response_status == status_code, GlobalErrorMessages.WRONG_STATUS_CODE.value
        return self

    def get_parsed_object(self):
        return self.parsed_object

    def __str__(self):
        return \
            f"\nStatus code: {self.response_status} \n" \
            f"Requested url: {self.response_url} \n" \
            f"response body: {self.response_json}"
```
Основной тест для такой валидации
```
import pytest
from src.pydantic_schemas.computer_pydantic import Computer, computer

def test_pydantic_object():
    comp = Computer.parse_obj(computer)
    print(comp)        
    print(comp.detailed_info.physical.photo)        # можно получить любой пункт информации
    # после получения конкретных данных их можно прогнать через assert для проверки  
```
*Pydantic умеет представлять/сгенирировать свою схему в виде json_schema, для этого:*
```
comp = Computer.parse_obj(computer)
print(comp.schema_json())
```

*Упроещение*
Для упрощения кода параметров Parametrize в тестах, вынесем enum части в отдельные классы.
Например есть код теста
```
@pytest.mark.parametrize("status", [
    "ACTIVE",
    "BANNED",
    "DELETED",
    "INACTIVE",
    ])
def test_something_status(status, get_userprofile_generator):       # фикстура, возвращающая генератор
    # print(get_userprofile_generator.build())     
    print(get_userprofile_generator.set_status(status).build())
```
создаем новый файл в enums/pyenum.py
```
from enum import Enum

class PyEnum(Enum):
    @classmethod
    def list(cls):
        return list(map(lambda c: c.value, cls))
```
в файле global_enums.py
```
from src.enums.global_enums import PyEnum

class Statuses(PyEnum): # меняем изначальный Enum на наш кастомный PyEnum выше
    INACTIVE = "INACTIVE"
    ACTIVE = "ACTIVE"
    BANNED = "BANNED"
    DELETED = "DELETED"
```
Теперь в тесте указываем
```
import pytest
from enums.global_enums import Statuses

@pytest.mark.parametrize("status", Statuses.list())    # теперь наш класс Statuses проходя через PyEnum будет выдавать список статусов
def test_something_status(status, get_userprofile_generator):
    # print(get_userprofile_generator.build())     
    print(get_userprofile_generator.set_status(status).build())
```
***

**Урок для начинающих по PyTest #9.1 | Подключаем SQLAlchemy к проекту и делаем простые запросы в базу**
Устанавливаем `pip install sqlalchemy`
в файле конфигурации configuration.py указываем путь к БД: `CONNECTION_ROW = "postgresql://postgres:postgres@db:5432/postgres"`
Создаем в корневом каталоге файл db.py
```
