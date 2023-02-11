# Запуск docker-compose проекта API_YAMDB
### Описание
Проект YaMDb собирает отзывы пользователей на произведения. Сами произведения в YaMDb не хранятся, здесь нельзя посмотреть фильм или послушать музыку.

### Технологии

* Python 3.7

* Django 2.2

* DRF 

* JWT + Djoser

* Postgresql

* Docker

* NGINX

# Запуск проекта
## Dev-режим:
- Клонируйте репозиторий и перейдите в него в командной строке.
```bash
git clone https://github.com/WispHes/infra_sp2.git
```
- Установите виртуальное окружение c версии Python 3.7 :
```bash
py -3.7 -m venv venv
```
- Активируйте виртуальное окружение :

 для windows-систем:
```bash
source venv/Scripts/activate
```
для *nix-систем:     
```bash
source venv/bin/activate
```
- Перейдите в директорию yatube_api и установите зависимости из файла requirements.txt:
```bash
cd api_yamdb

pip install -r requirements.txt
```
- Выполните миграции:
```bash
python manage.py makemigrations

python manage.py migrate
```
- После требуется создать суперпользователя:
```bash
python manage.py createsuperuser
```
- Чтобы запустить проект используйте команду:
```bash
python manage.py runserver
```
## Docker-compose:
### Предварительная настройка

- Шаблон наполнения .env расположенный по пути infra/.env
```bash
DB_ENGINE=django.db.backends.postgresql # указываем, что работаем с postgresql
DB_NAME=postgres # имя базы данных
POSTGRES_USER=postgres # логин для подключения к базе данных
POSTGRES_PASSWORD=postgres # пароль для подключения к БД (установите свой)
DB_HOST=db # название сервиса (контейнера)
DB_PORT=5432 # порт для подключения к БД 
```
- Заполнение докерфайла расположенный по пути api_yamdb/Dockerfile:
```bash
FROM python:3.7-slim

# Создать директорию вашего приложения.
RUN mkdir /app

# Скопировать с локального компьютера файл зависимостей
# в директорию /app.
COPY requirements.txt /app

# Выполнить установку зависимостей внутри контейнера.
RUN pip3 install -r /app/requirements.txt --no-cache-dir

# Скопировать содержимое директории /api_yamdb c локального компьютера
# в директорию /app.
COPY api_yamdb/ /app

# Сделать директорию /app рабочей директорией. 
WORKDIR /app

# Выполнить запуск сервера разработки при старте контейнера.
CMD ["gunicorn", "api_yamdb.wsgi:application", "--bind", "0:8000" ] 
```
- Заполнение docker-compose расположенный по пути infra/docker-compose:
```bash
# версия docker-compose
version: '3.8'

# имена и описания контейнеров, которые должны быть развёрнуты
services:
  # описание контейнера db
  db:
    # образ, из которого должен быть запущен контейнер
    image: postgres:13.0-alpine
    # volume и связанная с ним директория в контейнере
    volumes:
      - /var/lib/postgresql/data/
    # адрес файла, где хранятся переменные окружения
    env_file:
      - ./.env
  web:
    build:
      context: ../
      dockerfile: api_yamdb/Dockerfile
    restart: always
    volumes:
      # Контейнер web будет работать с данными, хранящиеся в томе static_value, 
      # через свою директорию /app/static/
      - static_value:/app/static/
      # Данные, хранящиеся в томе media_value, будут доступны в контейнере web 
      # через директорию /app/media/
      - media_value:/app/media/
    # «зависит от», 
    depends_on:
      - db
    env_file:
      - ./.env

  # Новый контейнер
  nginx:
    # образ, из которого должен быть запущен контейнер
    image: nginx:1.21.3-alpine

    # запросы с внешнего порта 80 перенаправляем на внутренний порт 80
    ports:
      - "80:80"

    volumes:
      # При сборке скопировать созданный конфиг nginx из исходной директории 
      # в контейнер и сохранить его в директорию /etc/nginx/conf.d/
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf

      # Контейнер nginx будет работать с данными, хранящиеся в томе static_value, 
      # через свою директорию /var/html/static/
      - static_value:/var/html/static/

      # Данные, хранящиеся в томе media_value, будут доступны в контейнере nginx
      # через директорию /var/html/media/
      - media_value:/var/html/media/

    depends_on:
      # Контейнер nginx должен быть запущен после контейнера web
      - web

volumes:
  # Новые тома 
  static_value:
  media_value:

```
### Запуск

- Переходим в папку с docker-compose:
```bash
cd infa
```
- Запустите контейнер:
```bash
docker-compose up -d --build 
```
- Выполните миграции:
```bash
docker-compose exec web python manage.py makemigrations

docker-compose exec web python manage.py migrate
```
- Создайте суперпользователя:
```bash
docker-compose exec web python manage.py createsuperuser
```
- Соберите статику:
```bash
docker-compose exec web python manage.py collectstatic --no-input
```

# Документация
Документация будет доступна после запуска проекта по адресу /redoc/.

## Примеры некоторых запросов API
Регистрация пользователя:
```
POST /api/v1/auth/signup/
```
Получение данных своей учетной записи:
```
GET /api/v1/users/me/
```
Получение списка всех отзывов:
```
GET /api/v1/titles/{title_id}/reviews/
```
Добавление комментария к отзыву:
```
POST /api/v1/titles/{title_id}/reviews/{review_id}/comments/
```
Добавление новой категории:
```
POST /api/v1/categories/
```
Удаление жанра:
```
DELETE /api/v1/genres/{slug}
```
Частичное обновление информации о произведении:
```
PATCH /api/v1/titles/{titles_id}
```


Полный список запросов API находятся в документации.

### Авторы
[Остапчук Дмитрий](https://github.com/WispHes) 
