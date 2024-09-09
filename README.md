## Paso a Paso para Desplegar con Vagrant y Virtualbox

En esta rama vamos a desplegar el mismo proyecto, pero usando Vagrant y Virtualbox. La estrategia que se va usar es la siguiente: 

- Cada servicio se ejecuta en una VM separada con una dirección IP privada única.
- Para cada VM, se instala Docker y se ejecuta el contenedor correspondiente utilizando la imagen de Docker Hub.
- Se especifica la memoria asignada a cada VM, se debe contar con almenos 15G de almacenamiento disponible.
- Todas las VMs están en la misma red privada, lo que permite la comunicación entre los microservicios.

### 1. Crear el archivo Vagrantfile
```
Vagrant.configure("2") do |config|

    config.vm.boot_timeout = 900

    # Configuración para Redis
    config.vm.define "redis" do |redis|
      redis.vm.box = "ubuntu/focal64"
      redis.vm.network "private_network", ip: "192.168.56.10"
      redis.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end
      redis.vm.provision "docker" do |d|
        d.run "redis", image: "redis:7.0", args: "-p 6379:6379"
      end
    end
  
    # Configuración para Users API
    config.vm.define "users-api" do |users_api|
      users_api.vm.box = "ubuntu/focal64"
      users_api.vm.network "private_network", ip: "192.168.56.11"
      users_api.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end
      users_api.vm.provision "docker" do |d|
        d.run "users-api", image: "jesusgarces22/users-api:latest", args: "-p 8083:8083", env: {
          "JWT_SECRET" => "PRFT",
          "SERVER_PORT" => "8083"
        }
      end
    end
  
    # Configuración para Auth API
    config.vm.define "auth-api" do |auth_api|
      auth_api.vm.box = "ubuntu/focal64"
      auth_api.vm.network "private_network", ip: "192.168.56.12"
      auth_api.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end
      auth_api.vm.provision "docker" do |d|
        d.run "auth-api", image: "jesusgarces22/auth-api:latest", args: "-p 8000:8000", env: {
          "JWT_SECRET" => "PRFT",
          "AUTH_API_PORT" => "8000",
          "USERS_API_ADDRESS" => "http://192.168.56.11:8083"
        }
      end
    end
  
    # Configuración para Log Message Processor
    config.vm.define "log-message-processor" do |log_processor|
      log_processor.vm.box = "ubuntu/focal64"
      log_processor.vm.network "private_network", ip: "192.168.56.13"
      log_processor.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end
      log_processor.vm.provision "docker" do |d|
        d.run "log-message-processor", image: "jesusgarces22/log-message-processor:latest", env: {
          "REDIS_HOST" => "192.168.56.10",
          "REDIS_PORT" => "6379",
          "REDIS_CHANNEL" => "log_channel"
        }
      end
    end
  
    # Configuración para Todos API
    config.vm.define "todos-api" do |todos_api|
      todos_api.vm.box = "ubuntu/focal64"
      todos_api.vm.network "private_network", ip: "192.168.56.14"
      todos_api.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end
      todos_api.vm.provision "docker" do |d|
        d.run "todos-api", image: "jesusgarces22/todos-api:latest", args: "-p 8082:8082", env: {
          "JWT_SECRET" => "PRFT",
          "TODO_API_PORT" => "8082",
          "REDIS_HOST" => "192.168.56.10",
          "REDIS_PORT" => "6379",
          "REDIS_CHANNEL" => "log_channel"
        }
      end
    end
  
    # Configuración para Frontend
    config.vm.define "frontend" do |frontend|
      frontend.vm.box = "ubuntu/focal64"
      frontend.vm.network "private_network", ip: "192.168.56.15"
      frontend.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
      end
      frontend.vm.provision "docker" do |d|
        d.run "frontend", image: "jesusgarces22/frontend:latest", args: "-p 8080:8080", env: {
          "PORT" => "8080",
          "AUTH_API_ADDRESS" => "http://192.168.56.12:8000",
          "TODOS_API_ADDRESS" => "http://192.168.56.14:8082"
        }
      end
    end
  
    # Configuración para cAdvisor
    config.vm.define "cadvisor" do |cadvisor|
      cadvisor.vm.box = "ubuntu/focal64"
      cadvisor.vm.network "private_network", ip: "192.168.56.16"
      cadvisor.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
      end
      cadvisor.vm.provision "docker" do |d|
        d.run "cadvisor", image: "google/cadvisor:latest", args: "-p 8081:8080 -v /:/rootfs:ro -v /var/run:/var/run:ro -v /sys:/sys:ro -v /var/lib/docker/:/var/lib/docker:ro"
      end
    end
  
    # Configuración para Prometheus
    config.vm.define "prometheus" do |prometheus|
      prometheus.vm.box = "ubuntu/focal64"
      prometheus.vm.network "private_network", ip: "192.168.56.17"
      prometheus.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
      end
      prometheus.vm.provision "docker" do |d|
        d.run "prometheus", image: "prom/prometheus", args: "-p 9090:9090 -v /vagrant/prometheus.yml:/etc/prometheus/prometheus.yml"
      end
    end
  
    # Configuración para Grafana
    config.vm.define "grafana" do |grafana|
      grafana.vm.box = "ubuntu/focal64"
      grafana.vm.network "private_network", ip: "192.168.56.18"
      grafana.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
      end
      grafana.vm.provision "docker" do |d|
        d.run "grafana", image: "grafana/grafana", args: "-p 3000:3000 -v grafana-storage:/var/lib/grafana", env: {
          "GF_SECURITY_ADMIN_PASSWORD" => "admin"
        }
      end
    end
  
  end
```
### 2.  Iniciar el Despliegue con Vagrant
```
vagrant up

#Tambien puedes usar 

vangrant up nombre_de_la_VM
```
![](/arch-img/logVangrantusers.PNG)
Esto puede tardar mas de 30 minutos.

### 3. Verificar el Estado de las VMs
Una vez completado el comando anterior, se puede verificar que todas las VMs estén en funcionamiento con:
```
vagrant status
```
![](/arch-img/running.PNG)
### 4. Acceder a los Servicios
Cada microservicio se ejecutará en una dirección IP privada en la red local:

* Redis: ```192.168.56.10:6379```
* Users API: ```192.168.56.11:8083```
* Auth API: ```192.168.56.12:8000```
* Log Message Processor: ```192.168.56.13```
* Todos API: ```192.168.56.14:8082```
* Frontend: ```192.168.56.15:8080```
* cAdvisor: ```192.168.56.16:8081```
* Prometheus: ```192.168.56.17:9090```
* Grafana: ```192.168.56.18:3000```

### 5. Apagar las VMs

```
vagrant halt
```

### 6. Detruir las VMs

```
vagrant destroy -f
```

**Evidencias:**
![](/arch-img/vagrantFrontend.PNG)
![](/arch-img/VMs.PNG)