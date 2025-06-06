
# Despliegue de WordPress y Base de Datos con Docker

Autor: Alejandro Toloza  
Versión: 3.1  
Sistema operativo: Linux Lite 7.2   
Referencia: [Tutorial de El Pelado Nerd](https://www.youtube.com/watch?v=eoFxMaeB9H4&list=PLqRCtm0kbeHAep1hc7yW-EZQoAJqSTgD-&index=5)

## 📸 Capturas del proceso

> (Incluir aquí imágenes relevantes. Inserta los archivos en el mismo directorio del README si es posible)

![Instalación de Docker](images/instalacion-docker.png)
![Error distutils](images/error-distutils.png)
![Contenedor WordPress funcionando](images/wordpress-up.png)
![Configuración Nginx](images/nginx-config.png)

## 📦 Docker Compose básico

```yaml
version: '3.1'
services:
  wordpress:
    image: wordpress:php7.1-apache
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: root
      WORDPRESS_DB_NAME: wordpress
    links:
      - mysql:mysql
  mysql:
    image: mysql:8.0.13
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - ~/docker/mysql-data:/var/lib/mysql
```
```yaml
version: '3.8'
services:
  wordpress:
    image: wordpress:php8.4-fpm
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: root
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress_data:/var/www/html  # Persistir los archivos de WordPress     config
    depends_on:
      - mysql
  mysql:
    image: mysql:8.0.42-debian
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - mysql_data:/var/lib/mysql
  nginx:
    image: nginx:latest
    ports:
      - 8080:80  # Exponer el puerto 8080 para acceder a WordPress
    volumes:
      - wordpress_data:/var/www/html  # Compartir archivos de WordPress
      - ./nginx.conf:/etc/nginx/conf.d/default.conf  # Montar archivo configuración Nginx
    depends_on:
      - wordpress
volumes:
  wordpress_data:
  mysql_data:  # Volumen compartido para los archivos de WordPress
```

...

## 🔗 Referencias

- [Instalación de Docker en Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Instalación de Docker Compose](https://docs.docker.com/compose/install/linux/)
