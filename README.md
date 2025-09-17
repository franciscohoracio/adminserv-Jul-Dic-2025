# Repositorio de Prácticas - Administración de Servidores

¡Bienvenido! Este repositorio está diseñado como material de apoyo para la materia de Administración de Servidores. Contiene una serie de prácticas guiadas para introducir a los estudiantes en el mundo de la contenerización utilizando Docker y Docker Compose.

## Prerrequisitos

Antes de comenzar, asegúrate de tener instaladas las siguientes herramientas en tu sistema:

-   **Git:** Para clonar y gestionar las versiones de este repositorio.
-   **Docker Engine:** El motor principal para construir y ejecutar contenedores.
-   **Docker Compose:** Para orquestar aplicaciones multi-contenedor.

## Estructura del Repositorio

El repositorio está organizado en directorios, donde cada uno representa una práctica autocontenida.

```
/
└───docker-practices/
    ├───1-Hola-Mundo/
    ├───2-Servidor-Web/
    ├───3-Modificar-Contenedor/
    ├───4-Datos-Persistentes/
    ├───5-Imagen-Personalizada/
    ├───6-WordPress/
    ├───7-WordPress-Persistente/
    └───8-MultiStage/
```

## Guía de Prácticas

A continuación se detalla el objetivo y los comandos para cada práctica. **Es importante ejecutar los comandos desde el directorio raíz de cada práctica.**

---

### Práctica 1: Hola Mundo

-   **Objetivo:** Familiarizarse con el comando `docker run` ejecutando un contenedor simple que imprime un mensaje en la consola.
-   **Ubicación:** `docker-practices/1-Hola-Mundo`

**Comandos:**

```bash
# Ejecuta un contenedor basado en la imagen 'hello-world'
docker run hello-world
```

---

### Práctica 2: Servidor Web Básico

-   **Objetivo:** Aprender a ejecutar un servidor web Nginx en un contenedor y exponer un puerto a la máquina anfitriona (host).
-   **Ubicación:** `docker-practices/2-Servidor-Web`

**Comandos:**

```bash
# Ejecuta un contenedor Nginx en segundo plano (-d) y mapea el puerto 8080 del host al 80 del contenedor (-p)
docker run -d -p 8080:80 --name webserver nginx
```

-   **Verificación:** Abre tu navegador y visita `http://localhost:8080`. Deberías ver la página de bienvenida de Nginx.

---

### Práctica 3: Modificar un Contenedor en Ejecución

-   **Objetivo:** Aprender a acceder a la terminal de un contenedor en ejecución para inspeccionar o modificar su contenido.
-   **Ubicación:** `docker-practices/3-Modificar-Contenedor`

**Comandos:**

```bash
# 1. Asegúrate de tener el contenedor 'webserver' de la práctica anterior corriendo. Si no, ejecútalo:
docker run -d -p 8080:80 --name webserver nginx

# 2. Accede a la terminal del contenedor
docker exec -it webserver bash

# 3. Dentro del contenedor, modifica la página de bienvenida
echo "<h1>Hola desde el contenedor!</h1>" > /usr/share/nginx/html/index.html
exit
```

-   **Verificación:** Recarga `http://localhost:8080` en tu navegador. El mensaje deberá haber cambiado.

---

### Práctica 4: Datos Persistentes con Volúmenes

-   **Objetivo:** Entender cómo usar volúmenes para persistir datos, de modo que no se pierdan al eliminar un contenedor.
-   **Ubicación:** `docker-practices/4-Datos-Persistentes`

**Comandos:**

```bash
# Ejecuta un contenedor Nginx, montando el directorio actual (.) en el directorio web del contenedor
docker run -d -p 8080:80 --name webserver-persistente -v "$(pwd)/index.html":/usr/share/nginx/html/index.html nginx
```

-   **Verificación:**
    1.  Visita `http://localhost:8080`. Verás el contenido del `index.html` local.
    2.  Modifica el archivo `index.html` en tu máquina.
    3.  Recarga la página en el navegador. Los cambios se reflejarán instantáneamente.

---

### Práctica 5: Creación de una Imagen Personalizada

-   **Objetivo:** Aprender a escribir un `Dockerfile` para crear una imagen de Docker personalizada que contenga una aplicación simple.
-   **Ubicación:** `docker-practices/5-Imagen-Personalizada`

**Comandos:**

```bash
# 1. Construye la imagen a partir del Dockerfile y etiquétala como 'mi-web'
docker build -t mi-web .

# 2. Ejecuta un contenedor a partir de tu nueva imagen
docker run -d -p 8080:80 --name mi-web-personalizada mi-web
```

-   **Verificación:** Abre `http://localhost:8080` en tu navegador para ver tu página personalizada.

---

### Práctica 6: Orquestación con Docker Compose (WordPress)

-   **Objetivo:** Introducir `docker-compose` para definir y ejecutar una aplicación multi-contenedor (WordPress y su base de datos MySQL).
-   **Ubicación:** `docker-practices/6-WordPress`

**Comandos:**

```bash
# Levanta los servicios (WordPress y MySQL) definidos en docker-compose.yml
docker-compose up -d
```

-   **Verificación:**
    -   Visita `http://localhost:8081` para iniciar la instalación de WordPress.
    -   Para detener y eliminar los contenedores, usa: `docker-compose down`. **Nota: La base de datos se perderá.**

---

### Práctica 7: WordPress con Datos Persistentes

-   **Objetivo:** Mejorar la práctica anterior añadiendo volúmenes a `docker-compose` para que los datos de la base de datos y los archivos de WordPress persistan.
-   **Ubicación:** `docker-practices/7-WordPress-Persistente`

**Comandos:**

```bash
# Levanta los servicios con persistencia de datos
docker-compose up -d
```

-   **Verificación:**
    -   Visita `http://localhost:8082` para instalar WordPress.
    -   Ejecuta `docker-compose down`. Los datos de la base de datos (`db_data`) y los archivos de WordPress (`wp-content`) permanecerán en tu disco local.
    -   Vuelve a ejecutar `docker-compose up -d`. Tu sitio de WordPress seguirá intacto.

---

### Práctica 8: Builds Multi-Etapa (Multi-Stage)

-   **Objetivo:** Aprender a utilizar builds multi-etapa para crear imágenes de producción optimizadas, pequeñas y seguras, usando el ejemplo de una aplicación React.
-   **Ubicación:** `docker-practices/8-MultiStage`

**Comandos:**

```bash
# 1. Construye la imagen. Docker se encargará de la etapa de build y la de producción.
docker build -t react-app-prod .

# 2. Ejecuta la aplicación final en un contenedor Nginx
docker run -d -p 8080:80 --name mi-app-react react-app-prod
```

-   **Verificación:** Visita `http://localhost:8080` para ver la aplicación de React en ejecución. Compara el tamaño de esta imagen (`react-app-prod`) con una imagen de desarrollo que incluyera todas las dependencias de Node.js para notar la diferencia.
