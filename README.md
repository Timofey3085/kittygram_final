## ОПИСАНИЕ ПРОЕКТА 

# Kittygram - сервис для любителей котиков.

Workflow Status

Что умеет проект:

Добавлять, просматривать, редактировать и удалять котиков.
Добавлять новые и присваивать уже существующие достижения.
Просматривать чужих котов и их достижения.

# Установка
Клонируйте репозиторий на свой компьютер:

git clone git@github.com:Timofey3085/kittygram_final.git
cd kittygram
Создайте файл .env и заполните его своими данными. Перечень данных указан в корневой директории проекта в файле .env.example.

# Создание Docker-образов
Замените username на ваш логин на DockerHub:

cd frontend
docker build -t username/kittygram_frontend .
cd ../backend
docker build -t username/kittygram_backend .
cd ../nginx
docker build -t username/kittygram_gateway . 
Загрузите образы на DockerHub:

docker push username/kittygram_frontend
docker push username/kittygram_backend
docker push username/kittygram_gateway

# Деплой на сервере
Подключитесь к удаленному серверу

ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 
Создайте на сервере директорию kittygram через терминал

mkdir kittygram

# Установка docker compose на сервер:

sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt-get install docker-compose-plugin
В директорию kittygram/ скопируйте файлы docker-compose.production.yml и .env:

scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
* ath_to_SSH — путь к файлу с SSH-ключом;
* SSH_name — имя файла с SSH-ключом (без расширения);
* username — ваше имя пользователя на сервере;
* server_ip — IP вашего сервера.
Запустите docker compose в режиме демона:

sudo docker compose -f docker-compose.production.yml up -d
Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/

# На сервере в редакторе nano откройте конфиг Nginx:

sudo nano /etc/nginx/sites-enabled/default
Измените настройки location в секции server:

location / {
    proxy_set_header Host $http_host;
    proxy_pass http://127.0.0.1:9000;
}

Проверьте работоспособность конфига Nginx:

sudo nginx -t
Если ответ в терминале такой, значит, ошибок нет:

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
Перезапускаем Nginx

sudo service nginx reload
Настройка CI/CD
Файл workflow уже написан. Он находится в директории

kittygram/.github/workflows/main.yml
Для адаптации его на своем сервере добавьте секреты в GitHub Actions:

DOCKER_USERNAME                # имя пользователя в DockerHub
DOCKER_PASSWORD                # пароль пользователя в DockerHub
HOST                           # ip_address сервера
USER                           # имя пользователя
SSH_KEY                        # приватный ssh-ключ (cat ~/.ssh/id_rsa)
SSH_PASSPHRASE                 # кодовая фраза (пароль) для ssh-ключа

TELEGRAM_TO                    # id телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
TELEGRAM_TOKEN                 # токен бота (получить токен можно у @BotFather, /token, имя бота)

Автор

[Timofey - Razborshchikov](https://github.com/Timofey3085)