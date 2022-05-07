# Introducción

Solucion de la prueba tecnica para Craftech.io.

---

## Prueba 1

![Prueba 1 diagrama de red](https://raw.githubusercontent.com/axelluguercio/technical_interview/main/prueba1/chart.png)

---

El diagrama de red muestra una arquitectura de cargas variable y alta disponibilidad utilizando el servicio de **Elastic Container Service**, **RDS** y **DynamoBD** para base de datos relacional y no relacional respectivamente en **AWS**.

---

### *VPC*

- ***10.0.1.0/20*** -> **IP** de la VPC.

- ***15*** -> *subredes diponibles*.

- ***10.0.2.0/27 - 10.0.4.0/27*** -> para **subredes privadas** con 29 ip reservadas para cada una.

- ***10.0.5.0/30 - 10.0.4.0/30*** -> para **subredes publicas** con 1 ip reservada para NAT gateway.

---

### *Alta Disponibilidad*

> La **VPC** esta deployada en 3 zonas geograficas para ofrecer alta disponibilidad y que se puedan repartir las cargas de trabajo.

---

### *Route 53*

> Ofrece un servicio administrado de **DNS** para que los usuarios puedan acceder a la applicación con un dominio sin tener que recordarse una ip.

---

### *Application Load Balancer*

> Se encarga de  mejorar la carga que recibe la aplicación balanceando el trafico en las 3 zonas disponibles.

---

### *Internet Gateway*

> Brinda inbound/outbound hacia internet.

---

### *NAT gateway*

> El backend consume 2 servicios externos por lo que necesita acceder a internet, NAT gateway ofrece rutear el trafico desde las subredes privadas hacia internet de manera segura.

---

### *ECS*

> Proporciona servicio administrado y escalable de contenedores dentro estan deployado el frontend y el backend.

---

### *RDS*

> Servicio administrado para la base de datos relacional, facilmente escalable.

### *DynamoBD*

> Para alojar la base de datos no relacional.

---

## Prueba 2

La aplicacíon consta de un backend en Django y un frontend en reactjs. Para la base de datos se utiliza PostgresSQL.

Para compilar e desplegar la aplicación se necesita de **minikube** para cluster local o una instancia de kubernetes en cualquier cloud publica.

- [Para instalar minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Kubernetes documention](https://kubernetes.io/es/)
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine?hl=es/)
- [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/)

#### Para minikube

```
# Inicia el cluster de minikube como nested vm
minikube start --driver=none

# Lista la configuracion actual con la que inicia
minikube show config

```

#### Para kubernetes en AWS o GCP

```
# hay que copiar el archivo de kubeconfig y regenerar ese archivo a nivel local para kubectl
cat ~/.kube/config

```

#### Desplega el deployment de PostgreSQL 

```
kubectl apply -f k8s/postgresql/.

```

#### Desplega el deployment de Django

```
kubectl apply -f k8s/django/.

```

#### Desplega el deployment de reactjs

```
kubectl apply -f k8s/reactjs/.

```

#### Crear el ingress controller para el proxy pass

```
kubectl apply -f 00-ingress-controlle.yaml

```

#### Verificar que los deployment y los Pods esten corriendo

```
kubectl get --all

# Para ver el progeso del pod
kubectl describe pod <Pod name>

```

Para acceder a nuestra aplicación primero hay que ver nuestra ip para acceder desde afuera, en el caso de tener minikube podemos ver la ip de minikube y abrir los puertos nuestro router para poder acceder desde internet.

```
minikube ip

Example: 192.168.49.2

Abrir puerto http de nuestro router para que haga un port-forward a 192.168.49.2 (la ip de minikube local).

```

Si nuestro cluster esta en la nube. Podemos ver el external IP address que nos proporciona el servicio de ingress y acceder por:

- [frontend en reactjs](http://<ingress-ip>/)
- [backend en django](http://<ingress-ip>/admin)

### docker-compose

Para desplegar la app para desarrollo podemos levantar el archivo de docker compose, para esto es necesario tener instalado docker e docker-compse.

- [Para instalar docker](https://docs.docker.com/)

```
# Levantar la app desde el archivo docker-compose.yaml (Situarse en la misma ubicacion del archivo)
docker-compose up -vv .

```

## Prueba 3

Para CI/CD, deploye una imagen de nginx:latest y copie el index.html dentro del dir html.
Cree una carpeta .github/workflows para las pipelines de GitHub Actions.

Para la parte de credenciales agregue los secret de mi DOCKER_PASS y DOCKER_USER.

Para que Github actions pueda deployar la imagen en mi cluster,
(como en mi caso tenia minikube) tuve que abrir el puerto de la API de kubernetes y crear una regla de "port-forwarding" que redireccione el trafico a mi maquina que aloja la vm de minikube.

Por ultimo agregue un secret llamado KUBE_CONFIG_DATA que almacena la salida de mi archivo kubeconfig en base 64 donde reemplazando la ip local con la ip publica.

```
# code base64 del archivo kubeconfig (para minikube o GKE)
cat ~/.kube/config | base64

```

#### El cicd.yaml consta de 2 jobs:

- El job docker lo que hace es que cada vez que hago un push a la rama main regenera la imagen de nginx y lo sube a mi repositorio de DOCKER_HUB.
```
jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      -
        name: Set up QEMU # Setea la maquina virtual
        uses: docker/setup-qemu-action@v2
      -
        name: Set up Docker Buildx 
        uses: docker/setup-buildx-action@v2
      -
        name: Login to DockerHub # Login a Docker Hub usando los secrets 
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}
      -
        name: Checkout
        uses: actions/checkout@v1
      -
        name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: ./prueba3
          file: ./prueba3/Dockerfile         # dockerfile path para buildear
          push: true
          tags: 'aluguercio/nginx-ci:latest' # metadata latest para ultima version

```

- El ultimo jobs toma mi kubeconfig para conectarse a mi cluster de minikube y inyecta la nueva imagen en el deployment de nginx, por ultimo revisa si el deployment esta corriendo correctamente.

```
  deploy-to-cluster:
    name: deploy to cluster
    runs-on: ubuntu-latest
    needs: docker # solo si el job docker es successfull
    steps:
      - 
        name: deploy to cluster
        uses: steebchen/kubectl@master
        env:
          KUBE_CONFIG_DATA: ${{ secrets.KUBE_CONFIG_DATA }} # el archivo de kubeconfig de kubectl en base64 para poder conectarse al cluster remoto
        with:
          args: set image --record deployment/nginx-deployment nginx=aluguercio/nginx-ci:latest # Cambia la imagen del container nginx deployado en el cluster por la nueva que se pusheo al repo
      - 
        name: verify deployment
        uses: steebchen/kubectl@master
        env:
          KUBE_CONFIG_DATA: ${{ secrets.KUBE_CONFIG_DATA }}
          KUBECTL_VERSION: "1.20"
        with:
          args: '"rollout status deployment/nginx-deployment"' # verifica que el deployment de nginx este corriendo

```




