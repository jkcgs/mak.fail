---
layout: post
title: "Django en producción (versión simple)"
permalink: /django-prod/tldr/
---

1\. Instalar vim y configurar mirror

```
sudo apt update
sudo apt install vim
sudo vim /etc/apt/sources.list
```

```
# Usar este comando de vim
:%s/deb.debian.org/mirror.ufro.cl
# Quedará así (comenta o elimina otras líneas):

deb http://security.debian.org buster/updates main
deb-src http://security.debian.org buster/updates main
deb http://mirror.ufro.cl/debian/ buster main contrib
deb-src http://mirror.ufro.cl/debian/ buster main contrib
deb http://mirror.ufro.cl/debian/ buster-updates main contrib
deb-src http://mirror.ufro.cl/debian/ buster-updates main contrib
```

2\. Guarda. Actualiza los repositorios y el software del sistema:

```
sudo apt update
sudo apt upgrade
```

3\. Instalar software

```
sudo apt install python3 python3-pip python3-virtualenv git zip curl gnupg2 ca-certificates lsb-release
```

4\. Habilitar repos de Nginx (solo uno de los dos):

```
# Estable
echo "deb http://nginx.org/packages/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
# Mainline
echo "deb http://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

Instalar y habilitar el servicio de Nginx:

```
curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
sudo apt update
sudo apt install nginx
sudo systemctl enable --now nginx
```

5\. uWSGI:

```
sudo pip3 install uwsgi
sudo mkdir -p /etc/uwsgi/vassals
sudo vim /etc/uwsgi/emperor.ini
```

```ini
[uwsgi]
emperor = /etc/uwsgi/vassals
```

Crear service unit para Systemd

```
sudo vim /etc/systemd/system/uwsgi.service
```

```ini
[Unit]
Description=uWSGI Emperor
After=syslog.target

[Service]
ExecStart=/usr/local/bin/uwsgi --ini /etc/uwsgi/emperor.ini
RuntimeDirectory=uwsgi
Restart=always
KillSignal=SIGQUIT
Type=notify
StandardError=syslog
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

Activar servicio uWSGI

```
sudo systemctl daemon-reload
sudo systemctl enable --now uwsgi
```

6\. Activar proyecto mediante Nginx:

Se asume que el proyecto está listo, configurado, y en la ruta `/opt/project`.
Además, que el módulo principal de Django es `system`. Además, que los
archivos estáticos están en `/opt/project/collected_static`.

Archivo "vassal" de uWSGI del proyecto:

```
sudo vim /etc/uwsgi/vassal/project.ini
```

```ini
[uwsgi]
chdir  = /opt/project
module = system.wsgi
home   = /opt/project/.venv

uid          = nginx
gid          = nginx
master       = true
processes    = 2
threads      = 2
socket       = /tmp/project.sock
chmod-socket = 664
vacuum       = true
```

Configurar el projecto para ser expuesto por Nginx.

```
sudo vim /etc/nginx/conf.d/project.conf
```

```
server {
    listen 80;
    server_name _;

    # Cambiar según la ubicación de los estáticos y su URL
    location /static/ {
        alias   /opt/project/collected_static/;
        gzip on;
    }

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;

        uwsgi_pass unix:/tmp/project.sock;
        include uwsgi_params;
    }
}
```

Verificar configuración y reiniciar Nginx:

```
sudo nginx -t
sudo systemctl restart nginx
```

<div class="flex-browse">
    <a href="/django-prod/" class="btn">← Volver a introducción</a>
</div>
