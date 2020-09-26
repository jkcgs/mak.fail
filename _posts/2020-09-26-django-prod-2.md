---
layout: post
title: "Django en producción (parte 2)"
permalink: /django-prod/parte-2/
---

## Preparar uWSGI

uWSGI nos permite exponer nuestra aplicación corriendo a través de
un puerto o un socket UNIX, y este último siendo el que se utilizará
y que después Nginx tomará para exponerlo mediente HTTP.

### Instalación

Si se siguió la guía anterior, ya deberías tener instalado Python 3
y `pip`, que es el gestor de dependencias y paquetes de Python.
Ejecuta el siguiente comando para instalar uWSGI:

```
sudo pip3 install uwsgi
```

Con `pip` además se pueden instalar paquetes de forma independiente
del sistema operativo, y que funcionan como ejecutables del sistema.
En este caso, `uwsgi` queda en la ruta `/usr/local/bin/uwsgi`.

### Configuración

Teniendo uWSGI, este debe ser ejecutado como servicio, mediante el
modo "Emperor", el cual ejecutará a sus "vassals" desde una serie
de archivos configurables en una carpeta específica. Ejecutar el
siguiente comando para crear las carpetas correspondientes:

```
sudo mkdir -p /etc/uwsgi/vassals
```

Luego, crear el siguiente archivo con su contenido:

```
sudo vim /etc/uwsgi/emperor.ini
```

```ini
[uwsgi]
emperor = /etc/uwsgi/vassals
```

### uWSGI como servicio de Systemd

De modo de que uWSGI se ejecute desde el inicio del sistema, se debe
registrar como servicio, creando el siguiente archivo con su contenido:

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

Y luego, ejecutar los siguientes comandos para dejar el servicio corriendo:

```
sudo systemctl daemon-reload
sudo systemctl enable --now uwsgi
```

<div class="flex-browse">
    <a href="/django-prod/parte-1/" class="btn">← Volver a la parte 1</a>
    <a href="/django-prod/parte-3/" class="btn">Siguiente (preparar aplicación) →</a>
</div>