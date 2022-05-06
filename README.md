

# Introducción

Solucion de la prueba tecnica para Craftech.io.

## Prueba 1

![Prueba 1 chart de ejemplo](https://raw.githubusercontent.com/axelluguercio/technical_interview/main/arquitecture.png)

El chart muestra un kluster de kubernetes GKE ofrecido por GCP para lo que es entre IaaS e IaaP.
GKE se va a encargar de que el cluster sea de alta dispinibilidad. Nos preocupamos por provicionar los deployment y objetos de kubernetes necesarios para levantar nuestra aplicación.

#### Ingress controller

Para acceder a nuestra applicación desde internet, proporcionamos un objeto ingress para hacer un proxy pass desde el puerto http/https al puerto donde esta escuchando cada servicio de nuestra app.

#### Frontend js

El ingress controller va a redireccionar el trafico hacia el servicio de frontend, el frontend va a empezar a renderizar los template cargados de los apropiados servicios de base de datos. Cada POD se va a comunicar con el otro mediante su correspondiente servicio.

#### Backend MongoDB

Proporciona base de datos no relacional. Las credenciales se almacena en el secret correspondiente.

#### Backend PostgresSQL

Proporciona base de datos relacional. Usa configmap y los secrets para tomar las variables de entorno y credenciales.

Cada deployment va a tener su replicaSet, esto quiere decir que por ejemplo cuando haya mucha carga de trabajo en el POD A el deployment de frontend se va a encagar de aumentar su replicar para balancear adecuadamente la carga.





