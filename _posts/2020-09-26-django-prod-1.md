---
layout: post
title: "Django en producción (parte 1)"
permalink: /django-prod/parte-1/
---

## Preparación de ambiente

En esta sección se preparará el sistema operativo para ejecutar todo
lo necesario, es decir, instalar el software necesario y otras dependencias.

### Configuración de mirrors

Aquí configuraremos e instalaremos software desde los repositorios de Debian.
Recordar que esto es pensando en una máquina nueva.

1\. Instala vim porque es sencillo para reemplazar texto en un archivo, en este
caso, para cambiar los mirror en uso con APT.

```
sudo apt update
sudo apt install vim
```

2\. Actualizar los mirrors en el archivo de lista de mirrors.
```
sudo vim /etc/apt/sources.list
```

Debería tener al menos las siguientes líneas:

```
deb http://security.debian.org buster/updates main
deb-src http://security.debian.org buster/updates main
deb http://deb.debian.org/debian/ buster main contrib
deb-src http://deb.debian.org/debian/ buster main contrib
deb http://deb.debian.org/debian/ buster-updates main contrib
deb-src http://deb.debian.org/debian/ buster-updates main contrib
```

Para reemplazar los mirror (excepto el de Security), escribe en `vim`:

```
:%s/deb.debian.org/mirror.ufro.cl
```

Y quedará algo así:

```deb
deb http://security.debian.org buster/updates main
deb-src http://security.debian.org buster/updates main
deb http://deb.debian.org/debian/ buster main contrib
deb-src http://deb.debian.org/debian/ buster main contrib
deb http://deb.debian.org/debian/ buster-updates main contrib
deb-src http://deb.debian.org/debian/ buster-updates main contrib
```

Si hay líneas relacionadas con el "cd-rom" de Debian, eliminarlas. En vim,
si te posicionas sobre la línea que quieres borrar y pulsas `dd`,
se eliminará la línea completa.

3\. Actualiza los repositorios y actualiza el software del sistema
con los siguientes comandos:

```
sudo apt update
sudo apt upgrade
```

Si se actualizó el kernel (se actualizaron paquetes llamados linux-*),
deberías reiniciar.

### Instalación de software necesario

Haz la instalación ejecutando los siguientes comandos:

```
sudo apt install python3 python3-pip python3-virtualenv git zip curl gnupg2 ca-certificates lsb-release
```

Como se puede ver, se instala Python 3 y otras necesidades.

### Instalación de Nginx

Estos detalles se obtienen desde [el sitio de Nginx](https://nginx.org/en/linux_packages.html#Debian).

Si lo que estás instalando es realmente producción, ejecuta lo siguiente
para activar el repositorio estable de Nginx:

```
echo "deb http://nginx.org/packages/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

En caso contrario, puedes usar el último release, no necesariamente estable:

```
echo "deb http://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list
```

Luego, ejecutar estos comandos paso a paso, para instalar y ejecutar
el servicio de Nginx:

```
curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
sudo apt update
sudo apt install nginx
sudo systemctl enable --now nginx
```

<div class="flex-browse">
    <a href="/django-prod/" class="btn">← Volver a introducción</a>
    <a href="/django-prod/parte-2/" class="btn">Siguiente (preparar uWSGI) →</a>
</div>
