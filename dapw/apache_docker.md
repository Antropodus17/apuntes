# APUNTES PARA CREAR UN SERVIDOR WEB APACHE Y CONFIGURARLO

Tabla de contenidos
- [APUNTES PARA CREAR UN SERVIDOR WEB APACHE Y CONFIGURARLO](#apuntes-para-crear-un-servidor-web-apache-y-configurarlo)
  - [CREAR SERVIDOR](#crear-servidor)
    - [DOCKERFILE](#dockerfile)
    - [DOCKER COMPOSE](#docker-compose)
  - [CONFIGURACION DEL SERVIDOR](#configuracion-del-servidor)
    - [VIRTUAL HOST](#virtual-host)


## CREAR SERVIDOR

El primer paso va a ser crear la imágen del servidor de apache, para ello crearemos un Dockerfile.

### DOCKERFILE
---
Indicaremos la imagen de la que partiremos, en este caso **debian:12**  
`FROM debian:12`  
El siguiente paso es actualizar los repositorios del apt e instalaremos el servidor web **apache2** y el lenguaje **PHP** acordandose de concatenarlos con el operador && o con dos ***RUN*** y confirmando la instalacion con el parametro ***-y***
1. `RUN apt update && apt install -y apache2 php`
2. `RUN apt update` y `RUN apt install -y apache2 php`

Una vez instalado todo, hay que indicar el comando para que el servidor se levante y el docker se mantenga.  
`ENTRYPOINT ["apachectl","-D","FOREGROUND"]`

El resultado final debería quedar algo así:

```Dockerfile
FROM debian:12

RUN apt update && apt install -y apache2 php

ENTRYPOINT [ "apachectl","-D","FOREGROUND" ]
```

### DOCKER COMPOSE
Lo siguiente es configurar el docker-compose.yml  

Dentro del compose.yml vamos a declarar el servicio web donde indicaremos la imágen del servidor recien creada,  en mi caso esta en el directorio `docker/web`.  

```yml
services:
   web:
      build: docker/web
```

Lo siguiente es declarar los puertos por donde el servidor escuchara las peticiones, en este caso escucharemos en nuestro puerto **8000** y lo pasaremos como puerto **80** de la máquina virtual.
```yml
services:
   web:
      build: docker/web
      ports:
        - 8000:80
```
Por último queda colocar los volúmenes donde estaran las páginas web y los archivos de configuracion para los virtual hosts. 

Estos últimos deben ir en el directorio correspondiente del servidor, en nuestro caso esa ruta es: `/etc/apache2/sites-enabled/`.

Puedes añadir los elementos de configuracion uno por uno o la carpeta contenedora.  
En el  siguiente caso los archivos de configuracion estan en el directorio `configs` y las páginas web en `pages`, estas las mapeamos a `/web`, aunque podría estar en otro directorio, no es obligatorio.
```yml
services:
   web:
      build: docker/web
      ports:
         - 8000:80
      volumes:
         - ./configs:/etc/apache2/sites-enabled
         - ./pages:/web
```

## CONFIGURACION DEL SERVIDOR

Ahora explicaremos la configuración de los server hosts.

Lo primero, los archivos de configuracion han de tener extension `.conf` para que el servidor los acepte, es a su vez una manera de desactivar una configuración, cambiandole la extension.

### VIRTUAL HOST
---
Lo principal a configurar es el virtual host, empezaremos con las directivas más básicas
- **<VirtualHost \*:80>** Etiqueta del ámbito, al igual que en los XML tiene etiqueta de cierre.
  - **VirtualHost**: Ámbito para definir las directivas.
  - **\***: Indica la ip por la que entra al virtual host, el asterisco indica todas.
  - **:80**: Indica el puerto por el que escucha el virtual host para la ip anterior.
    - **ServerName**: Directiva para indicar el nombre de dominio por el que se entra al virtual host.
    - **ServerAdmin**: Directiva para indicar el email del admin que se ofrecera en caso de errores para contactarlo.
    - **ServerAlias**: Directiva para indicar variantes del nombre de dominio, normalmente para añadir el subdominio **www.**
    - **DocumentRoot**: Directiva para indicar el directorio raiz para el virtual host.
    - **DirectoryIndex**: Directiva para indicar el orden de acceso por defecto si en la url no se especifica el fichero, en este caso primero va el `sample.php` y por último `z.html`
- **<Directory /web/server1>**: Etiqueta del ámbito, al igual que en los XML tiene etiqueta de cierre.
  - **Directory**: Ámbito para definir las directivas, al estar dentro de otro ámbito solo funciona cuando se esta en el ámbito superior.
  - **/web/server1**: Directorio al cuál aplicar las directivas.
    - **Require**: Directiva para indicar los permisos de acceso.
      - **all**: Aplica la directiva a todas las conexiones.
      - **granted**: Permite el acceso incondicional a los archivos.

```Apache
<VirtualHost *:80>
    ServerName proba.lan
    ServerAdmin a23sergiopn@iessanclemente.net
    ServerAlias www.proba.lan
    DocumentRoot /web/server1
    DirectoryIndex sample.php index.html a.html z.html 

    <Directory /web/server1>
        Require all granted
    </Directory>

</VirtualHost>
```