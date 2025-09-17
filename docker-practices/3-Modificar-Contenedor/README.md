# Práctica 3: Modificar un Contenedor en Ejecución

## Objetivo
El objetivo es aprender a interactuar con un contenedor que ya está en ejecución. Usaremos el comando `docker exec` para obtener una terminal dentro del contenedor y modificar su contenido en vivo. También demostraremos que estos cambios son efímeros: se pierden si el contenedor se elimina y se vuelve a crear desde la imagen original.

## Conceptos Clave
- **`docker exec`:** Este comando nos permite ejecutar un proceso dentro de un contenedor que ya está corriendo. Es la herramienta principal para "entrar" a un contenedor y diagnosticar problemas o hacer cambios rápidos.
- **Flags `-i` y `-t`:** Usadas conjuntamente (`-it`), nos proporcionan una sesión de terminal interactiva (un shell) dentro del contenedor.
- **Sistema de Archivos Efímero:** Los cambios que haces en el sistema de archivos de un contenedor no persisten si el contenedor es eliminado. Los contenedores son, por diseño, descartables.

## Instrucciones

### 1. Iniciar el Contenedor
Primero, inicia un contenedor Nginx como en la práctica anterior.
```bash
docker run -d -p 8080:80 --name mi-servidor-web-modificable nginx
```

### 2. Acceder al Contenedor
Ahora, vamos a obtener un shell (una terminal) dentro de nuestro contenedor Nginx.
```bash
docker exec -it mi-servidor-web-modificable bash
```
- `exec`: Ejecuta un comando en el contenedor.
- `-it`: Nos da un shell interactivo.
- `mi-servidor-web-modificable`: El nombre de nuestro contenedor.
- `bash`: El comando que queremos ejecutar (el shell Bash).

Tu prompt de la terminal debería cambiar, indicando que ahora estás dentro del contenedor (ej: `root@<id_contenedor>:/#`).

### 3. Modificar la Página de Bienvenida
El contenido web de Nginx se sirve desde `/usr/share/nginx/html`. Vamos a reemplazar el contenido del archivo `index.html`.

```bash
# Dentro del contenedor
echo '<h1>¡Hola Mundo desde mi contenedor modificado!</h1>' > /usr/share/nginx/html/index.html
```

### 4. Verificar el Cambio
- **Sal del contenedor:** Escribe `exit` en la terminal del contenedor y presiona Enter.
- **Visita tu navegador:** Refresca la página en `http://localhost:8080`. Ahora deberías ver tu mensaje personalizado: "¡Hola Mundo desde mi contenedor modificado!".

### 5. Demostrar la Naturaleza Efímera
Ahora, veamos qué pasa si eliminamos el contenedor y creamos uno nuevo.

```bash
# Detener y eliminar el contenedor que modificamos
docker stop mi-servidor-web-modificable
docker rm mi-servidor-web-modificable

# Crear un contenedor nuevo con el mismo nombre y desde la misma imagen original
docker run -d -p 8080:80 --name mi-servidor-web-modificable nginx
```

Si ahora refrescas `http://localhost:8080` en tu navegador, verás que tu mensaje personalizado **ha desaparecido**. Ha vuelto la página original de "Welcome to nginx!".

## Conclusión
Los cambios realizados con `docker exec` son temporales y solo afectan al contenedor en cuestión. Para hacer cambios permanentes, se deben usar otras estrategias como los **volúmenes** (ver práctica siguiente) o la creación de **imágenes personalizadas**.

## Limpieza
No olvides detener y eliminar el último contenedor que creaste.
```bash
docker stop mi-servidor-web-modificable
docker rm mi-servidor-web-modificable
```