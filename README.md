
# Despliegue de WordPress y Base de Datos con Docker

#### Autor: Alejandro Toloza  
#### Versión: 3.1  
#### Sistema operativo: Linux Lite 7.2. 
#### Entorno: Notebook Lenovo (Intel dual core 2020 2.4Ghz  4gb)
#### Referencia: [Tutorial de El Pelado Nerd](https://www.youtube.com/watch?v=eoFxMaeB9H4&list=PLqRCtm0kbeHAep1hc7yW-EZQoAJqSTgD-&index=5)

## Docker Compose básico

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

> 

![image](https://github.com/user-attachments/assets/a4fd89b2-9c44-4391-8eaf-4cf56c43f8d2)
![image](https://github.com/user-attachments/assets/1d8b9015-f621-4e53-9773-48ef042b777e)


### Mas allá de correr sin problemas en Linux Lite quise probarlo en Ubuntu 24.04
#### Sistema Operativo: Ubuntu 24.04.
#### Entorno: CPU I5-2500K 3.3GHZ 16 GB DDR3

### Configurar el repositorio apt de Docker. Agregar la clave GPG oficial de Docker:

```
  ~$ sudo apt-get update
	~$ sudo apt-get install ca-certificates curl
	~$ sudo install -m 0755 -d /etc/apt/keyrings
	~$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o 	/etc/apt/keyrings/docker.asc
	~$ sudo chmod a+r /etc/apt/keyrings/docker.asc
```
### Agregar el repositorio a las fuentes de Apt: 

```
  ~$ echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
   $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") 	stable" | \	~$ sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
	~$ sudo apt-get update

```
### Instalar los paquetes Docker:

> 
![image](https://github.com/user-attachments/assets/dbaff981-73b0-4c5b-a059-985db80d9d70)

Error de GPG https://download.docker.com/linux/ubuntu noble InRelease: Las firmas siguientes no se pudieron verificar porque su clave pública no está disponible: NO_PUBKEY 7EA0A9C3F273FCD8. No se puede verificar la autenticidad del repositorio Docker.  Para cuando la configuración del repositorio continúa fallando, Docker proporciona un script conveniente que configura automáticamente el repositorio e instala Docker:

```
~$ curl -fsSL https://get.docker.com -o get-docker.sh
~$ sudo sh get-docker.sh
```
Añadí manualmente la llave GPG usando apt-key (Fallback) porque el método firmado no funcionaba, utilicé el método de apt-key más antiguo.
#### Importé la llave GPG directamente:
```
~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
#### Actualizé la configuración del repositorio firmada:
```
~$ echo "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu noble stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
#### Actualizé de nuevo:
```
~$ sudo apt update
```
#### Instalé Docker y el pluging de docker compose:
```
~$ sudo apt install -y docker-ce docker-ce-cli containerd.io docker-	buildx-plugin docker-compose-plugin
```
### Verificar que la instalación está corriendo con éxito con el run de la imagen hello-world.
```
~$ sudo docker run hello-world
```
>
![image](https://github.com/user-attachments/assets/c8df6afb-f776-4eb9-88c3-a22e2b93c27e)

Error:  Archivo "/usr/lib/python3/dist-packages/compose/cli/main.py", línea 9, en modul. de distutils.spawn import find.ejecutable 
El error ModuleNotFoundError: No module named 'distutils' occurre porque el modulo distutils el cual era parte de la librería standard de Python ha sido eliminado a partir de Python 3.12  Docker-compose está tratando de importar distutils.spawn.find_executable y no lo encuentra porque la versión de Ubuntu 24.04 ya trae Python 3.12 por defecto. Esto crea incompatibilidad con las viejas versiones de docker-compose.

#### Instalé paquete que incluye una versión de distutils y puede usarse como una capa de compatibilidad
```
~$ sudo apt install -y python3-setuptools 
```
#### Reinstalé el pluging de docker compose
```
~$ sudo apt-get install docker-compose-plugin
```
Luego de haber reinstalado el pluging de docker compose levanté el contenedor con el comando sobre el archivo docker-compose.yaml idéntico al que había corrido en LinuxLite
(Versión 3.1)
```
~$ sudo docker-compose up -d
```
>
![image](https://github.com/user-attachments/assets/20545f57-f5d4-45da-a753-291e9b5c9b58)

Tiraba error de conexión con la base de datos por eso quise actualizar las versiónes, desde https://hub.docker.com/ , de la imagen de mysql a 8.0.42-debian y también la de wordpress a  php8.4-fpm que en ese momento eran las últimas.
#### Modifiqué el archivo yaml
```
version: '3.1'
services:
  wordpress:
    image: wordpress:php8.4-fpm
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
    image: mysql:8.0.42-debian
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - ~/docker/mysql-data:/var/lib/mysql
```
Tiró un nuevo error
>
![image](https://github.com/user-attachments/assets/475be3a0-301c-4cda-b883-f4f78cc8ee33)
![image](https://github.com/user-attachments/assets/4713e331-e2d5-43c1-ad22-59c90e211e4d)

Para corregir el problema de KeyError: ContainerConfig:

-Actualicé la versión del compose en el archivo de yaml:
version: '3.1' a version: '3.8'
 
-Eliminé enlaces: La directiva de enlaces (mysql:mysql). no hace falta en la nueva versión de 	Docker Compose, ésta crea automáticamente una red predeterminada para sus servicios, y 	los servicios pueden comunicarse utilizando sus nombres de servicio.  
	links:
	  - mysql:mysql
	
-Para asegurar que wordpress comience después de mysql, agregué la línea:
config
    depends_on:
      - mysql

Al seguir con el problema de no poder acceder al localhost me encontré con el problema de que la imagen de  wordpress:php8.4-fpm es para PHP-FPM (FastCGI Process Manager) es una implementación avanzada de FastCGI diseñada específicamente para manejar solicitudes PHP de manera eficiente, especialmente en aplicaciones web como WordPress), lo que significa que PHP-FPM es el componente que procesa las solicitudes PHP del sitio WordPress. que típicamente requiere un servidor web separado para atender solicitudes HTTP. 

#### Así que tuve que añadir un servicio Nginx al docker-compose.yaml. 
```
nginx:
    image: nginx:latest
    ports:
      - 8080:80  # Exponer el puerto 8080 para acceder a WordPress
    volumes:
      - wordpress_data:/var/www/html  # Compartir archivos de WordPress
      - ./nginx.conf:/etc/nginx/conf.d/default.conf  # Montar archivo configuración Nginx
    depends_on:
      - wordpress
```
#### Finalmente pude acceder al localhost, instalar WordPress, personalizar la página y también eliminar los contenedores con la persistencia de los datos. Con el siguiente archivo yaml modificado:

## Docker Compose Ubuntu 24.04 
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
>
![image](https://github.com/user-attachments/assets/590967fd-ad6b-4aa9-a798-915f50c55f5b)
![image](https://github.com/user-attachments/assets/af2baca9-115e-44ae-9ae7-6041aa73169e)
![image](https://github.com/user-attachments/assets/51820186-70ea-4caf-a5d1-8cc0bb637c7a)

Luego a la hora de querer cargar un archivo de 355mb tiró el error que el tamaño del archivo excede el tamaño permitido del sitio. Esto excede el límite de tamaño de subida predeterminado establecido por WordPress, PHP o Nginx. Por defecto, WordPress y PHP imponen restricciones en los tamaños de carga de archivos, típicamente alrededor de 2 MB a 64 MB, dependiendo de la configuración.
>
![image](https://github.com/user-attachments/assets/33089d23-1a8f-4cd2-932e-76c177633a95)

Para actualizar la configuración de PHP tuve que crear un archivo nombrado uploads.ini:

```
upload_max_filesize = 500M  --(permite archivos de hasta 500mb)
post_max_size = 500M        --(permite solicitudes de hasta 500mb)
memory_limit = 512M         --(garantiza que PHP tenga suficiente memoria) 
max_execution_time = 300    --(tiempo garantizado para la ejecución de los scripts en segundos para que pueda subir el archivo sin problemas)
```
Luego tuve que agregar una línea al archivo docker-compose.yaml para poder montar ese archivo en el contenedor de WordPress:

## Docker Compose Ubuntu 24.04 completo con el montaje del archivo uploads.ini
```
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
      - wordpress_data:/var/www/html  # Persistir los archivos de WordPress
      - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini #Agregué/subir+355mb   config
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
Y finalmente pude subir el archivo pdf de 355Mb
>
![image](https://github.com/user-attachments/assets/d0248744-9f0c-4643-b338-5990a92d17b6)

# Uso de Docker Compose
 El uso de Docker Compose es recomendado para no pifiarle a todas las variables de entorno y comandos que hay que hacer al levantar un contenedor o varios con todos los servicios. 
	Ya con docker instalado en el sistema, si tuviese que levantar los contenedores y conectarlos entre sí por linea de comandos, en mi caso se vería algo así. 
##### 1°- Crear los directorios para que los datos persistan, usando la configuración bind mounts (./wordpress_data y ./mysql_data). Es una forma de montar un directorio del host en el sistema de archivos del contenedor. Cualquier cambio en los archivos dentro del contenedor se refleja en el host y viceversa.

```
 ~$ mkdir -p ~/proyecto_docker/wordpress_data ~/proyecto_docker/mysql_data
 ~$sudo chmod -R 755~/proyecto_docker/wordpress_data   ~/proyecto_docker/mysql_data
```
##### 2°- Crear un archivo nginx.conf , con toda la configuración del servidor Nginx para definir como manejar solicitudes HTTP, establecer reglas de seguridad, pasar solicitudes PHP a PHP-FPM (en mi caso, al contenedor wordpress en el puerto 9000. Éste es el puerto interno donde el contenedor de WordPress (usando PHP-FPM) escucha solicitudes PHP. Nginx se comunica con WordPress en este puerto dentro de la red Docker, pero este puerto no está expuesto al host.

```
  ~$ nano ~/proyecto_docker/nginx.conf
```
##### 3°- Crear una Red Docker. Los contenedores necesitan comunicarse entre sí (Nginx con WordPress, y WordPress con MySQL). Para eso, crear una red Docker personalizada:

```
 ~$ sudo docker network create wordpress_network
```
##### 4°- Correr el contenedor con la configuración correcta para MySQL con las variables de entorno, la imagen requerida, conectar el contenedor a la red, se especifica el directorio local para persistir los datos, etc.

```
~$ sudo docker run -d \ --name mysql \ --network wordpress_network \ -v ~/proyecto_docker/mysql_data:/var/lib/mysql \ -e MYSQL_ROOT_PASSWORD=root \ -e MYSQL_DATABASE=wordpress \--restart unless-stopped \ mysql:8.0.42-debian \ --default-authentication-plugin=mysql_native_password
```
##### 5°- Correr el contenedor de WordPress con la configuración del host de la base de datos, contraseñas, imágen requerida de WordPress, conecta el contenedor a la red para comunicarse con el contenedor MySQL, etc.  

```
  ~$ sudo docker run -d \ --name wordpress \ --network wordpress_network \ -v ~/proyecto_docker/wordpress_data:/var/www/html\-e WORDPRESS_DB_HOST=mysql \
  -e WORDPRESS_DB_USER=root \-e WORDPRESS_DB_PASSWORD=root \ -e WORDPRESS_DB_NAME=wordpress \--restart unless-stopped \  wordpress:php8.4-fpm
```
##### 6°- Iniciar el contenedor de Nginx donde se expone el puerto del host para acceder al servicio, la imagen usada, conecta el contenedor a la red para comunicarse con el contenedor de WordPress, etc

```
  ~$ sudo docker run -d \ --name nginx \--network wordpress_network \-v ~/ proyecto_docker/wordpress_data:/var/www/html \-v~/proyecto_docker/nginx.conf:     
   /etc/nginx/conf.d/default.conf \-p 8080:80 \ --restart unless-stopped \ nginx:latest
```
##### 7°- Para detener los contenedores 

```
 ~$ sudo docker stop nginx wordpress mysql
```
##### 8°- Para iniciar los contenedores. La diferencia con run es que start inicia un contenedor exitente que no está corriendo en cambio run crea el contenedor a partir de una imagen y lo inicia.

```
 ~$ sudo docker start mysql wordpress nginx
```
### Con Docker-Compose a partir del archivo docker-compose.yaml en donde está toda la configuración de cada uno de los contenedores simplemente sería un sólo comando para iniciarlos y otro para detenerlos y eliminarlos:

```
~$ sudo docker-compose up -d (-d para poder seguir usando la terminal)
~$ sudo docker-compose down (para detenerlos y eliminarlos)
```
...

## Referencias

- [Instalación de Docker en Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Instalación de Docker Compose](https://docs.docker.com/compose/install/linux/)
- https://grok.com/
