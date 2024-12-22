## 1. Установка Nginx

Вводим в терминал команду
```Bash
sudo apt install nginx
```

## 2.  Получение своего ip адреса

Вводим в терминал команду
```Bash
ip a
```
В моём случае и везде далее будет фигурировать ip 192.168.10.10

## 3. Настройка конфига Django
Открываем файл backend/backend/settings.py
### 3.1. Меняем данные для подключения к БД
```Python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': '<Имя_Базы_Данных>',
        'USER': 'Имя_пользователя',
        'PASSWORD': 'Пароль_пользователя',
        'HOST': 'Адрес_Базы_Данных',
        'PORT': 'Порт_Базы_данных',
    }
}
```


### 3.2. Меняем доверенные адреса

```Python

CSRF_TRUSTED_ORIGINS = ['http://127.0.0.1:3000','http://192.168.10.10:8080']
CORS_ALLOWED_ORIGINS = ['http://127.0.0.1:3000','http://192.168.10.10:8080']

```

### 3.3. Меняем политику куки

```Python
CSRF_COOKIE_SECURE = False
SESSION_COOKIE_SECURE = False
```


## 4. Подготовка виртуального окружения

```Bash
python3 -m venv env
source env/bin/activate
pip install -r backend/requirements.txt
```

## 5. Запуск Django

```Shell
python3 manage.py makemigrations
python3 manage.py migrate
pip install gunicorn
touch gunicorn_corfig.py
nano gunicorn_config.py
```

```gunicorn_config
command = '<Путь_до_папки_env>/bin/gunicorn'
pythonpath = '<Путь_до_папки_backend>'
bind = '<IP_PORT_Django>'(127.0.0.1:8000)
workers = 3
```

```Shell
gunicorn -c gunicorn_config.py backend.wsgi
```

### 6. Настройка React

Папка frontend
```Shell
npm i --force
npx next build
npx next start -H 127.0.0.1 -p 3000 > /dev/null 2>&1 &
```

## 7. Настройка nginx
```Shell

sudo touch /etc/nginx/sites-available/collectioner
nano /etc/nginx/sites-available/collectioner

```

```collectioner
server{
	listen 8080;
	server_name 192.168.10.10;
	location /api {
		proxy_pass http://127.0.0.1:8000/api;
	}

	location /static {
		proxy_pass http://127.0.0.1:8000/static;
	}

	location /admin-panel-collectioner {
		proxy_pass http://127.0.0.1:8000/admin-panel-collectioner;
	}

	location /media {
		proxy_pass http://127.0.0.1:8000/media;
	}
	location / {
		proxy_pass http://127.0.0.1:3000;
	}
}

```

```shell
sudo service nginx restart
```
