# Práctica 6: Orquestación con Docker Compose

## Objetivo
Hasta ahora, hemos manejado contenedores de forma individual. El objetivo de esta práctica es aprender a definir y ejecutar aplicaciones compuestas por múltiples contenedores (como un sitio web y su base de datos) usando `Docker Compose`. Esto simplifica enormemente la gestión de aplicaciones complejas.

## Conceptos Clave
- **Docker Compose:** Una herramienta para definir y correr aplicaciones Docker multi-contenedor. Utiliza un archivo YAML (`docker-compose.yml`) para configurar los servicios de la aplicación.
- **`docker-compose.yml`:** El corazón de Docker Compose. En este archivo se definen:
  - **Servicios (`services`):** Cada servicio corresponde a un contenedor. En nuestro caso, tendremos un servicio `wordpress` y un servicio `db`.
  - **Redes (`networks`):** Compose crea una red privada por defecto para todos los servicios del archivo, permitiendo que se comuniquen entre sí usando sus nombres de servicio como si fueran hostnames.
  - **Variables de Entorno (`environment`):** Una forma de pasar configuración a los contenedores, como credenciales de base de datos.
- **`depends_on` (no presente, pero importante):** Una directiva que puede usarse para indicar que un servicio depende de otro (aunque no espera a que esté "listo", solo a que se inicie).

## Archivo `docker-compose.yml`
Este es el archivo que define nuestra aplicación WordPress:

```yaml
version: '3.3'

services:
   db:
     image: mysql:5.7
     restart: always
     environment:
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: password
       MYSQL_RANDOM_ROOT_PASSWORD: '1'

   wordpress:
     image: wordpress:latest
     restart: always
     ports:
       - 8081:80 # Ojo, el puerto es 8081
     environment:
       WORDPRESS_DB_HOST: db:3306 # Se conecta al servicio 'db'
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: password
```

### Análisis del Archivo
- **Servicio `db`:**
  - Usa la imagen `mysql:5.7`.
  - `restart: always` asegura que el contenedor se reinicie si se cae.
  - `environment` configura la base de datos MySQL (nombre, usuario y contraseña).
- **Servicio `wordpress`:**
  - Usa la última imagen de `wordpress`.
  - `ports: - 8081:80` mapea el puerto 80 del contenedor al 8081 de nuestra máquina.
  - `environment` le dice a WordPress cómo conectarse a la base de datos. **¡Nota clave!** `WORDPRESS_DB_HOST` es `db:3306`. `db` es el nombre del servicio de la base de datos. Docker Compose hace que este nombre sea resoluble en la red interna que crea.

## Instrucciones

### 1. Iniciar la Aplicación
Con Docker Compose, ya no usamos `docker run`. Desde la carpeta `6-WordPress`, simplemente ejecuta:

```bash
# -d para modo desacoplado (detached)
docker-compose up -d
```
Docker Compose leerá el archivo `docker-compose.yml`, descargará las imágenes `wordpress` y `mysql` si no las tienes, y creará e iniciará los dos contenedores.

### 2. Verificar el Estado
Puedes ver los contenedores en ejecución con:
```bash
docker-compose ps
```
O con el comando de siempre, `docker ps`.

### 3. Instalar WordPress
Visita `http://localhost:8081` en tu navegador. Verás el famoso instalador de WordPress de 5 minutos. Completa la instalación.

## ¡Advertencia Importante! Datos No Persistentes
Este `docker-compose.yml` tiene un gran defecto con fines didácticos: **no hemos definido volúmenes para la base de datos**. Esto significa que toda la configuración de WordPress y el contenido que crees (posts, páginas, etc.) se almacenan dentro del contenedor `db`.

**Si ejecutas `docker-compose down`, que detiene y elimina los contenedores, ¡perderás toda tu base de datos!**

Esta es una lección fundamental que se resolverá en la siguiente práctica.

## Limpieza
Para detener y **eliminar** los contenedores, la red y otros recursos creados por `up`:

```bash
docker-compose down
```