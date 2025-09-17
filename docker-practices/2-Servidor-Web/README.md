# Práctica 2: Servidor Web con Nginx

## Objetivo
En esta práctica, aprenderemos a ejecutar un contenedor en segundo plano (detached mode) y a mapear puertos entre el host y el contenedor. Esto nos permitirá acceder a una aplicación web que se ejecuta dentro de un contenedor desde nuestro navegador.

## Conceptos Clave
- **Modo Desacoplado (Detached):** Usando la bandera `-d`, el contenedor se ejecuta en segundo plano, liberando la terminal.
- **Mapeo de Puertos:** La bandera `-p` nos permite redirigir el tráfico de un puerto en la máquina host a un puerto dentro del contenedor. La sintaxis es `-p <puerto_host>:<puerto_contenedor>`.
- **Nombres de Contenedores:** Es una buena práctica asignar nombres a nuestros contenedores con `--name` para poder gestionarlos fácilmente.
- **Imagen `nginx`:** Una imagen oficial muy popular que contiene el servidor web Nginx.

## Instrucciones
1.  Ejecuta el siguiente comando para iniciar un servidor Nginx. Mapearemos el puerto `8080` de nuestra máquina al puerto `80` del contenedor (que es el puerto por defecto de Nginx).

    ```bash
    docker run -d -p 8080:80 --name mi-servidor-web nginx
    ```

2.  Verifica que el contenedor está en ejecución con `docker ps`.

    ```bash
    docker ps
    ```
    Deberías ver una salida que muestra el contenedor `mi-servidor-web` con el estado `Up` y el mapeo de puertos `0.0.0.0:8080->80/tcp`.

3.  Abre tu navegador web y visita `http://localhost:8080`.

## Resultado Esperado
Al visitar la URL en tu navegador, verás la página de bienvenida de Nginx: "Welcome to nginx!".

![Página de bienvenida de Nginx](https://raw.githubusercontent.com/docker-library/docs/master/nginx/logo.png)

Esto confirma que tu contenedor está funcionando correctamente y que el mapeo de puertos fue exitoso.

## Limpieza
Para detener y eliminar el contenedor, puedes usar los siguientes comandos:

1.  **Detener el contenedor:**

    ```bash
    docker stop mi-servidor-web
    ```

2.  **Eliminar el contenedor:**

    ```bash
    docker rm mi-servidor-web
    ```