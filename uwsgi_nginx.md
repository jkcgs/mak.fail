---
layout: default
title: "Por ordenar"
---

## Contenido random

```
sudo apt install vim
sudo vim /etc/apt/sources.list

deb http://deb.debian.org/debian/ buster main contrib
deb-src http://deb.debian.org/debian/ buster main contrib

deb http://deb.debian.org/debian/ buster-updates main contrib
deb-src http://deb.debian.org/debian/ buster-updates main contrib

:%s/deb.debian.org/mirror.ufro.cl

deb http://mirror.ufro.cl/debian/ buster main contrib
deb-src http://mirror.ufro.cl/debian/ buster main contrib

deb http://mirror.ufro.cl/debian/ buster-updates main contrib
deb-src http://mirror.ufro.cl/debian/ buster-updates main contrib

* Comment out cd-rom entries

sudo apt update
sudo apt upgrade

sudo apt install python3 python3-pip python3-virtualenv git zip curl gnupg2 ca-certificates lsb-release

# https://nginx.org/en/linux_packages.html#Debian

# Not prod
echo "deb http://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
# Prod
echo "deb http://nginx.org/packages/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list

curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -

sudo apt update
sudo apt install nginx
sudo systemctl enable --now nginx

sudo pip3 install uwsgi

sudo mkdir -p /etc/uwsgi/vassals
sudo vim /etc/uwsgi/emperor.ini

[uwsgi]
emperor = /etc/uwsgi/vassals

sudo vim /etc/systemd/system/uwsgi.service

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

sudo systemctl daemon-reload
sudo systemctl enable --now uwsgi


# Oracle

wget https://download.oracle.com/otn_software/linux/instantclient/19600/instantclient-basic-linux.x64-19.6.0.0.0dbru.zip
sudo mkdir /opt/oracle
cd /opt/oracle/
sudo unzip ~/instantclient-basic-linux.x64-19.6.0.0.0dbru.zip
sudo apt install libaio1

sudo vim /etc/profile.d/oracleinstantclient.sh
export LD_LIBRARY_PATH=/opt/oracle/instantclient_19_6:$LD_LIBRARY_PATH
export PATH=/opt/oracle/instantclient_19_6:$PATH
sudo chmod +x /etc/profile.d/oracleinstantclient.sh
sudo sh -c "echo /opt/oracle/instantclient_19_6 > /etc/ld.so.conf.d/oracle-instantclient.conf"
sudo ldconfig

reboot


# git pull

cd /opt
sudo git clone http://github.com/user/project.git
sudo chown -R nginx:nginx project
cd project
sudo -u nginx python3 -m virtualenv -p python3 .venv
sudo -u nginx .venv/bin/pip install -r requirements.txt

# Oracle
sudo -u nginx .venv/bin/pip install cx_Oracle
# End oracle

sudo -u nginx cp local_settings{.example,}.py
sudo -u nginx vim local_settings.py  # Editar ajustes

sudo vim /etc/uwsgi/vassals/project.ini

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

log-x-forwarded-for = true
log-format      = %(addr) - %(user) "%(method) %(uri) %(proto)" %(status) %(size) "%(referer)" "%(uagent)"


sudo vim /etc/nginx/conf.d/project.conf

server {
    listen 80;
    server_name _;

    # Cambiar según la ubicación de los estáticos y su URL
    location /bstatic/ {
        alias   /opt/project/collected_static/;
        gzip on;
    }

    location ~ ^/(admin|api)/ {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;

        uwsgi_pass unix:/tmp/project.sock;
        include uwsgi_params;
    }

    location / {
        # Cambiar según la ubicación del front
        root /opt/project-front;
        try_files $uri /index.html;
    }
}



sudo usermod -aG nginx normaluser
# Relog-in

# Revisar que la configuración esté ok
sudo nginx -t
# Aplicar configuración
sudo systemctl restart nginx

# frontend

sudo apt install rsync
sudo mkdir /opt/project-frontend/
sudo chown -R nginx:nginx /opt/project-frontend/
```