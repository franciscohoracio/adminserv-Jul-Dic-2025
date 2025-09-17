# Práctica 4: Persistencia de Datos con Volúmenes (Bind Mounts)

## Objetivo
Resolver el problema de la pérdida de datos que vimos en la práctica anterior. Aprenderemos a usar **volúmenes** para persistir datos. Específicamente, usaremos un **bind mount**, que sincroniza un archivo o carpeta de nuestra máquina (el host) con un archivo o carpeta dentro del contenedor.

## Conceptos Clave
- **Volúmenes:** Es el mecanismo preferido para persistir datos generados y usados por contenedores Docker. Los volúmenes son gestionados por Docker y desacoplan los datos del ciclo de vida del contenedor.
- **Bind Mounts:** Un tipo de volumen que mapea un directorio o archivo del sistema de archivos del host al sistema de archivos del contenedor. Los cambios son bidireccionales e instantáneos.
- **Flag `-v` o `--volume`:** Se usa para crear el bind mount. La sintaxis es `-v /ruta/absoluta/en/host:/ruta/en/contenedor`.

## Archivos de la Práctica
En esta carpeta, tenemos un archivo `index.html` con el siguiente contenido:
```html
<!DOCTYPE html>
<html>
<head>
<title>Datos Persistentes</title>
</head>
<body>
<h1>Esta página es persistente</h1>
</body>
</html>
```
Vamos a "montar" este archivo dentro de un contenedor Nginx para que sirva esta página en lugar de la de por defecto.

## Instrucciones

### 1. Iniciar el Contenedor con un Bind Mount
Ejecuta el siguiente comando. Este le dice a Docker que inicie un contenedor Nginx y que mapee nuestro `index.html` local al `index.html` que Nginx usa por defecto.

```bash
# Asegúrate de estar en la carpeta 4-Datos-Persistentes
docker run -d -p 8080:80 \
  -v $(pwd)/index.html:/usr/share/nginx/html/index.html \
  --name mi-servidor-persistente nginx
```
- `$(pwd)`: Es un comando de la terminal que se expande a la ruta del directorio actual (del host). Esto hace que el comando sea portable.
- `-v $(pwd)/index.html:/usr/share/nginx/html/index.html`: Mapea el archivo `index.html` del directorio actual del host al archivo `/usr/share/nginx/html/index.html` dentro del contenedor.

### 2. Verificar el Resultado
Visita `http://localhost:8080` en tu navegador. Verás la página con el título "Esta página es persistente".

### 3. Probar la Persistencia y Sincronización en Vivo
1.  **Modifica el archivo local:** Abre el archivo `index.html` en tu editor de código (en tu máquina host) y cambia el texto. Por ejemplo:
    ```html
    <h1>¡Los cambios en el host se reflejan al instante!</h1>
    ```
2.  **Guarda el archivo** y **refresca la página** en tu navegador (`http://localhost:8080`).

Verás que el cambio aparece **inmediatamente** en el navegador, sin necesidad de reiniciar ni tocar el contenedor. Esto se debe a que el contenedor está leyendo el archivo directamente desde tu sistema de archivos.

### 4. Demostrar la Persistencia entre Contenedores
Si eliminas el contenedor y creas uno nuevo con el mismo comando, la página seguirá mostrando tu `index.html` local, demostrando que los datos ya no dependen del ciclo de vida del contenedor.

```bash
# Detener y eliminar el contenedor
docker stop mi-servidor-persistente
docker rm mi-servidor-persistente

# Volver a crearlo
docker run -d -p 8080:80 \
  -v $(pwd)/index.html:/usr/share/nginx/html/index.html \
  --name mi-servidor-persistente nginx
```
La página en `http://localhost:8080` seguirá mostrando tu contenido personalizado.

## Limpieza
No olvides detener y eliminar el contenedor al finalizar.
```bash
docker stop mi-servidor-persistente
docker rm mi-servidor-persistente
```