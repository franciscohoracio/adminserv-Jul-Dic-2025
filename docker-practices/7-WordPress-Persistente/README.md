# Práctica 7: WordPress con Datos Persistentes

## Objetivo
Solucionar el problema de la pérdida de datos de la práctica anterior. En esta lección, aprenderemos a usar **volúmenes** con Docker Compose para asegurar que los datos de nuestra aplicación (la base de datos de WordPress y sus archivos) persistan incluso después de que los contenedores sean eliminados.

## Conceptos Clave
- **Volúmenes Nombrados (`Named Volumes`):** Son la forma recomendada de persistir datos en Docker. Son áreas de almacenamiento que Docker gestiona. Nosotros solo les damos un nombre. Son independientes del ciclo de vida del contenedor y del sistema de archivos del host.
- **Bind Mounts:** Como vimos en la práctica 4, mapean un directorio del host a un directorio del contenedor. Son útiles para desarrollo, cuando queremos ver cambios del código al instante.
- **Directiva `volumes`:**
  - **A nivel raíz:** Declara los volúmenes nombrados que estarán disponibles para los servicios.
  - **Dentro de un servicio:** Monta un volumen (nombrado o bind mount) en una ruta específica dentro del contenedor.

## Archivo `docker-compose.yml`
Esta versión mejorada introduce la persistencia:

```yaml
version: '3.3'

services:
   db:
     image: mysql:5.7
     # Monta el volumen nombrado 'db_data' en la carpeta de datos de MySQL
     volumes:
       - db_data:/var/lib/mysql
     restart: always
     environment:
       # ... (igual que antes)

   wordpress:
     image: wordpress:latest
     # Monta la carpeta local 'wp-content' en la carpeta de contenido de WordPress
     volumes:
       - ./wp-content:/var/www/html/wp-content
     ports:
       - 8082:80 # Ojo, el puerto es 8082
     restart: always
     environment:
       # ... (igual que antes)

# Declara el volumen nombrado que usará el servicio 'db'
volumes:
    db_data:
```

### Análisis de los Cambios
- **Servicio `db`:**
  - `volumes: - db_data:/var/lib/mysql`: Esta línea le dice a Docker que monte el volumen llamado `db_data` en la ruta `/var/lib/mysql` dentro del contenedor. `/var/lib/mysql` es donde MySQL guarda todos sus datos. Ahora, los datos de la base de datos vivirán en el volumen `db_data`, fuera del contenedor.
- **Servicio `wordpress`:**
  - `volumes: - ./wp-content:/var/www/html/wp-content`: Aquí usamos un **bind mount**. Se creará una carpeta `wp-content` en nuestro directorio local. Esta carpeta se sincronizará con la del contenedor. Ahí es donde WordPress guarda temas, plugins y archivos subidos. Esto es genial para el desarrollo de temas o plugins.
- **Sección `volumes` a nivel raíz:**
  - `volumes: db_data:`: Aquí se declara formalmente el volumen nombrado `db_data`. Si no se especifica un `driver`, Docker usa el driver por defecto, que lo crea y gestiona localmente.

## Instrucciones

### 1. Iniciar la Aplicación
```bash
docker-compose up -d
```
Al ejecutar esto, Docker creará el volumen `db_data` y la carpeta `wp-content` si no existen.

### 2. Instalar WordPress y Crear Contenido
- Visita `http://localhost:8082` y completa la instalación de WordPress.
- Una vez dentro, crea un nuevo Post de prueba. Por ejemplo, con el título "Mi primer post persistente".

### 3. Probar la Persistencia
Este es el momento de la verdad.

1.  **Detén y elimina los contenedores:**
    ```bash
    docker-compose down
    ```
    Este comando elimina los contenedores y la red, pero **no toca los volúmenes nombrados ni los bind mounts**.

2.  **Vuelve a iniciar la aplicación:**
    ```bash
    docker-compose up -d
    ```
    Compose reutilizará el volumen `db_data` existente y la carpeta local `wp-content`.

### 4. Verificar el Resultado
- Visita `http://localhost:8082` de nuevo. **No verás el instalador**. Serás redirigido directamente a tu sitio de WordPress ya configurado.
- Busca tu post de prueba. ¡Ahí estará! Tus datos han sobrevivido.

## Limpieza Final
Cuando quieras eliminar todo, incluyendo la base de datos, debes usar la bandera `-v` en el comando `down`.

```bash
# Detiene, elimina contenedores Y elimina los volúmenes nombrados
docker-compose down -v
```
**Nota:** Este comando no elimina los bind mounts (la carpeta `wp-content`), ya que es una carpeta de tu proyecto. Deberás borrarla manualmente si lo deseas.