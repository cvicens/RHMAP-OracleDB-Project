Buenas

Elementos de la prueba:
Una VM en Azure que corre un docker que tiene una Oracle XE con un script que carga datos de ejemplo y un procedimiento PL
El docker que corre en Azure y que yo corro en local para desarrollar más rápido
Un docker que corres en local para poder ejecutar:
Una Cloud App aquí
Un MBaaS Service aquí
Variable de entorno en el MBaaS Service para apuntar a la XE que corren en Azure: NODE_ORACLEDB_CONNECTIONSTRING con este valor => 52.166.65.138:41521/xe
Para acceder a la VM en Azure:
IP:  52.166.65.138
Usuario: cvicens
Fichero de clave: azure-docker_id_rsa
Comando: ssh -i azure-docker_id_rsa cvicens@52.166.65.138
Para parar, ejecutar de nuevo el docker en Azure (si hiciera falta):
cd rhmap/oraclexe-rhmap/

cat de README.md

Copias las vars de entorno del fichero README: PROJECT_ID, IMAGE_NAME, IMAGE_VERSION, CONTAINER_NAME; con sus valores claro.

Exportas las vars como en README.md, con los valores que allí aparecen

Ahora con docker stop y docker rm utilizando la var CONTAINER_NAME te cargas el docker

Como tienes las vars cargadas, para ejecutar de nuevo el contenedor como demonio: docker run -p=40022:22 -p=41521:1521 -d -v $(pwd)/projects:/usr/projects -e ORACLE_ALLOW_REMOTE=true --name $CONTAINER_NAME $PROJECT_ID/$IMAGE_NAME:$IMAGE_VERSION

Con docker ps coges el ID y con: docker logs -f <ID> puedes ver como va el arranque,

Al final tiene que poner lo siguiente:
Commit complete.
Procedure created.
Procedure created.
SQL> Disconnected from Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

El git del docker que contiene la Oracle XE: https://github.com/cvicens/oraclexe-rhmap
El git del docker para desarrollar en local, con mongo, redis y las librerías de Oracle aquí: https://github.com/cvicens/oracle-mbaas-service-rhmap

Para preparar el entorno en local:
Clonas los dos gits en un directorio
$ mkdir -p dominion/patronato
$ git clone https://github.com/cvicens/oraclexe-rhmap
$ git clone https://github.com/cvicens/oracle-mbaas-service-rhmap
Creamos y arrancamos el docker con la DB, ahora no lo queremos demonio en local (o como quieras Edu)
$ cd oraclexe-rhmap
Cargamos variables (las mismas y mismos valores que en Azure), las tienes en el README.md del docker => 
export PROJECT_ID="rhmap-docker"
export IMAGE_NAME="oraclexe-11g-node-4.4"
export IMAGE_VERSION="v1.0"
export CONTAINER_NAME="oraclexe-rhmap-docker-dev"
Creamos la imagen: docker build -t $PROJECT_ID/$IMAGE_NAME:$IMAGE_VERSION .
Ejecutamos la imagen en modo interactivo, vamos bash
$ docker run -p=40022:22 -p=41521:1521 -it --rm -v $(pwd)/projects:/usr/projects -e ORACLE_ALLOW_REMOTE=true --name $CONTAINER_NAME $PROJECT_ID/$IMAGE_NAME:$IMAGE_VERSION /bin/bash
Arrancamos el entorno de desarrollo, con -it -rm no debe ser un demonio (esto es necesario por la librerías -que te comparto luego por Drive- de Oracle que no valen para Mac ya me darás las gracias ;-):
$ cd oracle-mbaas-service-rhmap
Crea un directorio 'projects' si es que no viene con el clonado, es el directorio de desarrollo dentro de la imagen
Copia los zips de oracle: instantclient-basic-linux.x64-12.2.0.1.0.zip y instantclient-sdk-linux.x64-12.2.0.1.0.zip al directorio, el build de la imagen hace el unzip, etc.
Cargamos variables (estas son distintas, ojo), las tienes en el README.md del docker => 
export PROJECT_ID="rhmap-docker"
export IMAGE_NAME="oracle-mbaas-service-node-4.4"
export IMAGE_VERSION="v1.0"
export CONTAINER_NAME="oracle-mbaas-service-rhmap-docker-dev"
Creamos la imagen: docker build -t $PROJECT_ID/$IMAGE_NAME:$IMAGE_VERSION .
$ docker run -p=8001:8001 -p=8011:8011 -it --rm -v $(pwd)/projects:/usr/projects --name $CONTAINER_NAME $PROJECT_ID/$IMAGE_NAME:$IMAGE_VERSION /bin/bash
Clona los proyectos relevantes en la carpeta /usr/projects/
Cloud App: git@git.tom.redhatmobile.com:techlab/Administracion-Electronica-Administracion-Electronica-Cloud-App.git
MBaaS Service: git@git.tom.redhatmobile.com:techlab/Patronato-DB-Service-Patronato-DB-Service.git
npm install de ambos proyectos, verás que el MBaaS service utiliza la librería de Oracle: oracledb
Para levantar los dos proyectos en el docker: cloud app en 8011 y mbaas en 8001
docker ps => para coger el id y luego docker exec -it <ID> /bin/bash para engancharte otra consola el mismo docker
OJO, en el MBAAS, el fichero ./lib/dbconfig.js es donde se dice donde está la BD en local... mira en la línea 11
connectString : process.env.NODE_ORACLEDB_CONNECTIONSTRING || "192.168.1.131:41521/xe"
Como ves corriendo en RHMAP la var de entorno lo soluciona, pero en local la ip 192.168.1.131 es la de tu mac, vamos que la tienes que cambiar... y no la subas a GIT... otra forma sería expor de la var en el docker probablemente más elegante....
Para probar:
Local (en el docker se entiende): 
http://localhost:8011/vt/2017/69
http://localhost:8011/vt/pdf/2017/69 -> la del PDF te va a fallar si arrancas la Cloud App con grunt serve:local, esto es porque deja el HTML con el que se genera el PDF en ./lib/tmp y watch reinicia la cloud 
para probar el PDF mejor arranca con: node application.js pero antes exporta esta variables (OJO, siempre, en el docker):
export FH_SERVICE_MAP="{\"qmuab4ydkglk6lmpevz6sgzb\":\"http://127.0.0.1:8001\",\"SERVICE_GUID_2\":\"https://host-and-path-to-service\"}";

export FH_USE_LOCAL_DB = 'true'

En RHMAP
https://techlab-bunnkavopk7ps5sobzyohac6-teama.mbaas2.tom.redhatmobile.com/vt/2017/7
https://techlab-bunnkavopk7ps5sobzyohac6-teama.mbaas2.tom.redhatmobile.com/vt/pdf/2017/7 -> aquí funciona sin problemas no hay watch dando por saco
Por cierto /vt/:year/:codigo_municipio ;-)

Ahora crea una app con Ionic 2 siguiendo este artículo: http://www.opensourcerers.org/ionic-v2-app-for-rhmap/ probablemte (o se harcodea) necesites un servicio que consulta la tabla MUNICIPALITIES para un drop down...

De todos modos hablamos un día de estos, vete dando feedback de este monstruo (luego no es complicado... pero este email es un poco monstruo)

Saludos!