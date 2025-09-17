# Índice de Prácticas de Docker

Bienvenido a las prácticas de Docker. Cada directorio contiene una lección autocontenida con un archivo `README.md` que explica en detalle los objetivos, comandos y conceptos clave.

Se recomienda seguir las prácticas en orden para una curva de aprendizaje progresiva.

| # | Práctica | Objetivo Principal | Conceptos Clave | Directorio |
|:-:|:---|:---|:---|:---|
| 1 | **Hola Mundo** | Ejecutar tu primer contenedor y verificar la instalación de Docker. | `docker run`, ciclo de vida del contenedor. | [1-Hola-Mundo](./1-Hola-Mundo/) |
| 2 | **Servidor Web** | Exponer un puerto de un contenedor a tu máquina local. | `docker run -d -p`, modo desacoplado, mapeo de puertos. | [2-Servidor-Web](./2-Servidor-Web/) |
| 3 | **Modificar Contenedor** | Acceder a un contenedor en ejecución para hacer cambios. | `docker exec -it`, sistema de archivos efímero. | [3-Modificar-Contenedor](./3-Modificar-Contenedor/) |
| 4 | **Datos Persistentes** | Sincronizar una carpeta local con el contenedor para persistir datos. | `volúmenes` (`-v`), bind mounts. | [4-Datos-Persistentes](./4-Datos-Persistentes/) |
| 5 | **Imagen Personalizada** | Construir tu propia imagen a partir de un `Dockerfile`. | `docker build`, `Dockerfile` (FROM, COPY, CMD). | [5-Imagen-Personalizada](./5-Imagen-Personalizada/) |
| 6 | **WordPress con Compose** | Orquestar una aplicación de 2 contenedores (app + BD). | `docker-compose`, servicios, redes internas. | [6-WordPress](./6-WordPress/) |
| 7 | **WordPress Persistente** | Usar volúmenes en Compose para guardar los datos de forma segura. | Volúmenes nombrados, `docker-compose down -v`. | [7-WordPress-Persistente](./7-WordPress-Persistente/) |
| 8 | **Builds Multi-Etapa** | Optimizar imágenes para producción, haciéndolas más pequeñas y seguras. | `Multi-stage builds`, `COPY --from`. | [8-MultiStage](./8-MultiStage/) |