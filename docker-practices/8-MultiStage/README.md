# Práctica 8: Builds Multi-Etapa para Optimización

## Objetivo
Aprender una de las técnicas más importantes para crear imágenes de producción: los **builds multi-etapa (multi-stage builds)**. El objetivo es generar una imagen final que sea lo más pequeña y segura posible, conteniendo únicamente lo necesario para ejecutar la aplicación, sin incluir herramientas de compilación, dependencias de desarrollo o el código fuente.

## El Problema: Imágenes de Desarrollo vs. Producción
Para construir una aplicación (como en este caso, una de React), necesitamos muchas herramientas: Node.js, `npm`, todas las `devDependencies` de `package.json`, etc. Esto genera una imagen muy pesada.

Sin embargo, para **ejecutar** la aplicación de React una vez construida, solo necesitamos un servidor web estático (como Nginx) y los archivos HTML, CSS y JS resultantes. Incluir Node.js y todo lo demás en la imagen de producción es ineficiente y aumenta la superficie de posibles ataques.

## La Solución: Builds Multi-Etapa
Un build multi-etapa nos permite usar múltiples `FROM` en un solo `Dockerfile`. Cada `FROM` inicia una nueva "etapa". Podemos copiar selectivamente archivos (artefactos) de una etapa a otra.

## Archivos de la Práctica
- **`src/` y `package.json`**: El código fuente y las dependencias de una aplicación `create-react-app`.
- **`Dockerfile`**: El archivo que define nuestro build de dos etapas.

### Análisis del `Dockerfile`
```dockerfile
# --- Etapa 1: Build (La llamamos 'builder') ---
# Usamos una imagen completa de Node para tener npm y todas las herramientas
FROM node:18-alpine AS builder

WORKDIR /app

# Copiamos solo package.json para aprovechar el cache de Docker
COPY package*.json ./

# Instalamos TODAS las dependencias (de desarrollo y producción)
RUN npm install

# Copiamos el resto del código fuente
COPY . .

# Ejecutamos el script que crea la build de producción en la carpeta /app/build
RUN npm run build

# --- Etapa 2: Production (La final) ---
# Empezamos desde cero con una imagen súper ligera de Nginx
FROM nginx:stable-alpine

# La magia: Copiamos SOLO la carpeta /app/build de la etapa 'builder' 
# al directorio web de Nginx en nuestra nueva imagen final.
COPY --from=builder /app/build /usr/share/nginx/html

# Exponemos el puerto y definimos el comando para arrancar Nginx
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Instrucciones

### 1. Construir la Imagen Multi-Etapa
Desde la carpeta `8-MultiStage`, ejecuta el comando de build.
```bash
docker build -t mi-react-app .
```
Docker ejecutará ambas etapas. La etapa `builder` es temporal y se descarta al final, dejando solo la imagen final creada a partir de la segunda etapa.

### 2. ¡Verificar el Ahorro de Espacio!
Compara el tamaño de tu nueva imagen con la imagen de Node que usamos para construirla.
```bash
docker images
```
Busca `mi-react-app` y `node`. Notarás que `mi-react-app` (basada en `nginx:stable-alpine`) es significativamente más pequeña que la imagen `node:18-alpine`. ¡Hemos logrado nuestro objetivo!

### 3. Ejecutar la Aplicación Optimizada
Inicia un contenedor a partir de tu nueva imagen ligera.
```bash
docker run -d -p 8080:80 --name mi-app-final mi-react-app
```

### 4. Verificar el Resultado
Visita `http://localhost:8080`. Deberías ver la aplicación de React funcionando perfectamente, servida desde una imagen Nginx mínima.

## Limpieza

1.  **Detén y elimina el contenedor:**
    ```bash
    docker stop mi-app-final
    docker rm mi-app-final
    ```
2.  **Elimina la imagen:**
    ```bash
    docker rmi mi-react-app
    ```