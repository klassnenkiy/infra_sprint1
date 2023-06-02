# Kittygram - социальная сеть для размещения фотографий котиков

## Описание проекта
Kittygram - это социальная сеть, разработанная в рамках учебного курса по Python от Яндекс.Практикум. Она позволяет пользователям регистрироваться, загружать фотографии своих котиков с кратким описанием и просматривать фотографии котиков других пользователей. Основная цель проекта - обучение процессу деплоя приложения на сервер.

## Технологии

- Django==3.2.3
- djangorestframework==3.12.4
- djoser==2.1.0
- webcolors==1.11.1
- Pillow==9.0.0
- pytest==6.2.4
- pytest-django==4.4.0
- pytest-pythonpath==0.7.3
- python-dotenv==1.0.0
 
## Установка проекта на локальный компьютер из репозитория 
Чтобы установить проект на локальном компьютере, выполните следующие шаги:

- Склонируйте репозиторий с помощью команды git clone <адрес репозитория>.
- Перейдите в директорию с клонированным репозиторием.
- Создайте виртуальное окружение с помощью команды python3 -m venv venv.
- Активируйте виртуальное окружение.
- Установите зависимости, выполнив команду pip install -r requirements.txt.
- Создайте файл .env в директории /backend/kittygram_backend/.
- В файле .env установите ваш SECRET_KEY в формате: SECRET_KEY = '<ваш_ключ>'.

# Деплой проекта на удаленный сервер
Чтобы задеплоить проект на удаленном сервере, выполните следующие шаги:
## Подключение сервера к аккаунту на GitHub
- Убедитесь, что на сервере установлен Git. Выполните команды sudo apt update и git --version, чтобы проверить наличие Git.
- Если Git не установлен, установите его с помощью команды sudo apt install git.
- Сгенерируйте пару SSH-ключей на сервере с помощью команды ssh-keygen.
- Сохраните открытый ключ (от символов ssh-rsa до конца) и добавьте его к вашему аккаунту на GitHub.
- Склонируйте проект с GitHub на сервер с помощью команды git clone git@github.com:<ваш_аккаунт>/<имя_проекта>.git.

## Запуск backend проекта на сервере
- Установить пакетный менеджер и утилиту для создания виртуального окружения `sudo apt install python3-pip python3-venv -y`
- находясь в директории с проектом создать и активировать виртуальное окружение `python3 -m venv venv`  `source venv/bin/activate` 
- установить зависимости `pip install -r requirements.txt`
- выполнить миграции `python manage.py migrate`
- создать суперюзера `python manage.py createsuperuser`
- отредактировать settings.py на сервере: в список ALLOWED_HOSTS добавить внешний IP-адрес вашего сервера и адреса `127.0.0.1` и `localhost` . ALLOWED_HOSTS = ['<внешний адрес вашего сервера>', '127.0.0.1', 'localhost']

