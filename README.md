

# Introducción

Solucion de la prueba tecnica para Craftech.io.

## Prueba 1

![Prueba 1 chart de ejemplo](https://raw.githubusercontent.com/axelluguercio/technical_interview/main/arquitecture.png)

El chart muestra un kluster de kubernetes GKE ofrecido por GCP para lo que es entre IaaS e IaaP.
GKE se va a encargar de que el cluster sea de alta dispinibilidad. Nos preocupamos por provicionar los deployment y objetos de kubernetes necesarios para levantar nuestra aplicación.

### Ingress controller

Para acceder a nuestra applicación desde internet, proporcionamos un objeto ingress para hacer un proxy pass desde el puerto http/https al puerto donde esta escuchando cada servicio de nuestra app.

### Frontend js

El ingress controller va a redireccionar el trafico hacia el servicio de frontend, el frontend va a empezar a renderizar los template cargados de los apropiados servicios de base de datos. Cada POD se va a comunicar con el otro mediante su correspondiente servicio.

### Backend MongoDB

Proporciona base de datos no relacional. Las credenciales se almacena en el secret correspondiente.

### Backend PostgresSQL

Proporciona base de datos relacional. Usa configmap y los secrets para tomar las variables de entorno y credenciales.

Cada deployment va a tener su replicaSet, esto quiere decir que por ejemplo cuando haya mucha carga de trabajo en el POD A el deployment de frontend se va a encagar de aumentar su replicar para balancear adecuadamente la carga.

## Prueba 2

La aplicacíon consta de un backend en Django y un frontend en reactjs. Para la base de datos se utiliza PostgresSQL.

Para compilar e desplegar la aplicación se necesita de minikube para cluster local o una instancia de kubernetes en cualquier cloud publica y docker instalado localmente.

- [Para instalar minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Kubernetes documention](https://kubernetes.io/es/)
- [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine?hl=es/)
- [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/)
- [Para instalar docker](https://docs.docker.com/)

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

Desplega el deployment de PostgreSQL 

```
kubectl apply -f k8s/postgresql/.

```

Desplega el deployment de Django

```
kubectl apply -f k8s/django/.

```

Desplega el deployment de reactjs

```
kubectl apply -f k8s/reactjs/.

```

Crear el ingress controller para el proxy pass

```
kubectl apply -f 00-ingress-controlle.yaml

```

Verificar que los deployment y los Pods esten corriendo

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

Si nuestro cluster esta en la nube. Podemos ver el external IP address que nos proporciona el servicio de ingress y acceder a http://<ingress-ip>/ o http://<ingress-ip>/admin.




