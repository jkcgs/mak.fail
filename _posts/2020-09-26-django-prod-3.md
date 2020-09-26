---
layout: post
title: "Django en producción (parte 3)"
permalink: /django-prod/parte-3/
---

## Preparar aplicación Django

Esto es un copy-paste de lo que queda por ordenar

### Clonar repositorio y configurar

Se clonará un repositorio desde Github y se dejará en `/opt`. Luego, se
utilizará al usuario de `nginx` para que tenga permisos de escritura sobre
la carpeta del proyecto, con el cual se ejecutará el resto de comandos.

Se entiende que el proyecto se llama "project" y se ubicará en
`/opt/project`.

```
cd /opt
sudo git clone http://github.com/user/project.git
sudo chown -R nginx:nginx project
cd project
sudo -u nginx python3 -m virtualenv -p python3 .venv
sudo -u nginx .venv/bin/pip install -r requirements.txt
```

Los proyectos basados en Django los suelo hacer de modo que las configuraciones
quede en el archivo `local_settings.py`, guardados en el repositorio como
`local_settings.example.py`, de modo que hay que copiarlo y editarlo.

```
sudo -u nginx cp local_settings{.example,}.py
sudo -u nginx vim local_settings.py
```

### Configurar "vassal" de uWSGI

La aplicación se debe exponer como "vassal" de uWSGI, por ello se creará
un archivo de configuración que cumplirá este propósito. Tener en cuenta que
mis proyectos Django tienen su estructura cambiada. Todos tienen la carpeta
del proyecto inicial con el nombre "system", por ende, toda la configuración
base se encuentra en esa carpeta. Revisar la referencia en el archivo
de configuración de *vassal* de abajo.

```
sudo vim /etc/uwsgi/vassals/project.ini
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

log-x-forwarded-for = true
log-format      = %(addr) - %(user) "%(method) %(uri) %(proto)" %(status) %(size) "%(referer)" "%(uagent)"
```

### Configurar Nginx

Por último, se debe crear la configuración de Nginx para que 
dirija las peticiones correspondientes a la aplicación web.

En primer lugar, se debe ejecutar el comando `collectstatic` de
Django para que copie los archivos estáticos correspondientes a una carpeta
determinada por la configuración de la aplicación. En esta ocasión se asume
que es "collected_static" dentro de la carpeta del proyecto.

Ejecutar el siguiente comando dentro de la carpeta del proyecto y
aceptar lo que diga el diálogo:

```
sudo -u nginx .venv/bin/python manage.py collectstatic
```

Con esto, crear la siguiente configuración de Nginx:

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

Luego, ejecutar los dos siguientes comandos. Si el primero dice que la
configuración está bien, ejecutar el segundo. Si ocurre un error, corregir
primero antes de correr el siguiente comando.

```
sudo nginx -t
sudo systemctl restart nginx
```

Con esto, la aplicación estará corriendo en http://127.0.0.1/ o en la IP
a la cual tengas acceso al servidor.

<div class="flex-browse">
    <a href="/django-prod/parte-2/" class="btn">← Volver a la parte 2</a>
</div>