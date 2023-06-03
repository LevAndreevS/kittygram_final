# Проект Kittygram: о наших пушистых друзьях.
Мой проект kittygram располжен на сайте [https://kittygrambylev.ddns.net/](https://kittygrambylev.ddns.net/)
- Вы можете зарегистрироваться и добавлять Ваших пушистых рузей
- Обязательно необходимо указать имя и год рождения питомца.
- Можно добавлять фотографии и указывать умения котэ.
# Основные технологии:

- Проект написан на Python 3.9;
- Бэкенд-приложение на Django 3.2.3;
- Django REST Framework;
- Библиотека Simple JWT;
- Gunicorn 20.1.0
- Фронтенд-приложение на React
- npm 9.5.1
- База данных - PostgreSQL 13.10
- Docker version 24.0.2, build cb74dfc

# Структура проекта:
## 4 контейнера и 2 volume директории.
## - контейнер с backend разделом (серверная часть проекта)
## - контейнер с fronend разделом
## - база даннах postgressql
## - контейнер gateway(nginx)
это внутренний сервер nginx, он будет пробрасывать порт для подключения с nginx, расположенным на сервере. 
## - volume директория - подключаемая статика
backend static и frontend static
## - volume директория - для подключения media picture

# Проект можно запустить на локальной машине
```PYTHON
git clone git@github.com:LevAndreevS/kittygram_final.git
cd kittygram_final/
```
**Cоздать и активировать виртуальное окружение:**

```PYTHON
python3 -m venv venv
source venv/bin/activate
```
**Установить зависимости из файла requirements.txt:**

```PYTHON
pip3 install -r requirements.txt	
```
- в корне проекта создать .env файл и добавить туда. В качестве примера используйте .env.example, файл так же расположен в корневой папке.
```
# Файл .env
POSTGRES_USER=имя пользователя
POSTGRES_PASSWORD=password
POSTGRES_DB=пароль пользователя
DB_HOST=имя базы данных
DB_PORT=порт
```
- в settings.py необходимо добавить:
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('POSTGRES_DB', 'django'),
        'USER': os.getenv('POSTGRES_USER', 'django'),
        'PASSWORD': os.getenv('POSTGRES_PASSWORD', ''),
        'HOST': os.getenv('DB_HOST', ''),
        'PORT': os.getenv('DB_PORT', 5432)
    }
}
```

```Python
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'collected_static'

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```
- С помощью файла doсker-compouse.yml собрать образы и на их основе запустить контейнеры:
```Bash
sudo docker compose up --build
```
- Собрать статику Django в новом окне терминала
```Bash
sudo docker compose exec backend python manage.py collectstatic
```
- Теперь из этой директории копируем статику в /backend_static/static/ эта статика попадёт на volume static в папку /static/
```Bash
sudo docker compose exec backend cp -r /app/collected_static/. /backend_static/static/
```
 - При обращении к адресу http://localhost:9000/ должна отобразиться главная страница.
 - Порт указан в doсker-compouse.yml.
 - После проверки работоспособности сайта останвяливаем контейнеры Ctrl + C или вводим в дополнительном терминале: 
```Bash
sudo docker compose stop
```
- С помощью этой команды можно названия образов:
```Bash
sudos docker image ls -a
```
- Прежде чем загружать образы на DockerHub, нужно аутентифицировать там докер-демона. Выполните команду аутентификации:
```Bash
sudo docker login
# А можно сразу указать имя пользователя:
sudo docker login -u username
```
- Отправляем образы на https://hub.docker.com/
```Python
sudo docker push username/image_name_backend:latest
sudo docker push username/image_name_frontend:latest
sudo docker push username/image_name_gateway:latest
```
# Как запустить проект сервере необходимо:
- Выполнить шаг: **"Проект можно запустить на локальной машине"**, он выше, после структуры проекта.
- Проверяем данные и изменяем на свои в файле doсker-compouse.production.yml. Названия докер образов должны совпадать с названием образов в аккаунте на Dockerhub. 
```Bash
version: '3'

volumes:
  pg_data_production:
  media:
  static_volume:

services:
  db:
    image: postgres:13
    env_file: .env
    volumes:
      - pg_data_production:/var/lib/postgresql/data
  backend:
    image: username/image_name:latest
    env_file: .env
    volumes:
      - static_volume:/backend_static
      - media:/app/media/
  frontend:
    image: username/image_name:latest
    env_file: .env
    command: cp -r /app/build/. /frontend_static/
    volumes:
      - static_volume:/frontend_static
  gateway:
    image: username/image_name:latest
    env_file: .env
    volumes:
      - static_volume:/staticfiles/
      - media:/app/media/
    ports:
      - 9000:80
```
- В дополнительном окне терминала подключитесь к серверу:
```Bash
# Ключ -i позволяет указать путь до SSH-ключа, отличный от пути по умолчанию.
ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом_без_расширения login@ip

# Например:
# ssh -i D:/Dev/vm_access/yc-yakovlevstepan yc-user@153.0.2.10
```
- Создайте на сервере директорию kittygram/
- Локально со своей машины отправьте на сервер doсker-compouse.production.yml и .env
```Bash
scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittyrgam/docker-compose.production.yml .env
```
- На сервере установите Docker - скрипт для установки докера с официального сайта
```Bash
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```
- Дополнительно к Docker установите утилиту Docker Compose:
```Bash
sudo apt-get install docker-compose-plugin
sudo systemctl status docker
```
Выполните команду аутентификации:
```Bash
sudo docker login
# А можно сразу указать имя пользователя:
sudo docker login -u username
```
- Скачайте image образы проекта с dokerhub
```Bash
sudo docker compose -f docker-compose.production.yml pull
```
- Запустите их демоном
```Bash
sudo docker compose -f docker-compose.production.yml up -d
```
- Создайте миграции, соберите статикуу и скопируте её в volume директорию
```Bash
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate

sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic

sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```

# Об Авторе:
- студент 53 когорты python - developer Яндекс - Практикума.
[**Лев Андреев**](https://github.com/LevAndreevS)