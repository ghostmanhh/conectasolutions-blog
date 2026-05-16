# Conecta Solutions — Blog con Jekyll

Sitio web corporativo y blog técnico construido con Jekyll 3.10.

## Requisitos

- Ruby >= 2.7
- Bundler (`gem install bundler`)

## Instalación local

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-usuario/conectasolutions-blog.git
cd conectasolutions-blog

# 2. Instalar dependencias
bundle install

# 3. Correr el servidor de desarrollo
bundle exec jekyll serve --livereload

# El sitio estará en http://localhost:4000
```

## Estructura del proyecto

```
conectasolutions/
├── _config.yml          # Configuración principal
├── _layouts/
│   ├── default.html     # Layout base (header + footer)
│   └── post.html        # Layout de artículo individual
├── _includes/
│   ├── header.html      # Cabecera del sitio
│   └── footer.html      # Pie de página
├── _posts/              # Artículos del blog
│   └── YYYY-MM-DD-titulo-del-post.md
├── assets/
│   ├── css/main.css     # Estilos principales
│   └── js/main.js       # JavaScript
├── blog/index.html      # Índice del blog (con paginación)
├── sobre-nosotros.md    # Página "Sobre nosotros"
├── contacto.md          # Página de contacto
└── index.html           # Página principal
```

## Cómo publicar un nuevo artículo

1. Crea un archivo en `_posts/` con el formato:
   ```
   YYYY-MM-DD-titulo-del-post.md
   ```

2. Agrega el front matter al inicio:
   ```yaml
   ---
   layout: post
   title: "Título del artículo"
   subtitle: "Descripción corta opcional"
   date: 2025-06-01
   categories: [Datos]
   tags: [sql, datos, tutorial]
   author: "Tu Nombre"
   read_time: 5
   excerpt: "Resumen corto que aparece en el listado del blog."
   ---
   ```

3. Escribe el contenido en Markdown.

4. Commit y push — GitHub Pages lo publica automáticamente.

## Despliegue en GitHub Pages

1. Crea un repositorio en GitHub (puede ser público o privado con plan de pago)
2. Ve a **Settings > Pages**
3. En "Source" selecciona la rama `main` y directorio `/`
4. El sitio estará disponible en `https://tu-usuario.github.io/nombre-repo/`

Para un dominio personalizado (conectasolutions.com):
1. Agrega un archivo `CNAME` en la raíz con el contenido: `conectasolutions.com`
2. Configura los DNS de tu dominio apuntando a GitHub Pages

## Personalización

- **Colores y tipografía**: edita las variables en `assets/css/main.css` (sección `:root`)
- **Información del sitio**: edita `_config.yml`
- **Navegación**: edita `_includes/header.html`