## Запуск frontend проекта на сервере
- установить на сервер `Node.js`   командами
`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\`
`sudo apt-get install -y nodejs`
- установить зависимости frontend приложения. Из директории `<ваш проект>/frontend/` выполнить команду: `npm i`

## Установка и запуск Gunicorn
- при активированном виртуальном окружении проекта установить пакет gunicorn `pip install gunicorn==20.1.0`
- открыть файл settings.py проекта и установить для константы `DEBUG` значение `False` `DEBUG = False`
- В директории _/etc/systemd/system/_ создайте файл _gunicorn.service_ `sudo nano /etc/systemd/system/gunicorn.service`  со следующим кодом (без комментариев):

	    [Unit]
    
	    Description=gunicorn daemon
    
	    After=network.target
    
	    [Service]
    
	    User=yc-user
    
	    WorkingDirectory=/home/<имя пользователя в системе>/<имя проекта>/backend/
    
	    ExecStart=/home/<имя пользователя в системе>/<имя проекта>/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
    
	    [Install]
    
	    WantedBy=multi-user.target

Чтобы точно узнать путь до Gunicorn можно при активированном виртуальном окружении использовать команду `which gunicorn`

## Установка и настройка Nginx

 - На сервере из любой директории выполнить команду: `sudo apt install nginx -y`
- Для установки ограничений на открытые порты выполнить по очереди команды: `sudo ufw allow 'Nginx Full'`  `sudo ufw allow OpenSSH`
- включить файервол `sudo ufw enable`
	### собрать и разместить статику frontend-приложения.
- Перейти в директорию _<имя_проекта>/frontend/_  и выполнить команду `npm run build` Результат сохранится в директории ..._/frontend/build/_.  В системную директорию сервера _/var/www/_ скопировать содержимое папки _/frontend/build/_
- открыть файл конфигурации веб-сервера `sudo nano /etc/nginx/sites-enabled/default` и заменить его содержимое следующим кодом:

    	
        server {
    
	        listen 80;
	        server_name публичный_ip_вашего_удаленного_сервера;
        
	        location / {
            root   /var/www/<имя проекта>;
            index  index.html index.htm;
            try_files $uri /index.html;
	        }
    
	    }
- проверить корректность конфигурации `sudo nginx -t`
- перезагрузить конфигурацию Nginx `sudo systemctl reload nginx`
	### настроить проксирование запросов
- Открыть файл конфигурации Nginx _/etc/nginx/sites-enabled/default_ и добавить в него ещё один блок `location`

	    server {
    
	        listen 80;
	        server_name публичный_ip_вашего_удаленного_сервера;
    
	        location /api/ {
	            proxy_pass http://127.0.0.1:8080;
	        }
	        
		    location /admin/ {
			    proxy_pass http://127.0.0.1:8000;
				}
			
	        location / {
	            root   /var/www/<имя_проекта>;
	            index  index.html index.htm;
	            try_files $uri /index.html;
	        }
    
	    }

- Сохранить изменения, проверить и перезагрузить конфигурацию веб-сервера:

	    sudo nginx -t
	    sudo systemctl reload nginx

	### собрать и настроить статику для backend-приложения.
- в файле _settings.py_ прописать настройки 
	

	    STATIC_URL = 'static_backend'
	    STATIC_ROOT = BASE_DIR / 'static_backend'

- активировать виртуальное окружение проекта, перейти в директорию с файлом _manage.py_ и выполнить команду `python manage.py collectstatic`
- в директории_<имя_проекта>/backend/_ будет создана директория _static_backend/_ 
- Скопировать директорию _static_backend/_ в директорию _/var/www/<имя_проекта>/_

## Добавление доменного имени в настройки Django
- в файле _settings.py_ добавить в список `ALLOWED_HOSTS` доменное имя: 
	ALLOWED_HOSTS = ['ip_адрес_вашего_сервера', '127.0.0.1', 'localhost', 'ваш-домен'] 
- сохранить изменения и перезапустить gunicorn `sudo systemctl restart gunicorn`
- внести изменения в конфигурацию Nginx. Открыть конфигурационный файл командой: `sudo nano /etc/nginx/sites-enabled/default`
- Добавьте в строку `server_name` выданное вам доменное имя (через пробел, без `< >`):

		server {
		...
		    server_name <ваш-ip> <ваш-домен>;
		...
		}
- Проверить конфигурацию `sudo nginx -t` и перезагрузить её командой `sudo systemctl reload nginx`, чтобы изменения вступили в силу.

 ## Получение и настройка SSL-сертификата
 **Установка certbot**
 - Зайдите на сервер и последовательно выполните команды:

	    sudo apt install snapd
	    sudo snap install core; sudo snap refresh core
	    sudo snap install --classic certbot
	    sudo ln -s /snap/bin/certbot /usr/bin/certbot
- запустить certbot и получить SSL-сертификат:

		sudo certbot --nginx
- сертификат автоматически сохранится на вашем сервере в системной директории _/etc/ssl/_  Также будет автоматически изменена конфигурация Nginx: в файл _/etc/nginx/sites-enabled/default_ добавятся новые настройки и будут прописаны пути к сертификату.
- перезагрузить конфигурацию Nginx `sudo systemctl reload nginx`

## Автор проекта
Станислав Тюлягин - [GitHub](https://github.com/klassnenkiy)



