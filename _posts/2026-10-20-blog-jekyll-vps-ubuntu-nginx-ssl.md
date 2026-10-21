---
layout: post
title: "Cómo montar un blog con Jekyll en un VPS Ubuntu 22.04 con Nginx y SSL"
date: 2026-10-20
categories: [Infraestructura]
excerpt: "Jekyll genera HTML estático, Nginx lo sirve y Certbot pone el candado. Así lo montamos en un VPS, sin instalar nada en la máquina local."
---

Un blog no necesita WordPress, base de datos ni un panel pesado. Con **Jekyll** escribes en Markdown, se genera HTML estático y un servidor web común lo entrega en milisegundos. Esta es la configuración que usamos para este mismo sitio, sobre un VPS con Ubuntu 22.04.

## Por qué un sitio estático

- **Seguridad:** no hay base de datos ni código ejecutándose en el servidor, así que la superficie de ataque es mínima.
- **Velocidad:** Nginx entrega archivos ya generados.
- **Costo:** un VPS pequeño es suficiente.
- **Control:** todo el contenido son archivos de texto que puedes versionar con Git.

## 1. Instalar Ruby y Jekyll

```bash
sudo apt update
sudo apt install -y ruby-full build-essential zlib1g-dev

echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

gem install jekyll bundler
```

Configurar `GEM_HOME` evita instalar gemas con `sudo`, que es una fuente común de problemas de permisos.

## 2. Crear el sitio y generarlo

```bash
jekyll new conectasolutions
cd conectasolutions
bundle exec jekyll build
```

El resultado queda en la carpeta `_site/`. Esa es la carpeta que Nginx va a servir.

## 3. Servirlo con Nginx

```bash
sudo apt install -y nginx
sudo nano /etc/nginx/sites-available/conectasolutions
```

```nginx
server {
    listen 80;
    server_name tudominio.com www.tudominio.com;

    root /home/usuario/conectasolutions/_site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/conectasolutions /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Antes de continuar, apunta el registro **A** de tu dominio a la IP del VPS.

## 4. Activar HTTPS con Certbot

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d tudominio.com -d www.tudominio.com
```

Certbot modifica la configuración de Nginx y programa la renovación automática. Puedes comprobarla con `sudo certbot renew --dry-run`.

## Errores que nos encontramos

**Permisos en `_site/`.** Si Nginx responde 403, casi siempre es que no puede entrar a tu carpeta personal. Se resuelve con `chmod o+x /home/usuario` o moviendo el sitio a `/var/www/`.

**Un 404 donde esperabas una página.** En Jekyll cada página necesita su `permalink` en el encabezado. Si el archivo se llama `libros.md` pero el menú apunta a `/libros-recursos/`, el enlace falla. Revisa que ambos coincidan.

**Fechas con `\n` literal.** Un formato de fecha mal escrito en la plantilla mostraba `8 May\n2025`. Se corrige usando un formato limpio como `{% raw %}{{ post.date | date: "%-d %b %Y" }}{% endraw %}`.

**El cambio no se ve.** Después de editar, ejecuta `bundle exec jekyll build` y recarga con `Ctrl + F5`.

## Flujo de trabajo diario

1. Creas un archivo `AAAA-MM-DD-titulo.md` en `_posts/`.
2. Ejecutas `bundle exec jekyll build`.
3. Nginx publica el cambio de inmediato.

Si no quieres instalar nada en tu equipo, conéctate al VPS desde VS Code con la extensión Remote SSH y edita los archivos directamente en el servidor.
