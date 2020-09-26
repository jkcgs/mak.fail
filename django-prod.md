---
layout: post
title: "Django en producción"
permalink: /django-prod/
---

## Introducción

Durante mi tiempo como desarrollador web de aplicaciones Python, he realizado
esta configuración incontables veces, y siempre estoy copiando y pegando de
aquí por acá las configuraciones, entonces mi objetivo es colocar toda esa
información en esta guía, que espero que le sirva a más personas también.

## Resumen

En esta guía el objetivo será llevar una aplicación desarrollada en Django
en producción en un servidor Linux (Debian Buster), con Nginx y uwsgi. 

Esto fue realizado en una máquina virtual nueva ("fresh"), la última versión
estable de Debian a la fecha (Buster, 10.5), que viene con Python 3.7.3,
instalando Nginx desde sus repositorios propios, e instalando uwsgi desde
pip, dado a que si se instala desde los repositorios de Debian, presenta
algunas incompatibilidades con algunas aplicaciones Python.

Se expone la aplicación mediante Nginx y uwsgi en modo "emperor", y cuyos 
servicios se ejecutan mediante Systemd.

Para continuar, ve a la siguiente parte de esta guía.

<a href="/django-prod/parte-1/" class="btn">Django en producción: parte 1 →</a>
