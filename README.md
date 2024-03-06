![example workflow](https://github.com/AndreyLeshchev/kittygram_final/workflows/Kittygram%20workflow/badge.svg)
# Kittygram
Kittygram — социальная платформа для обмена фотографиями своих любимых питомцев.

## Используемые технологии:
* Python 3.9
* Django==3.2.3
* djangorestframework==3.12.4
* Nginx
* gunicorn
## Как запустить проект:

1. Клонировать репозиторий и перейти в него в командной строке:

    ```
    git clone https://github.com/yandex-praktikum/kittygram_backend.git
    ```
    ```
    cd kittygram_backend
    ```

    Cоздать и активировать виртуальное окружение:
    
    ```
    python3 -m venv env
    ```
    
    * Если у вас Linux/macOS
    
        ```
        source env/bin/activate
        ```
    
    * Если у вас windows
    
        ```
        source venv/Scripts/activate
        ```
    
    Установить зависимости из файла requirements.txt:
    ```
    python3 -m pip install --upgrade pip
    ```
    ```
    pip install -r requirements.txt
    ```
    
    Выполнить миграции:
    
    ```
    python3 manage.py migrate
    ```
    
    Для переменных окружения в директории проекта создаем файл .env со следующим содержимым: 
    
    ```
    - POSTGRES_USER — имя пользователя БД (необязательная переменная, значение по умолчанию — postgres);
    - POSTGRES_PASSWORD — пароль пользователя БД (обязательная переменная для создания БД в контейнере);
    - POSTGRES_DB — название базы данных (необязательная переменная, по умолчанию совпадает с POSTGRES_USER).
    - DB_HOST — адрес, по которому Django будет соединяться с базой данных. При работе нескольких контейнеров в сети Docker network вместо адреса указывают имя контейнера, где запущен сервер БД, — в нашем случае это контейнер db.
    - DB_PORT — порт, по которому Django будет обращаться к базе данных. 5432 — это порт по умолчанию для PostgreSQL.
    - DEBUG = включать/выключать отладочный режим приложения в зависимости от текущего окружения (необязательная переменная, значение по умолчанию — False)
    - ALLOWED_HOSTS = список хостов/доменов, для которых может работать текущий сайт (необязательная переменная, значение по умолчанию — 127.0.0.1, localhost).
    - USE_DB - возможность сменить базу данных sqlite/postgresql (необязательная переменная, значение по умолчанию — postgresql).
    ```
2. Подготовка проекта для развертывания на сервере:
   
    Создать Docker образы:
    ```
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway . 
    ```
    Загрузить на DockerHub:
    ```
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```
3. Деплой на сервер:

   Подключитесь к удаленному серверу:
   ```
   ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера
   ```
   Создайте на сервере директорию kittygram через терминал:
   ```
   mkdir kittygram
   ```
   Установка docker compose на сервер:
   ```
   sudo apt update
   sudo apt install curl
   curl -fSL https://get.docker.com -o get-docker.sh
   sudo sh ./get-docker.sh
   sudo apt-get install docker-compose-plugin
   ```
   В директорию kittygram/ скопируйте файлы docker-compose.production.yml и .env:
   ```
   scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
   * ath_to_SSH — путь к файлу с SSH-ключом;
   * SSH_name — имя файла с SSH-ключом (без расширения);
   * username — ваше имя пользователя на сервере;
   * server_ip — IP вашего сервера.
   ```
   Запустите docker compose в режиме демона:
   ```
   sudo docker compose -f docker-compose.production.yml up -d
   ```
   Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:
   ```
   sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
   sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
   sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
   ```
   На сервере в редакторе nano откройте конфиг Nginx и измените настройки location в секции server:
   ```
   sudo nano /etc/nginx/sites-enabled/default
   ```
   ```
   location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:9000;
   }
   ```
   Перезапускаем Nginx:
   ```
   sudo service nginx reload
   ```
   ### Автор - [Андрей Лещев](https://github.com/AndreyLeshchev)
