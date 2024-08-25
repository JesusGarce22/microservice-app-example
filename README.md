# Aplicación de Microservicios - PRFT DevOps Training

Esta es la aplicación que usarás durante todo el entrenamiento. Esto, con suerte, te enseñará los fundamentos que necesitas para trabajar en un proyecto real. Encontrarás una aplicación básica de tareas pendientes (TODO) diseñada con una [arquitectura de microservicios](https://microservices.io). Aunque es una aplicación de tareas, es interesante porque los microservicios que la componen están escritos en diferentes lenguajes de programación o frameworks (Go, Python, Vue, Java y NodeJS). Con este diseño experimentarás con múltiples herramientas de construcción y entornos.

## Componentes
En cada carpeta puedes encontrar una explicación más detallada de cada componente:

1. [API de Usuarios](/users-api): Es una aplicación de Spring Boot. Proporciona perfiles de usuario. Por el momento, no ofrece CRUD completo, solo permite obtener un solo usuario y todos los usuarios.
2. [API de Autenticación](/auth-api): Es una aplicación en Go, que proporciona funcionalidad de autenticación. Genera tokens [JWT](https://jwt.io/) que se utilizan en las otras APIs.
3. [API de TODOs](/todos-api): Es una aplicación en NodeJS que proporciona funcionalidad CRUD sobre los registros de tareas pendientes (TODO) de los usuarios. Además, registra las operaciones de "crear" y "eliminar" en una cola de [Redis](https://redis.io/).
4. [Procesador de Mensajes de Registro](/log-message-processor): Es un procesador de colas escrito en Python. Su propósito es leer mensajes de una cola de Redis y mostrarlos en la salida estándar.
5. [Frontend](/frontend): Es una aplicación en Vue que proporciona la interfaz de usuario (UI).

## Arquitectura

A continuación se muestra el diagrama de componentes que describe cada uno de ellos y sus interacciones.
![microservice-app-example](/arch-img/Microservices.png)

## Paso a Paso para Desplegar y Monitorear con Docker Compose, Prometheus, Grafana y cAdvisor

### 1. Hacer fork al repositorio original
Haz un fork del repositorio a tu cuenta de GitHub para trabajar con tu propia copia del código.

### 2. Clonar el repositorio
Clona el repositorio localmente usando el comando:
```
git clone <url_del_repositorio>
```
### 3. Comprender el orden y las relaciones entre los microservicios
Familiarízate con la arquitectura general y el flujo de datos entre los microservicios.

![Diagrama](/arch-img/MyApp.drawio.png)
### 4. Crear los archivos Dockerfile para cada microservicio
Es necesario crear un archivo `Dockerfile` en cada uno de los directorios de los microservicios. Cada archivo debe contener las instrucciones específicas para construir la imagen Docker del servicio correspondiente.

### 5. Crear un archivo Docker Compose
Crea un archivo [docker-compose.yml](docker-compose.yml) con las configuraciones e instrucciones para orquestar el despliegue de todos los microservicios. Asegúrate de incluir las dependencias y la configuración de redes necesarias.

### 6. Desplegar con Docker Compose
Una vez que todo esté configurado, ejecuta el siguiente comando para desplegar los servicios en contenedores:
```
docker-compose up -d
```

Ingresar a [localhost:8080](http//:localhost:8080) para ver la pagina principal del fronted

![](/arch-img/Todos-app.png)

### 7. Subir imágenes a DockerHub
Sube las imágenes de los microservicios a tu cuenta de DockerHub para facilitar el despliegue en otros entornos. Use los siguientes comandos.

```bash
docker login
docker build -t <DockerHubUsername>/nombre-de-imagen:latest ./ubicacion
docker push <DockerHubUsername>/<nombre-de-imagen>:<tag>

#Ejemplo

docker build -t jesusgarces22/users-api:latest ./users-api
docker push jesusgarces22/users-api:latest

```

![](/arch-img/imgsDH.png)

### 8. Configurar Prometheus, Grafana y cAdvisor
Añade las configuraciones necesarias para monitorear los contenedores usando Prometheus, Grafana y cAdvisor modificando el [docker-compose.yml](docker-compose.yml). Define un [archivo de configuración para Prometheus](prometheus.yml) y asegúrate de que todos los endpoints de métricas estén accesibles.

### 9. Desplegar el sistema de monitoreo
Inicia Prometheus, Grafana y cAdvisor como servicios en contenedores usando Docker Compose.

```
docker-compose up -d
```
![](/arch-img/servicios_run.png)
![](/arch-img/contenedores%20en%20docker.png)

### 10. Iniciar sesión en Grafana
Accede a Grafana en el puerto configurado (por defecto http://localhost:3000), inicia sesión usando las credenciales predeterminadas (usuario: admin, contraseña: admin).

NOTA:  este flujo:

**cAdvisor** es el encargado de recolectar métricas sobre el rendimiento de los contenedores, como el uso de CPU, memoria, disco, y red. Este servicio expone las métricas en un formato que Prometheus puede entender a través de un endpoint HTTP (normalmente /metrics).

**Prometheus** es quien recoge y almacena las métricas de cAdvisor y otros servicios que estén exponiendo datos. Prometheus accede periódicamente al endpoint de cAdvisor (y otros servicios si están configurados) para extraer las métricas.

**Grafana** es la herramienta que visualiza las métricas almacenadas en Prometheus. Mediante consultas (queries) a la base de datos de Prometheus, Grafana genera gráficos y paneles de control (dashboards) para presentar las métricas de manera visual e interpretable.
- Primero, consulte las interfaces de Prometheus http://localhost:9090 y cAdvisor http://localhost:8081

![](/arch-img/prometheus-tarjet.png)

En esta imagen se pueden ver los contenedores que esta monitoreando prometheus

![](/arch-img/Cardvisor.png)

Aqui se puede ver los contenedores de Docker que esta monitoreando cadvisor mostrando su id

Algunas metricas que se pueden ver en Cadvisor:

![](/arch-img/Cardbana.png)

![](/arch-img/grafica_cardvana.png)

### 11. Crear un nuevo Data Source en Grafana
Dentro de Grafana, configura Prometheus como fuente de datos (Data Source) apuntando a la URL correcta (por ejemplo, http://prometheus:9090).

![](/arch-img/datasourc.png)

### 12. Construir un Dashboard en Grafana
Crea un nuevo Dashboard en Grafana donde podrás visualizar gráficas basadas en las métricas proporcionadas por Prometheus.

![](/arch-img/Dashbopard.png)

### 13. Configurar las métricas que se desean monitorear
Configura las queries necesarias para monitorear las métricas importantes. Puedes monitorear el uso de CPU, memoria, tráfico de red, y actividad de los microservicios. Algunos ejemplos acontinuacion:

![](/arch-img/Query.png)

![](/arch-img/memory-grafana.png)

![](/arch-img/up.png)

![](/arch-img//Memoria%20cache.png)

![](/arch-img/cache%20img%20max.png)


### 14. Interpretar las métricas
Observa las métricas en el Dashboard de Grafana y utilízalas para tomar decisiones informadas sobre la salud y el rendimiento de tus microservicios.