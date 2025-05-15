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
- Docker Compose: Necesita tener instalado Docker Compose para implementar los servicios. Consulte la documentación oficial [aquí](https://docs.docker.com/compose/install/) para obtener las instrucciones de instalación si no está incluida en su instalación de Docker.
- Imágenes necesarias: Necesita las imágenes de Docker necesarias para Hadoop y Spark. Puedes extraer las imágenes ejecutando t
