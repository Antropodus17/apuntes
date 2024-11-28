# APUNTES DOCKER

Tabla de contenidos
- [APUNTES DOCKER](#apuntes-docker)
  - [COMANDOS BASICOS](#comandos-basicos)
    - [DOCKER](#docker)
    - [DOCKER CONTAINER](#docker-container)
    - [DOCKER COMPOSE](#docker-compose)
  - [FICHEROS PARA LEVANTAR SERVICIOS](#ficheros-para-levantar-servicios)
    - [DOCKER-COMPOSE.YML](#docker-composeyml)
      - [SERVICES](#services)
      - [VOLUMES](#volumes)
      - [SECRETS](#secrets)
    - [DOCKERFILE](#dockerfile)
    - [.ENV](#env)

## COMANDOS BASICOS 

Apuntes para los comandos básicos en terminal de docker.

### DOCKER
---
Comandos básicos de docker
- `docker build 'Dockerfile'` Crea una imagen para usar a partir de un Dockerfile.
  
- `docker create [--name][-p] 'image'` Crea un contenedor a partir de una imagen.
  - `--name dockerName` Asigna un nombre al docker para su posterior manejo.
- `docker start 'dockerName'` Activa un el docker inactivo.
- `docker ps [-a]` Muestra la información del estado de los dockers levantados
  - `-a` Muestra a mayores la información de los dockers inactivos.
- `docker stop 'dockerName'` Para el estado del docker.
- `docker kill 'dockerName'` Para forzadamente el docker y lo vuelve inactivo.
- `docker rm 'dockerName'` Elimina el docker inactivo.
- `docker rename 'actualName' 'newName'` Modifica el nombre actual del docker al nuevo.

### DOCKER CONTAINER
---
Comando útil de container
- `docker container prune` Elimina todos los dockers inactivos.

### DOCKER COMPOSE
Estos comando han de ejecutarse sobre un archivo yml [docker-compose or compose](docker-compose.md#apuntes-docker-composeyml)

Comandos básicos:
- `docker compose build` Crea la imagen especificada en el yml.

- `docker compose up [serviceName]` Levanta el docker(También crea una network para los servicios).
  - `serviceName` Indica el servicio especificado en el compose.
- `docker compose run [serviceName] 'comando'` Levanta servicio y ejecuta el comando introducido.
- `docker compose down [serviceName]` Tira los servicios y los retira.


## FICHEROS PARA LEVANTAR SERVICIOS

Apuntes para los ficheros que utilizaremos con dockers.

### DOCKER-COMPOSE.YML
---

En el fichero `docker-compose.yml` utilizaremos 'etiquetas' de declaracion `services`, `volumes`, `secrets`.

IMPORTANTE: Las 'eitquetas' dentro de otras han de estar identadas como en python.

[Imágen de muestra del docker compose](./img/docker-composer-ejemplo.png)

#### SERVICES

Dentro de `services:` definimos los servicios que se van a levantar.

Cada servicio tendrá otras etiquetas para definir su comportamiento y creación.

- `image` Indica que imagen utilizar.
- `build` Indica el fichero del que crear la imágen.
  - `context` Indica la ruta donde buscar el Dockerfile.
  - `dockerfile` Indica el nombre del Dockerfile. Se pueden tener varios para cada imágen nombrandolos distintos y con la extensión .Dockerfile.
- `hostname` Indica el nombre del host para facilitar la comunicacion entre servicios de la misma network.
- `ports` Mapea los puertos de la máquina real con el docker e.g **'puertos_Maquina_Real:puertos_Maquina_Virtual'**.
- `volumes` Conecta las rutas del docker a la máquina real. Estos pueden ser una carpeta que sustituye la otra, un fichero que se añade a la carpeta del docker, o un volúmen creado con la 'etiqueta' raíz `volumes` no posible de ubicar, útil para las bases de datos.
- `secrets` Indica los secretos a cargar en el servicio. Estos han de estar creados en la 'etiqueta' raíz `secrets`.
- `enviroment` Lista las variables y su valor a cargar en  el servicio.
- `env_file` Indica la ruta del fichero .env desde el cúal cargar la variables de entorno.
  
#### VOLUMES

Define el nombre de los volúmenes para utilizar.

#### SECRETS

Declara el nombre del secret y su fichero con el contenido.

- `file` Especifica la ruta del fichero.

### DOCKERFILE
---

Fichero para indicar los pasos a la hora de crear una imagen.

[Imágen de un Dockerfile de ejemplo](./img/dockerfile-ejemplo.png)

- `FROM` Indica sobre que imágen empezar a configurar.
- `RUN` Ejecuta el comando a ejecutar. E.g apt update && apt install **-y** apache2.
- `WORKDIR` Cambia la ruta sobre la que se ejcutan los comandos y la ruta de entrada del docker. E.g como un cd.
- `EXPOSE` Abre el puerto para utilizar en el servicio.
- `CMD` Indica el comando a ejecutar al levantar el docker. Se ejecuta el último definido.
- `ENTRYPOINT` Como el `CMD` pero con más prioridad, si se combinan el `CMD` se convierte en argumentos para el `ENTRYPOINT`.

### .ENV
---

Es un fichero donde se almacenan todas las variables y sus valores a cargar en los servicios.

[Imágen de un .env de ejemplo](./img/env-ejemplo.png)

El formato es `NOMBRE_VARIABLE_ENTORNO='valor de la variable'`
Si la imágen lo permite se pueden pasar el valor como un `secret` [declarado en el docker-compose](#secrets).  
En estos casos, normalmente se le añade '_FILE' a la variable y su valor es `/run/secrets/nombre_del_secret`. Como se puede ver en la imágen.