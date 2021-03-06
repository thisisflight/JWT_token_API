## API отправки сообщений с использованием JWT-токена авторизации

В данном API полностью реализованы: 
- функционал регистрации пользователя с сохранением в БД; 
- логин пользователя посредством JWT-токена (с установкой токена в cookies либо передачей в заголовках запроса);
- логаут пользователя посредством удаления токена из cookies;
- отправка сообщений авторизованным пользователем (проверка наличия токена в заголовках запроса либо в куки);
- возможность вернуть последние N сообщений, отправив специальное сообщение на сервер.

Тесты покрывают все эндпоинты и весь реализованный функционал.

Для запуска проекта достаточно создать файл .env в папке настроек Django-проекта (папка jwt_auth, внутри которой файл ```settings.py```) и скопировать в него содержимое файла .env.dist в основной директории.

Запуск приложения из корневой директории осуществляется следующей командой:

```
docker-compose up -d
```
(убедитесь, что у вас установлены docker и docker-compose)

При первом запуске приложения создайте все необходимые миграции Django-проекта:

```
docker exec -it authapp sh -c "python manage.py migrate"
```

Порт контейнера 8000 проброшен на порт 8000 хостовой машины, поэтому можно делать запросы к эндпоинтам любым удобным способом с хоста.

### Регистрация пользователя

Реализуется при помощи POST-запроса по адресу:

```http://localhost:8000/signup/```

Пример запроса, который создаст пользователя:

```
curl http://localhost:8000/signup/ -X POST \
-H 'Content-type: application/json' \
-d '{"name": "some_name", "password": "some_password"}'
```

Ответ сервера:

```{"name":"some_name"}```

### Логин пользователя

Когда пользователь зарегистрирован, он может быть залогинен в системе при помощи JWT-токена.

Реализуется при помощи POST-запроса по адресу:

```
http://localhost:8000/login/
```

Пример запроса, при помощи которого осуществляется вход в систему (возьмем данные зарегистрированного ранее пользователя):
```
curl http://localhost:8000/login/ -X POST \
-H 'Content-type: application/json' \
-d '{"name": "some_name", "password": "some_password"}'
```

Сервер возвращает JSON-ответ с JWT-токеном:
```
{"token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoic29tZV9uYW1lIiwiZXhwIjoxNjUyODE2MDA5LCJpYXQiOjE2NTI4MDUyMDl9.8VQpB8_N4bdYyL2T_ThroHyKCZ2LX5AHimeH8Cziig8"}
```

### Логаут пользователя

Отправка POST-запроса на эндпоинт ```http://localhost:8000/logout/``` удалит token из cookies.

### Отправка сообщения залогиненным пользователем

Для отправки сообщения на сервер необходимо передать токен авторизации в заголовках запроса либо в cookies.

Пример запроса с токеном в заголовках, создающий сообщение на сервере:

```
curl http://localhost:8000/message/ -X POST \
-H 'Content-type: application/json' \ 
-H 'token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoic29tZV9uYW1lIiwiZXhwIjoxNjUyODE2MDA5LCJpYXQiOjE2NTI4MDUyMDl9.8VQpB8_N4bdYyL2T_ThroHyKCZ2LX5AHimeH8Cziig8' \ 
-d '{"name": "some_name", "text": "some_text"}'
```

Сервер вернет ответ ```{"status":"Сообщение создано"}```.

Также есть возможность получить N последних сообщений, отправленных залогиненным пользователем, при помощи специального сообщения с префиксом last и числом интересующих сообщений:

```last 5```

Пример запроса:

```
curl http://localhost:8000/message/ -X POST \
-H 'Content-type: application/json' \ 
-H 'token: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoic29tZV9uYW1lIiwiZXhwIjoxNjUyODE2MDA5LCJpYXQiOjE2NTI4MDUyMDl9.8VQpB8_N4bdYyL2T_ThroHyKCZ2LX5AHimeH8Cziig8' \ 
-d '{"name": "some_name", "text": "last 5"}'
```

В ответ сервер вернет структуру из 5 последних сообщений:

```
{
    "5": "some_text",
    "4": "some_text",
    "3": "some_text",
    "2": "some_text",
    "1": "some_text"
}
```