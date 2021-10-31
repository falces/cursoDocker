# Conceptos

## Image / Imagen

Una **imagen** en Docker es la representación estática  de una aplicación o servicio con su configuración y todas sus  dependencias. Las imágenes se utilizan para crear contenedores, y nunca  cambian.

Por ejemplo una imagen podría contener un sistema Ubuntu con un servidor Apache y una aplicación web.

Las imágenes pueden almacenarse localmente o remotamente en un repositorio conocido como **registro**, donde están disponibles por nombre y normalmente en diferentes versiones etiquetadas, por ejemplo `ubuntu:latest` o `mysql:5.7`. El más utilizado es [Docker Hub](https://hub.docker.com/), un repositorio en la nube para crear, probar, guardar y distribuir  imágenes. También proporciona a los usuarios un espacio para crear  repositorios privados, automatizar funciones de compilación, crear *webhooks* o compartir espacios de trabajo.

## Containers / Contenedores

Los **contenedores** ejecutan instancias de las imágenes. Al ejecutar una imagen se crea un contenedor.

Como las imágenes no cambian, las modificaciones realizadas durante la  ejecución de un contenedor no serán persistentes al detenerlo y volver a ejecutarlo. Pero es posible crear una nueva imagen, una nueva versión,  con los cambios realizados. Y si algo va mal podríamos volver de forma  sencilla a una versión anterior del contenedor.

## Volumes / Volúmenes

Los ficheros creados dentro de un contenedor no persisten entre ejecuciones. **Docker** [proporciona dos mecanismos](https://docs.docker.com/storage/) para que un contenedor almacene archivos en la máquina huésped y persistan después de detenerlo.

Los volúmenes son el mecanismo preferido para mantener la persistencia de datos. Es posible definir volúmenes en modo «*sólo lectura*». Y volúmenes que pueden compartirse por más de un contenedor, algunos en modo «*lectura/escritura*» y otros en modo «*sólo lectura*»

## Docker CLI

Herramienta Docker para terminal.

# Comandos: Docker CLI

## Crear imagen y ejecutar

Con el comando `run` levantamos un contenedor específico. Los contenedores están disponibles en *DockerHub*. Por ejemplo, podemos levantar el contenedor con la imagen *hello-world*:

```bash
$ docker run hello-world
```

O *busybox*:

```bash
$ docker run busybox
```

El comando `run` es lo mismo que ejecutar los comandos `create` y `start`. Para la imagen `hello-world`, el comando anterior es lo mismo que ejecutar:

```bash
$ docker create hello-world # nombre de la imagen
$ 0ccf62a4aaff2bb25b9d21e78d423251e17d710cf5ddaabc5858353b167431e9 # Nos devuelve el id del contenedor
# Con el id del contenedor levantamos la imagen:
$ docker start -a 0ccf62a4aaff2bb25b9d21e78d423251e17d710cf5ddaabc5858353b167431e9
```

El comando `create` crea una imagen del contenedor, mientras que `start` ejecuta el comando de inicio con el que se creó el contenedor. Con el atributo -a hacemos que se envíen las respuestas a nuestro terminal, si no lo usáramos no veríamos nada. Para únicamente ejecutar un contenedor previamente creado, usamos el comando `start`, y de esta forma no crearemos un nuevo contenedor. No podemos especificar un nuevo comando de inicio con el comando `start`.

Podemos hacer lo mismo especificando un nombre para el contenedor con el atributo `name`, lo que nos permite no usar los id de contenedor:

```bash
# Atributo: --name NombreDelContenedor
$ docker create --name HelloWorld hello-world
$ deb4d7ca1ecd350c455c35363f2cfff1b67743a57f5752aef5ce5db08b60a43d
$ docker start -a HelloWorld
```

Podemos especificar comandos a ejecutar dentro del contenedor en el momento de su ejecución:

```bash
$ docker run busybox ls
```

Nos mostrará el contenido del directorio raíz del contenedor. Esto es porque la imagen *busybox* tiene instalado el ejecutable `ls`. Si ejecutamos:

```bash
$ docker run hello-world ls
```

Tendremos un error, ya que la imagen hello-world no tiene el ejecutable `ls` instalado.

## Listar contenedores e imágenes

Listar contenedores en ejecución:

```bash
$ docker ps
```

Listar contenedores aunque no estén en ejecución:

```bash
$ docker ps -a
$ docker container ls -a
$ docker ps --all
```

Listar imágenes:

```bash
$ docker images
$ docker image ls
```

## Borrar contenedores e imágenes

Borrar contenedor detenido:

```bash
$ docker rm <container_id>
$ docker container rm <container_id>
```

Borrar contenedor en ejecución:

```bash
$ docker rm -f <container_id>
```

Borrar imagen sin instancias en ejecución:

```bash
$ docker rmi <image_id>
$ docker image rm <image_id>
```

Borrar imagen con instancias en ejecución:

```bash
$ docker rmi -f <image_id>
```

Borrar todas las imágenes:

```bash
$ docker rmi -f $(docker images -q)
```

Borrar  todo: contenedores parados, redes no utilizadas por ningún contenedor, imágenes colgadas, caché:

```bash
$ docker system prune
$ docker system prune -a
```

## Acceder a los logs

Acceder a los logs de un contenedor. Por ejemplo, si no hemos puesto el parámetro -a y no tenemos respuesta en nuestro terminal. Los logs se almacenan con cada ejecución del contenedor:

```bash
$ docker logs <container_id>
```

## Parar contenedores

Envía una SIGNTERM (señal de terminal) para que el proceso pare por sí solo:

```bash
$ docker stop <container_id>
```

Envía una SIGKILL (señal de matar) al proceso primario que esté ejecutando el contenedor:

```bash
$ docker kill <container_id>
```

Si tras el comando `stop` el contenedor no se detiene en 10 segundos, Docker cancela el `stop` y envía un `kill` al contenedor

## Múltiples comandos

Con `exec` lanzamos procesos sobre un contenedor en ejecución. Para enviar comandos a un contenedor en ejecución, con el atributo `it` abrimos la comunicación con el contenedor. Este atributo equivale a los parámetros `i` (cualquier cosa que escriba que vaya al canal de entrada del contenedor STDIN) y `t` (formatea texto, identación y muestra ayudas visuales).

Por ejemplo, enviamos el comando `sh` para ejecutar un terminal (también podría ser bash, zsh, Powershell...):

```bash
$ docker exec -it <container_id> sh
```

# Crear nuestras propias imágenes

## Dockerfile

Es un archivo de texto plano que contiene líneas con configuración: qué programas va a contener y qué va a hacer cuando se inicie su ejecución. Tiene este flujo:

1. Especificar una imagen base
2. Ejecutar comandos para instalar programas adicionales, dependencias, etc.
3. Especificar un comando para ejecutar al iniciar el contenedor

Ejemplo: vamos a crear nuestra propia imagen con Redis:

```dockerfile
# Con FROM decimos qué imagen vamos a usar como base:
FROM alpine

# Ejecutamos comandos mientras se prepara nuestra imagen
RUN apk add --update redis

# Qué instrucción se ejecuta cuando nuestra imagen se instancia
CMD ["redis-server"]
```

Una vez hecho esto, nos situamos en el directorio y creamos el contenedor:

```bash
$ docker build .
```

Descarga y crea el contenedor:

```bash
[+] Building 1.8s (6/6) FINISHED
 => [internal] load build definition from Dockerfile                                                 
 => => transferring dockerfile: 249B                                                                 
 => [internal] load .dockerignore                                                                     
 => => transferring context: 2B                                                                       
 => [internal] load metadata for docker.io/library/alpine:latest                                     
 => [1/2] FROM docker.io/library/alpine                                                               
 => [2/2] RUN apk add --update redis                                                                 
 => exporting to image                                                                               
 => => exporting layers                                                                               
 => => writing image sha256:38903694787e77ff8b67dc422c273d29be43a8ff643abbe6e2dd787c78ad6336         
```

¿Qué hemos hecho? El primer paso: hemos usado como base Alpine, una distribución Linux muy ligera (apenas 5 megas) y completa. En el segundo paso nos hemos descargado e instalado Redis y, por último, el tercer paso, ejecutamos el comando `redis-server` para iniciar el servidor Redis.

Y lo iniciamos:

```bash
$ docker run --name RedisApp 38903694787e77ff8b67dc422c273d29be43a8ff643abbe6e2dd787c78ad6336
```

Y ya tenemos nuestra propia imagen con Redis funcionando y con el nombre RedisApp.

### Imagen base

La primera línea de nuestro Dockerfile:

```dockerfile
FROM alpine
```

Instala una imagen base, un sistema operativo. Una imagen es como un ordenador sin sistema operativo, por lo tanto lo primero que hay que hacer es instalar el sistema operativo. Alpine es una distribución Linux que viene con una serie de programas preinstalados que nos ayudan a crear nuestra imagen.

En una imagen base podemos especificar, usando tags, diferentes versiones. En Docker Hub podemos consultar todas las etiquetas que una imagen admite. Dado que Alpine es una distribución muy completa y ligera, muchas imágenes tienen versiones con esta distribución. Por ejemplo, si quisiéramos instalar Node como base, esta imagen tiene una etiqueta para instalar una versión de Alpine con Node:

```dockerfile
FROM node:alpine
```

### Instalación de dependencias y comandos previos

La segunda línea de nuestro Dockerfile:

```dockerfile
RUN apk add --update redis
```

El comando `apk` (Alpine Package Keeper), es el gestor de paquetes de esta distribución Linux. En estas líneas (no tiene por qué ser una sola línea) irán instrucciones aptas para el sistema operativo elegido. Por ejemplo, podríamos tener:

```dockerfile
RUN apk add --update redis
RUN apk add --update gcc
```

Con lo que además de Redis habríamos instalado el GCC, el compilador de colecciones GNU.

### Copiar archivos

En ocasiones es posible que necesitemos instalar dependencias de nuestro proyecto. Por ejemplo, para una aplicación Node, usamos

```dockerfile
RUN npm install
```

Lo que leería nuestro archivo package.json para descargar las dependencias. Pero dado que este archivo no está dentro del contenedor temporal con el que se está creando la imagen, tendríamos un error. Lo que tenemos que hacer es copiar los archivos necesarios dentro del contenedor antes de instalar las dependencias. Primero especificamos un directorio de trabajo dentro del contenedor (si no lo especificamos intentaría copiar en el directorio raíz y tendríamos error) y luego copiamos los archivos.

Antes de instalar las dependencias, copiamos nuestros archivos al contenedor:

```dockerfile
COPY ./ ./
# También válido:
COPY . .
```

Pero, ¿qué pasa si modifico algún archivo de mi aplicación? No vería reflejado el cambio en el contenedor, ya que el comando `build` lee Dockerfile y es en este momento cuando se copian los archivos, por lo que una modificación posterior no se vería reflejada. Lo primero que nos viene a la cabeza es volver a ejecutar `build`, que nos funcionaría. En aplicaciones grandes la instalación de dependencias puede llevar mucho tiempo, por lo que no sería ágil que cada vez que modifique un archivo volviera a ejecutar `build` para reconstruir el contenedor. Esto lo podemos solucionar con una pequeña estrategia en la copia:

```dockerfile
COPY ./package.json ./
RUN npm install
COPY ./ ./

# También válido
COPY package.json .
RUN npm install
COPY . .
```

Con esto añadimos un paso más, muy corto, la copia de un archivo. Pero si este archivo no es modificado, cuando volvemos a construir el contenedor Docker no ejecuta el siguiente comando, ya que nada ha cambiado y, después, copia los archivos. Dado que no volvemos a instalar dependencias, el proceso `build` es mucho más rápido.

### Directorio de trabajo

Con `WORKDIR` especificamos cuál será el directorio de trabajo. A partir de este momento todo se hará de forma relativa a este directorio dentro del contenedor. `COPY` copia archivos desde nuestro ordenador al contenedor. Dado que hemos determinado el directorio de trabajo, si usamos `./` como destino los archivos se copiarán en el directorio de trabajo. Si no hubiéramos especificado directorio de trabajo, los archivos se habrían copiado en la raíz del contenedor o nos habría dado error, dependiendo de la versión de la imagen base.

El resultado sería:

```dockerfile
WORKDIR /usr/app
COPY ./ ./
RUN npm install

# Alternativo
WORKDIR /usr/app
COPY . .
RUN npm install
```

### Comando de inicio

La tercera línea de nuestro Dockerfile:

```dockerfile
CMD ["redis-server"]
```

Es la que ejecuta la aplicación en sí. Qué tiene que ejecutar la imagen cuando es iniciada como un contenedor.

### Custom Dockerfile

Podemos tener diferentes archivos Dockerfile, por ejemplo, según en el entorno en el que se vaya a desplegar la aplicación. Si vamos a crear una imagne para un entorno de desarrollo, podemos crear un archivo llamado Dockerfile.dev. Para construir una imagen usando este nombre de archivo, usamos el parámetro `-f`, donde especificamos el nombre del archivo:

```bash
$ docker build -f Dockerfile.dev .
```

### Exponer puertos

Con la instrucción EXPOSE podemos decir a Docker a través de qué puerto escuchará nuestra aplicación en tiempo de ejecución. Esto no mapea el puerto, por lo que no podremos acceder al contenedor a través del puerto especificado con esta instrucción; para esto usamos el parámetro -p del comando run. Este parámetro funciona a modo de documentación, ciertas aplicaciones (por ejemplo Travis para integración continua) leen esta configuración y gestionan los puertos:

```dockerfile
# Es lo mismo, por defecto el protocolo es TCP:
EXPOSE 80/tcp
EXPOSE 80

EXPOSE 80/udp
```

## Build: construir imágenes

Para crear nuestra imagen usaremos el comando:

```bash
$ docker build .
```

Este comando toma el contenido de Dockerfile y compone la imagen. También le hemos puesto un punto al final: el contexto. Este contexto es dónde está la ubicación donde tendremos los archivos y carpetas que vamos a encapsular en nuestra imagen y dónde está nuestro archivo Dockerfile.

Siguiendo el flujo de ejecución, Docker va creando imágenes y contenedores temporales de los que crea snapshots para crear nuevas imágenes temporales y nuevos contenedores con cada vez más programas (dependencias), tantas como hayamos especificado. Cuando no hay más instrucciones que ejecutar, devuelve la última imagen generada, que será nuestra imagen.

Mientras tengamos en nuestro Dockerfile detalladas imágenes o aplicaciones que hayan sido instaladas previamente, éstas se instalarán desde la caché y no se descargarán, lo que hará el proceso mucho más rápido.

### Etiquetas

Podemos etiquetar nuestra imagen para no tener que usar el container_id para su ejecución:

```bash
$ docker build -t myredis .
```

Podemos crear la etiqueta usando el formato:

```
nombre_de_usuario/repo_project_name:version
```

## Run: levantar contenedor

Con el comando run levantamos un contenedor partiendo de una imagen previamente construida. Con este comando podemos ver qué imágenes tenemos:

```bash
$ docker image ls
```

Donde vemos las imágenes, sus identificadores y, si tienen, su etiqueta.

Una vez hecho esto, podemos arrancar el contenedor con un nombre, haciendo referencia a la etiqueta y no al container_id:

```bash
$ docker run --name MyRedisApp myredis
```

Podemos ejecutar el contenedor en segundo plano y no bloquear el terminal, usando `-d`:

```bash
$ docker run -d --name MyRedisApp myredis
```

También podemos añadir comandos a ejecutar dentro del contenedor al final de la instrucción. En este caso añadimos el atributo `-it` para tener interacción con el terminal y ejecutamos el terminal `sh`:

```bash
$ docker run -it MyRedisApp sh
```

O ejecutar un comando que ejecuta test sobre un contenedor levantado usando su `id_container`:

```bash
$ docker exec -it 2568d84a681c npm run test
```

### Commit

Podemos crear imágenes de forma manual partiendo de contenedores que estén funcionando. Por ejemplo, iniciamos un contenedor con Alpine:

```
$ docker run -it alpine sh
```

E instalamos alguna aplicación en este contenedor:

```
# apk add --update redis
```

Esto habrá modificado el file system del contenedor, ya que ha instalado *Redis* en él. Sin cerrar este terminal para no cerrar el contenedor, en otro terminal podemos crear una nueva imagen a partir del estado actual de este contenedor:

```bash
$ docker ps # Para ver y copiar el id_container del contenedor que queremos copiar
$ docker commit -c 'CMD ["redis-server"]' d3fc85816e85 # -c => especificar el comando de inicio
sha256:b48d00800affea196a167d3dcc01f7248276e0be9a00757d072b76cdb646528d
# No es necesario copiar todo el hash, con un trozo es suficiente:
$ docker run --name MiInstancia b48d00800af
```

Y tendremos nuestra instancia de *Alpine* con *Redis* instalado corriendo con el nombre `MiInstancia`.

### Mapeo de puertos

Si levantamos un contenedor con un servicio que escucha peticiones en un puerto determinado, si intentamos acceder a dicho servicio desde nuestro ordenador (desde fuera del contenedor) no obtendremos respuesta, ya que la aplicación está escuchando peticiones dentro del contexto del contenedor. Para esto tenemos que levantar el contenedor especificando un mapeo de puertos, que no es más que una redirección de un puerto externo al contenedor a un puerto interno del contenedor.

```bash
# Atributo: -p puerto_externo:puerto_interno
$ docker run --name NodeAPP -p 8000:8080 simpleweb
```

Con esto levantamos un contenedor llamado NodeAPP y las peticiones externas al puerto 8000 serán atendidas por el puerto 8080 de nuestro contenedor.

### Volúmenes: mapear carpetas

Cada vez que modificamos un archivo de nuestro proyecto tenemos que reconstruir la imagen para que el archivo modificado se copie y quede dentro del contenedor que levantamos. Con los volúmenes ya no necesitamos esto, ya que lo que hacemos es, en lugar de copiar los archivos dentro del contenedor, creamos una referencia entre el directorio de trabajo del contenedor y el de nuestro ordenador local. Para configurar los volúmenes usamos el atributo `-v`:

```bash
$ docker run -p puerto_local:puerto_contenedor -v carpeta_local:carpeta_contenedor <id_imagen>

# Según el sistema operativo (pwd = directorio actual)
$ docker run -p 3000:8080 -v $(pwd):/app --name MyApp b48d00800af
$ docker run -p 3000:8080 -v /usr/name/myapp:/app --name MyApp b48d00800af
```

En ocasiones tenemos carpetas dentro del contenedor pero no fuera (por ejemplo, eliminamos node_modules para ahorrar tiempo en el build). Esto hará que run devuelva error ya que la referencia node_modules no encuentra carpeta en nuestro equipo; para corregirlo debemos añadir un volumen con la carpeta que no está en nuestro equipo:

```bash
$ docker run -p 3000:8080 -v /app/node_modules -v /usr/name/myapp:/app --name MyApp b48d00800af
```

Al indicar una carpeta del contenedor, sólo le decimos a Docker que no trate de mapear la unidad contra nada.

## Multi Step Build Proccess

Dentro de un único fichero Dockerfile podemos instanciar diferentes imágenes. Por ejemplo, para un servidor de producción no usaremos el servidor Node, usaremos Nginx,

```dockerfile
# Dockerfile

# Creamos la imagen con el alias 'builder'
FROM node:alpine AS builder
WORKDIR '/app'
COPY 'package.json' .
RUN npm install
COPY . .
RUN npm run build

FROM nginx
# Con el parámetro --from indicamos la imagen origen, usamos el alias 'build'
# y de la documentación de NGINX sacamos la carpeta donde se sirve por defecto:
COPY --from=builder /app/build /usr/share/nginx/html
```

Ahora ejecutaremos el comando:

```bash
$ docker build .

[+] Building 22.3s (14/14) FINISHED
 => [internal] load build definition from Dockerfile
 => => transferring dockerfile: 296B
 => [internal] load .dockerignore
 => => transferring context: 2B
 => [internal] load metadata for docker.io/library/nginx:latest
 => [internal] load metadata for docker.io/library/node:alpine
 => [internal] load build context
 => => transferring context: 5.99kB
 => [stage-1 1/2] FROM docker.io/library/nginx
 => [builder 1/6] FROM docker.io/library/node:alpine@sha256:417b3856d2e5d06385123f3924c36f5735fb1f690289ca69f2ac9
 => CACHED [builder 2/6] WORKDIR /app
 => CACHED [builder 3/6] COPY package.json .
 => CACHED [builder 4/6] RUN npm install
 => [builder 5/6] COPY . .
 => [builder 6/6] RUN npm run build
 => [stage-1 2/2] COPY --from=builder /app/build /usr/share/nginx/html
 => exporting to image
 => => exporting layers
 => => writing image sha256:9b49b4e173bb56e668ada14988fc6371d3b6310cfebc236b9389bb3520a3f314
```

Para construir la imagen tomamos el id de la imagen y la levantamos:

```bash
$ docker run -p 8080:80 9b49b4e173bb
```

Dado que hemos copiado el contenido de `/app/build` en la carpeta que Nginx sirve por defecto, sólo necesitamos mapear el puerto local que queremos utilizar (8080) con el puerto por defecto de Nginx (80).

## Stop: parar contenedores

Con el comando stop y el id o nombre del contenedor, podemos pararlo:

```bash
$ docker stop 852b5021a89b
$ docker stop nombre_del_contenedor
```

# Docker  Compose

## Qué es

Es una herramienta que nos sirve para compartir y compenetrar aplicaciones de diferentes contenedores. También se utiliza para levantar contenedores al mismo tiempo y nos simplifica el trabajo a la hora de pasar argumentos cuando ejecutamos `docker run`.

Supongamos que tenemos levantado un contenedor con Redis, funcionando y escuchando peticiones en su puerto. Por otro lado, tenemos levantado un contenedor Node en el que hemos instanciado una conexión con Redis. Pues bien, este último contenedor nos dará error ya que aunque ambos contenedores estén funcionando, no tienen capacidad para comunicarse entre ellos.

En esencia vamos a meter los mismos comandos que ejecutamos en terminal (`build`, `run`, etc) y los vamos a encapsular en un archivo `docker-compose.yaml` que ejecutaremos desde Docker Compose CLI:

```yaml
# docker-compose.yaml
version: '3'         # Línea obligatoria: versión del formato Docker Compose
services:            # Contenedores que necesitamos
  redis-server:      # Contenedor redis-server
    image: 'redis'   # Usando la imagen redis
  node-app:          # Contenedor node-app
    build: .         # Construído a partir de su Dockerfile
    ports:           # Mapeo de puertos
      - "4001:8081"
```

Cuando creamos estos dos contenedores a partir del mismo Docker Compose, éstos se pueden comunicar entre sí haciendo referencia únicamente al nombre del servicio, ya que se crea una red para albergar todos los contenedores indicados. Así, en nuestra aplicación, por ejemplo, Node, la conexión con el servidor Redis se hará:

```javascript
const redisClient = redis.createClient({
    host: 'redis-server', // Nombre del servicio en docker-compose.yaml
    port: 6379
});
```

Como hemos comentado, Docker Compose nos ayuda a ejecutar los comandos y parámetros que ejecutamos en terminal (`build`, `run`). Así:

```bash
$ docker run myimage
# Se corresponde con:
$ docker-compose up
$ docker-compose up -d # Ejecuta en segundo plano

$ docker build .
$ docker run myimage
# Se corresponde con:
$ docker-compose up --build

$ docker stop nombre_contenedor
# Se corresponde con
$ docker-compose down # Para y borra los contenedores
```

Como vemos, con `docker-compose up` no especificamos imagen, ya que lo que hacemos es leer el archivo `docker-compose.yaml` y ejecutar sus instrucciones, allí es donde están los nombres de las imágenes.

## Control de procesos

Con Docker Compose podemos controlar los procesos: qué hacer cuando nuestra aplicación falla y devuelve un código de error. Tenemos cuatro posibles actuaciones:

- `'no'` : no se reinicia el contenedor. Con comillas, ya que en yaml un no equivale a false, y no sería interpretado.
- `always` : se reinicia el contenedor siempre
- `on-failure` : se reinicia el contenedor sólo si se devolvió un código de error:
    - 0 = se detuvo el proceso, pero es ok, está controlado
    - (Resto de códigos) = código de error
- `unless-stopped` : levantar siempre a no ser que los desarrolladores, a través de la línea de comandos, paren el contenedor.

Se añade la instrucción `restart` seguida de la opción justo después de declarar el servicio en `docker-compose.yaml`:

```yaml
version: '3'
services:
  redis-server:
    image: 'redis'
    command:
      - --protected-mode no # Necesario por error en Redis
  node-app:
    restart: always # Se reiniciará el contenedor siempre.
    build: .
    ports:
      - "4001:8081"
```

## Comprobar el estado de los contenedores

Con el comando

```bash
$ docker-compose ps
```

vemos el estado de los contenedores que están en nuestro archivo `docker-compose.yaml`.

## Volúmenes

Docker Compose nos ayuda a simplificar los parámetros que añadimos a `docker run`. Uno de ellos es `volumes`, que nos sirve para mapear unidades entre nuestro ordenador y el contenedor:

```yaml
version: "3"
services:
  react-web-app:
    # build: .
    build:
      context: .
      dockerfile: Dockerfile.dev
    # environment:
      # - CHOKIDAR_USEPOLLING=true
    ports:
      - "3000:3000"
    volumes:
      - /app/node_modules
      - .:/app
```

En `volumes` hemos mapeado en primer lugar la carpeta que no tenemos en local y después la carpeta raíz de nuestra aplicación, tal y como hemos hecho anteriormente con `docker run`.

Aún conservamos la línea `COPY . .` en nuestro `Dockerfile.dev`. Es interesante mantener esta línea de cara a futura referencia, aunque queda inservible ya que cualquier acceso a /app irá directamente a nuestra carpeta local.

Para ver los puntos de montaje de un contenedor levantado:

```bash
$ docker inspect --format='{{json .Mounts}}' MyApp
```

# Kubernetes

## Comandos

### Iniciar un servicio

```bash
$ kubectl apply -f nombre-archivo.yaml
```

### Listar servicios por tipo

```bash
$ kubectl get pods
$ kubectl get services
$ kubectl get deployments

$ kubeclt get pods,services

$ kubectl get all
```

### Lista servicios con namespaces

```bash
$ kubectl get deployments --all-namespaces
$ kubectl get all --all-namespaces
```

```
NAMESPACE     NAME                READY   UP-TO-DATE   AVAILABLE   AGE
default       client-deployment   1/1     1            1           28m
default       web                 5/5     5            5           5m50s
kube-system   coredns             2/2     2            2           32h
```

> Fuente: https://stackoverflow.com/questions/40686151/kubernetes-pod-gets-recreated-when-deleted

### Descripción de un servicio

```bash
# kubectl describe [type]/[name]
$ kubectl describe deplyment/client-deployment
```



### Borrar despliegue

```bash
# kubectl delete -n [NAMESPACE] [type] [DEPLOYMENT]
$ kubectl delete -n default deployment web
```

> Fuente: https://stackoverflow.com/questions/40686151/kubernetes-pod-gets-recreated-when-deleted

### Cambiar una propiedad de un servicio

```bash
# kubectl set [property] [type]/[name] [container_name] = [new image to use]

# Forzar la actualización de un servicio:
$ kubectl set image deployment/client-deployment client=stephengrinder/multi-client:v5
```

# Recursos

## Docker

### Documentación oficial

https://docs.docker.com/

### AWS: ¿Qué es Docker?

https://aws.amazon.com/es/docker/

### Artículos

- Running a Docker container as a non-root user: https://medium.com/redbubble/running-a-docker-container-as-a-non-root-user-7d2e00f8ee15
- Running Docker Containers as Current Host User: https://jtreminio.com/blog/running-docker-containers-as-current-host-user/

## Kubernetes

### Official cheatsheet

https://kubernetes.io/docs/reference/kubectl/cheatsheet/

### Cursos IBM

https://www.ibm.com/es-es/cloud/kubernetes-service/kubernetes-tutorials?utm_content=SRCWW&p1=Search&p4=43700066871613664&p5=p&gclid=Cj0KCQjw_fiLBhDOARIsAF4khR39sNFIo5Ofe5sAc6FG527DTbobS-8ZCiYXDLAa4NB_c5SCxdzicZoaAlTVEALw_wcB&gclsrc=aw.ds

## Integración continua

### Travis

https://www.travis-ci.com/

