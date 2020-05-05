# AuthVK

## Авторизация в VK

AuthVK - модуль для авторизации и получения OAuth-токена для вашего приложения.
### Установка
---

Для того чтобы установить AuthVk, просто выполните:

`pip install git+https://github.com/SemenovAV/AuthVK#egg=authvk`

Для работы модуля требуется дополнительно установить:

- [requests](https://github.com/kennethreitz/requests/) 
- [setuptools](https://github.com/pypa/setuptools/)


### Использование
---

1. Импортируйте модуль в ваш проект.
2. Создайте екземпляр класса с требуемыми параметрами. 
3. Вызовите метод `get_auth()` для получения `access_token` , `user_id` и сессии.


Обязательные параметры:

  - `login` - логин для аунтентификации в VK.
  
  - `password` - пароль для аунтефикации в VK.
  
Необязателбьные параметры:

  - `config` - конфигурация:
  
      - `client_id` - id приложения для которого получается токен
    
      - `scope` - Разрешения которые требуются приложению
    
      - `redcirect_url` - Редирект урл указанный в настройках приложения 
    
      - `v` - версия апи
    
    пример:
```json
{
    "client_id": "7395093",
    "scope": "friends,groups,offline",
    "redirect_url": "https://oauth.vk.com/blank.html",
    "v": "5.103"
}

```

                
   - `user_agent` - строка user-agent.
   
   - `auth_url` - урл сервера OAuth авторизации VK.

   - `logger` - python логгер.
   
   - `auto` - включает основной рабочий режим, при котором обрабатывается функция 
   переданная параметром form_data_handler.
   
   - `form_data_handler` - функция обработчик данных приходящих с сервера. 
   
   
В функцию `form_data_handler` передается три параметра:

1. Контекст - экземпляр класса.
2. Словарь параметров полученный из парсера форм - то что надо обработать.
3. Поле из словаря параметров, которое обрабатывается.
Функция должна возвращать требуемое значение.



Пример, обработка аунтификации:

Если в словаре params есть поля `email` и `pass` - требуется аунтефикация, 
если обрабатывается поле `pass` (третьий параметр key == `pass`) - отдаем поле `password` экземпляра класса,
если нет отдаем поле `email` экземпляра класса.
```python
def handler(ctx, params, key):
    if 'email' in params and 'pass' in params:
        return ctx.password if key == 'pass' else ctx.email
```

По умолчанию в `form_data_handler` передается функция обработчик -  `form_data_handlers.hendler`

Функция по умолчанию обрабатывает:

- аунтификация по логину и паролю
- аунтификация, второй фактор - код (выкидывает запрос кода в консоль)
- Капча (выкидывает ссылку на капчу и запрос кода в консоль )
- авторизацию (автамотическое подтверждение всех переданных в `scope` разрешений)




При `auto=False` параметр `form_data_handler` не учитывается. Класс находится в тестовом 
режиме при котором удобно делать отладку.Первым следует вызывать метод `get_login_form`, 
далее по очереди для каждой формы:

1. `parse_form` - для обработки полей формы требующих внимания. Метод принимает один параметр - функцию обработчик.

2. `submit_form` - для отправки обработанной формы.


Пример работы:
```python
from AuthVK.core import Auth
from urllib.parse import urlencode

my_auth = Auth(login="12345", password="vasia")

data = my_auth.get_auth()

if data and 'session' in data:
    params = {
        'access_token': data.get('access_token'),
        'v': data.get('v')
    }
    session = data['session']
    response = session.get(f'https://api.vk.com/method/users.get?{urlencode(params)}')
...
```



