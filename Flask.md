1. Запустить VS Code от имени администратора, перейти в каталог проекта в PowerShell,
выполнить код ниже, появится папка env, содержащая файлы виртуального окружения
`python -m venv env`

2. Изменить политику, в PowerShell набрать
`Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser`

3. Войти в папку окружения (env), выполнить команду
`.\env\Scripts\activate.ps1`

4. Впереди в PowerShell появится маркер окружения (env), но VS Code может о нем все еще ничего не знать.
Нажать Ctrl+Shift + P, набрать `Python: Select Interpreter`
Указать нужный путь к python.exe в папке окружения env, это отобразится внизу в панели состояния.
Профит! Теперь можно устанавливать модули только для конкретного проекта.

5. Если нужно будет выйти, то в PowerShell выполнить `deactivate`, в выборе интерпетатора вернуться на глобальный.

`pip install Flask`
создаем основной файл сервера flsite.py
```
from flask import Flask

app = Flask(__name__)

@app.route("/")
@app.route("/index")
def index():
    return "index"

if __name__ == "__main__":
    app.run(debug=True)
```
Для импорта шаблонов в основной код программы сервера необходимо импортировать элемент шаблонизатора Jinja.
Рядом с основным файлом сервера создаем директорию templates для хранения шаблонов.
