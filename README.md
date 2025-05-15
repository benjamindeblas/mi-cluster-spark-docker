# mi-cluster-spark-docker
En este repositorio, documento los pasos para configurar un clúster Spark con Docker. El clúster es una pequeña red local (LAN) de hasta 20 nodos con el objetivo de procesar 1 TB de datos en más de una hora y media. Este no es un proyecto de producción, sino un experimento para aprender un poco sobre estas tecnologías y compartir lo aprendido.


## Objetivo

En este repositorio documentaré los pasos para configurar un cluster Spark usando Docker. 
El cluster tendrá hasta X nodos con el objetivo de procesar datos [describe tu caso de uso].

Este es un proyecto [experimental/de aprendizaje/de producción] para aprender sobre estas tecnologías.

## Estado del proyecto

- [ ] Documentación completa
- [ ] Configuración básica
- [ ] Pruebas iniciales
- [ ] Optimizaciones

## Cómo usar

1. Clona el repositorio
2. Ejecuta `./setup_swarm.sh`
3. [Otros pasos...]


-------------------
# Objetivo
En este repositorio, explicaré algunos de los pasos que seguí para configurar un clúster Spark con hasta 20 nodos usando Docker. Esta documentación abarca todo, desde la instalación del software necesario hasta la configuración de la red, e incluye pruebas e informes durante el proceso.

