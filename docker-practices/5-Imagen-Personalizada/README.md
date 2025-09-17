# Práctica 5: Creación de Imágenes Personalizadas con Dockerfile

## Objetivo
Dar el siguiente paso en Docker: en lugar de usar imágenes existentes, aprenderemos a construir nuestras propias imágenes personalizadas usando un archivo de instrucciones llamado `Dockerfile`. Esto nos permite empaquetar nuestra aplicación y sus archivos directamente en una imagen, haciéndola portable y fácil de distribuir.

## Conceptos Clave
- **`Dockerfile`:** Un archivo de texto plano que contiene una secuencia de instrucciones para construir una imagen de Docker. Es como una receta para nuestra imagen.
- **`docker build`:** El comando que lee el `Dockerfile` y crea una imagen a partir de él.
- **Contexto de Build (`build context`):** El conjunto de archivos en la ubicación especificada (generalmente `.`, el directorio actual) que Docker puede usar durante el proceso de build.
- **Instrucciones del Dockerfile:**
  - `FROM`: Especifica la imagen base sobre la cual construiremos la nuestra.
  - `COPY`: Copia archivos o directorios desde el contexto de build hacia el sistema de archivos de la imagen.
  - `EXPOSE`: Documenta qué puerto(s) el contenedor escuchará en tiempo de ejecución. No publica el puerto, solo sirve como documentación.
  - `CMD`: Define el comando por defecto que se ejecutará cuando se inicie un contenedor a partir de esta imagen.

## Archivos de la Práctica

1.  **`index.html`**: Una página web simple que queremos incluir en nuestra imagen.
    ```html
    <!DOCTYPE html>
    <html>...</html>
    ```
2.  **`Dockerfile`**: Nuestra "receta" para la imagen.
    ```dockerfile
    # Usar una imagen base ligera de Nginx
    FROM nginx:stable-alpine

    # Copiar nuestro index.html al directorio web de Nginx dentro de la imagen
    COPY index.html /usr/share/nginx/html

    # Documentar que el contenedor usará el puerto 80
    EXPOSE 80

    # Comando para iniciar Nginx cuando el contenedor arranque
    CMD ["nginx", "-g", "daemon off;"]
    ```

## Instrucciones

### 1. Construir la Imagen
Desde la terminal, y ubicados en la carpeta `5-Imagen-Personalizada`, ejecutamos el comando `docker build`.

```bash
docker build -t mi-imagen-nginx .
```
- `-t mi-imagen-nginx`: `-t` es por "tag" (etiqueta). Le estamos dando a nuestra imagen un nombre legible: `mi-imagen-nginx`.
- `.`: Este punto final es muy importante. Le dice a Docker que el contexto de build (los archivos que puede usar, como `index.html`) es el directorio actual.

Verás una salida en la terminal que muestra a Docker ejecutando cada paso del `Dockerfile`.

### 2. Verificar la Creación de la Imagen
Una vez finalizado el build, puedes listar tus imágenes locales para ver la que acabas de crear.
```bash
docker images
```
Deberías ver `mi-imagen-nginx` en la lista.

### 3. Ejecutar un Contenedor desde la Nueva Imagen
Ahora podemos iniciar un contenedor a partir de nuestra imagen personalizada. Nota que ya no necesitamos usar volúmenes, porque el archivo `index.html` ya está "dentro" de la imagen.

```bash
docker run -d -p 8080:80 --name mi-app-personalizada mi-imagen-nginx
```

### 4. Verificar el Resultado
Visita `http://localhost:8080` en tu navegador. Verás la página con el título "Esta es una imagen personalizada". ¡Lo lograste! Acabas de servir una página desde una imagen que tú mismo construiste.

## Limpieza
Para eliminar los recursos creados en esta práctica:

1.  **Detén y elimina el contenedor:**
    ```bash
    docker stop mi-app-personalizada
    docker rm mi-app-personalizada
    ```
2.  **Elimina la imagen:**
    ```bash
    docker rmi mi-imagen-nginx
    ```