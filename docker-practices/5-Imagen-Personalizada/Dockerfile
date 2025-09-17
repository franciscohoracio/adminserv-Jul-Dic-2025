FROM nginx:stable-alpine

# Copia el archivo index.html personalizado al directorio web de Nginx
COPY index.html /usr/share/nginx/html

# Expone el puerto 80 para permitir el tráfico HTTP
EXPOSE 80

# Comando para iniciar Nginx en primer plano
CMD ["nginx", "-g", "daemon off;"]
