# Práctica 1: Hola Mundo en Docker

## Objetivo
El objetivo de esta práctica es familiarizarse con el comando más básico de Docker, `docker run`. A través de la ejecución de un contenedor simple, verificaremos que la instalación de Docker es correcta y entenderemos el ciclo de vida básico de un contenedor.

## Conceptos Clave
- **Imagen de Docker:** Una plantilla de solo lectura con instrucciones para crear un contenedor. En este caso, usaremos la imagen oficial `hello-world`.
- **Contenedor de Docker:** Una instancia en ejecución de una imagen. Es un entorno aislado y ligero.
- **Docker Hub:** Un registro en la nube donde se almacenan y distribuyen imágenes de Docker, similar a GitHub para el código.

## Instrucciones
1.  Abre tu terminal.
2.  Ejecuta el siguiente comando para descargar y correr la imagen `hello-world`:

    ```bash
    docker run hello-world
    ```

## Explicación del Comando
Cuando ejecutas `docker run hello-world`, Docker realiza los siguientes pasos:
1.  **Busca la imagen localmente:** El demonio de Docker primero verifica si la imagen `hello-world` ya existe en tu máquina.
2.  **Descarga la imagen:** Si no la encuentra, se conecta a Docker Hub y descarga la última versión de la imagen (`hello-world:latest`).
3.  **Crea un nuevo contenedor:** A partir de esa imagen, crea una nueva instancia de contenedor.
4.  **Ejecuta el programa:** Dentro del contenedor, ejecuta el programa principal que está configurado en la imagen. Para `hello-world`, este programa imprime un mensaje informativo.
5.  **Muestra el resultado:** El resultado de la ejecución se muestra en tu terminal. Una vez que el programa finaliza, el contenedor se detiene.

## Resultado Esperado
Deberías ver un mensaje en tu terminal que comienza así:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

... (y más texto explicativo)
```

## Pasos Adicionales (¡Importante!)
El contenedor ya se detuvo, pero todavía existe.
1.  Para ver todos los contenedores, incluyendo los que se han detenido, usa el comando `docker ps -a`:

    ```bash
    docker ps -a
    ```
    Verás una entrada para el contenedor `hello-world` con el estado `Exited`.

2.  Para mantener tu sistema limpio, es una buena práctica eliminar los contenedores que ya no necesitas. Puedes eliminarlo usando su `CONTAINER ID` o su nombre:

    ```bash
    # Reemplaza <CONTAINER_ID> con el ID que viste en el paso anterior
    docker rm <CONTAINER_ID>
    ```

¡Felicidades, has ejecutado tu primer contenedor en Docker!