Si solo te interesan los pasos para configurar el clúster, puedes omitir las primeras secciones e ir directamente a la sección [Implementación del clúster](#cluster-deployment).

# Instalación del software
## Docker
Para instalar Docker en Debian, puedes consultar la documentación oficial [aquí](https://docs.docker.com/engine/install/debian/). En mi experiencia, recomiendo usar Docker Desktop, ya que es más intuitivo y fácil de usar. Sin embargo, puedes instalar Docker Engine si lo prefieres o si tienes recursos limitados. Además, esto es lo que probablemente encontrarás en entornos de trabajo reales.

Para conocer los pasos de instalación detallados, consulte el documento [Desglose de comandos](Docs/commands_breakdown.md#docker-installation).

# Prueba inicial de contenedores
Para probar la instalación de Docker, puede ejecutar el siguiente comando:
```bash
docker run hello-world
```
Si todo funciona correctamente, debería ver un mensaje indicando que la instalación se realizó correctamente.

# Spark
Para configurar Spark con Docker, usaremos contenedores Docker para crear un nodo maestro y uno de trabajo. Para obtener instrucciones detalladas sobre cómo extraer la imagen de Spark y ejecutar las pruebas iniciales, consulte la documentación [Prueba inicial de Spark](Docs/Detailed-steps/Initial-spark-test.md).

Hasta ahora, contamos con un clúster Spark en funcionamiento con un nodo maestro y uno de trabajo que se ejecutan en contenedores Docker. El siguiente paso es escalar el clúster para incluir más nodos de trabajo y probar su rendimiento con conjuntos de datos más grandes y aplicaciones más complejas.

# Docker Swarm
Docker Swarm es una herramienta de orquestación de contenedores que permite gestionar un clúster de nodos Docker en varias máquinas. Dado que las redes Docker se limitan a varios contenedores en el mismo host, en nuestro caso, se busca la posibilidad de tener varios contenedores en varios hosts.

Para obtener instrucciones detalladas sobre cómo configurar un Docker Swarm, crear redes superpuestas y etiquetar nodos, consulte el documento [Explicación de Swarm](Docs/Detailed-steps/Swarm-explained.md).

## Docker Compose
Para implementar servicios fácilmente en Docker Swarm, podemos usar Docker Compose. Docker Compose es una herramienta que permite definir y ejecutar aplicaciones Docker multicontenedor mediante un archivo YAML.

Para usar Docker Compose, debe crear un archivo `docker-compose.yml` en el directorio donde desea implementar los servicios. El archivo debe contener las configuraciones de los servicios que desea implementar.

Para obtener información detallada sobre los comandos para trabajar con Docker Compose y Swarm, consulte el documento [Commands Breakdown](Docs/commands_breakdown.md#docker-composestack-commands).

# Hadoop
Hadoop es un marco de almacenamiento y procesamiento distribuido que se utiliza comúnmente en aplicaciones de big data. Para obtener información detallada sobre Hadoop y HDFS en este proyecto, consulte el documento [HDFS Explained](Docs/Detailed-steps/HDFS-explained.md).

# Implementación del clúster

Los detalles de la implementación del clúster se encuentran en el archivo [Compose](Docs/docker-compose.yml), que usaremos para implementar los servicios en Docker Swarm.

También necesita obtener los scripts de Hadoop y Spark de la carpeta [Docs]. Los archivos necesarios son:

- `init-datanode.sh`: Este script se utiliza para inicializar el contenedor DataNode y configurarlo para que se conecte al NameNode.
- `start-hdfs.sh`: Este script se utiliza para iniciar los servicios HDFS en los nodos maestro y de trabajo.
- `spark-start.sh`: Este script se utiliza para iniciar los servicios Spark en los nodos maestro y de trabajo.
- `docker-compose.yml`: Este archivo contiene las configuraciones de los servicios, incluyendo los nodos maestro y de trabajo de Hadoop, así como los nodos maestro y de trabajo de Spark.
- `hadoop-config/`: Esta carpeta contiene los archivos de configuración de Hadoop, incluyendo `core-site.xml` y `hdfs-site.xml`, que se utilizan para configurar el clúster de Hadoop.

Tras incluir el clúster Hadoop en Docker Swarm, podemos implementar los servicios con Docker Compose.

Para implementar los servicios, si aún no ha clonado el repositorio, hágalo para evitar copiar los archivos manualmente.

Deberá navegar al directorio donde se encuentra el repositorio clonado y acceder a la carpeta `Docs`.

## Requisitos de software

Para ejecutar el clúster Spark-Hadoop, necesita tener instalado el siguiente software en su sistema:
- Docker: Puede instalar Docker siguiendo las instrucciones del documento [Instalación de Docker](Docs/commands_breakdown.md#docker-installation).
- Docker Compose: Necesita tener instalado Docker Compose para implementar los servicios. Consulte la documentación oficial [aquí](https://docs.docker.com/compose/install/) para obtener las instrucciones de instalación si no está incluido en su instalación de Docker.
- Imágenes necesarias: Necesita tener las imágenes de Docker necesarias para Hadoop y Spark. Puede extraer las imágenes ejecutando los siguientes comandos:
```bash
docker pull bitnami/spark:3.5.5
docker pull apache/hadoop:3.4.1
# No es necesario, pero se recomienda encarecidamente para los nodos maestros
docker pull jupyter/pyspark-notebook:latest
```

## Configuración de Docker Swarm

El primer paso es crear un Docker Swarm si aún no lo ha hecho. Puede hacerlo ejecutando el siguiente comando en el nodo maestro (para las pruebas iniciales, puede usar el nodo que desee, ya sea una instalación completa de Linux o una máquina virtual):
```bash
docker swarm init --advertise-addr <master-ip>
```

Copie el token generado tras ejecutar el comando, ya que se utilizará para unir los nodos de trabajo al swarm.

Luego, conecte los nodos de trabajo al enjambre ejecutando el siguiente comando en cada uno:
```bash
# En cada nodo de trabajo
docker swarm join --token <token> <master-ip>:2377
```

Después de conectar los nodos, verifique que formen parte del enjambre ejecutando `docker node ls` en el nodo maestro. Conserve los ID y nombres de host de los nodos para etiquetarlos posteriormente.

Luego, cree la red superpuesta que utilizarán los servicios con:
```bash
docker network create -d overlay --attachable hadoop_spark_net
```

Etiquete los nodos con la etiqueta `h-role` para identificar los nodos maestro y de trabajo. Esta operación se realiza en el nodo administrador, por lo que puede ejecutar los siguientes comandos para etiquetar los nodos:
```bash
# Para el nodo maestro
docker node update --label-add spark-role=master <master-node-id>
# Para los nodos de trabajo
docker node update --label-add h-role=worker <worker-node-id>
docker node update --label-add spark-role=worker <worker-node-id>
```
Puede reemplazar `<master-node-id>` y `<worker-node-id>` con el ID del nodo o el nombre de host de los nodos si no hay nombres de host duplicados en el clúster.

En cada nodo, debe crear los siguientes directorios para almacenar los datos HDFS:
```bash
sudo mkdir -p /data/hadoop
sudo chmod 777 /data/hadoop
```

Después de etiquetar todos los nodos y crear los directorios, puede implementar los servicios mediante Docker Compose:
```bash
docker stack deploy -c docker-compose.yml <stack-name>
```

Puede reemplazar `<stack-name>` con el nombre que desee asignar a la pila. Sin embargo, recomiendo encarecidamente usar nombres cortos, ya que el acceso a los contenedores se realiza mediante el nombre de la pila y el nombre del servicio. Si usa nombres largos, acceder a los contenedores será tedioso.

Compruebe el estado de los servicios ejecutando:
```bash
docker service ls
```
Si todo funciona correctamente, debería ver los servicios en ejecución y las réplicas de cada uno. Además, si desea ver los registros de un servicio específico, puede ejecutar:
```bash
docker service logs <stack-name>_<service-name>
```

Una vez que todo funcione correctamente, puede acceder a las interfaces web de Hadoop y Spark a través de las siguientes URL en su navegador web:
- Interfaz web de Hadoop: `http://<master-ip>:9870`
- Interfaz web de Spark: `http://<master-ip>:8080`
- Jupyter Lab: `http://<master-ip>:8888/lab`

Siga los pasos del documento [Pasos de ejecución](Docs/Detailed-steps/Execution-steps.md) para realizar una prueba sencilla y comprobar si el clúster de Hadoop funciona correctamente.

Notas importantes

Al implementar los servicios, es importante tener en cuenta la estructura de archivos, ya que los scripts utilizan rutas relativas a los archivos. Por lo tanto, es necesario mantener la estructura básica del repositorio para evitar errores.

La estructura básica del repositorio es la siguiente:

```
dockerized-spark-cluster-set-up/ # Repositorio raíz
│
├── README.md # Documentación principal
│
├── Docs/ # Contiene todos los archivos de implementación
│ │
│ ├── docker-compose.yml # Configuración principal de implementación
│ │
│ ├── hadoop-config/ # Archivos de configuración de Hadoop
│ │ ├── core-site.xml
│ │ └── hdfs-site.xml
│ │
│ ├── init-datanode.sh # Script de inicialización de DataNode
│ ├── start-hdfs.sh # Script de inicio de HDFS
│ ├── spark-start.sh # Script de inicio de Spark
│ │
│ └── [Archivos de documentación...] # Varios archivos de documentación de Markdown
│
└── [Otros archivos del repositorio...] # .gitignore, etc.
```

Si desea copiar solo los archivos necesarios para implementar los servicios, debe mantener la siguiente estructura:
```
DIRECTORIO-DE-IMPLANTACIÓN/
Si desea copiar solo los archivos necesarios para implementar los servicios, debe mantener la siguiente estructura:
```
DEPLOYMENT-DIRECTORY/ # Cualquier directorio en el administrador de swarm
│
├── docker-compose.yml # Configuración principal de implementación
│
├── hadoop-config/ # Debe estar en este subdirectorio
│ ├── core-site.xml
│ └── hdfs-site.xml
│
├── init-datanode.sh # Debe estar en el mismo directorio que docker-compose.yml
├── start-hdfs.sh
└── spark-start.sh
```

Debe ejecutar el comando `stack deploy` en el mismo directorio que el Archivo `docker-compose.yml`.

Si ha implementado correctamente los servicios, debería poder ver las interfaces web de Hadoop y Spark accediendo a las siguientes URL en su navegador web:
- Interfaz web de Hadoop: `http://<master-ip>:9870`
- Interfaz web de Spark: `http://<master-ip>:8080`
- Jupyter Lab: `http://<master-ip>:8888/lab`

Aquí verá algo similar a esto:
![Interfaz web de Hadoop](assets/HadoopS-cluster.png)
![Interfaz web de Spark](assets/SparkH-cluster.png)

## Implementación local (Ignorar si no se usan máquinas virtuales)

Al ejecutar Docker sin conexión, surge un problema crítico: al intentar implementar una pila, Docker Swarm intenta extraer las imágenes necesarias de Docker Hub. En un entorno sin conexión, esto falla porque los registros son inaccesibles, lo que provoca que la implementación se detenga con errores de "imagen no encontrada", incluso si las imágenes existen localmente en los nodos.

Para solucionar esto, debemos configurar un registro local de Docker accesible para todos los nodos del Swarm. Este registro alojará las imágenes necesarias. Siga estos pasos en el nodo administrador:

```bash
# 1. Implemente el servicio de registro
docker service create --name registry --publish published=5000,target=5000 registry:2

# Ejemplo para Spark
docker tag bitnami/spark:3.5.5 <registry-host-ip>:5000/bitnami/spark:3.5.5
docker push <registry-host-ip>:5000/bitnami/spark:3.5.5

# Ejemplo para Hadoop
docker tag apache/hadoop:3.4.1 192.168.56.2:5000/apache/hadoop:3.4.1
docker push 192.168.56.2:5000/apache/hadoop:3.4.1

# Ejemplo para Jupyter
docker tag jupyter/pyspark-notebook:latest 192.168.56.2:5000/jupyter/pyspark-notebook:latest
docker push 192.168.56.2:5000/jupyter/pyspark-notebook:latest

```
Crear el siguiente archivo para habilitar el registro local en cada nodo, incluido el administrador.

```bash
// ruta de archivo: /etc/docker/daemon.json
{
"insecure-registries": ["<registry-host-ip>:5000"]
}
```
