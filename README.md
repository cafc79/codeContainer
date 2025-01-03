# Contenedores, Kubernetes y GKE

## 1. Contenedores en Docker
### Construcción de la Imagen
Asegúrate de que el [Dockerfile](https://github.com/cafc79/ms-api-service/blob/main/Dockerfile) está en la raíz del directorio de tu proyecto, al igual que el archivo [.jar](https://github.com/cafc79/ms-api-service/blob/main/target/ms-api-service-0.0.1-SNAPSHOT.jar)
```ruby
FROM registry.access.redhat.com/ubi8/openjdk-11-runtime:1.13-1
WORKDIR /app
COPY ./ms-api-service-0.0.1-SNAPSHOT.jar /app/ms-api-service-0.0.1-SNAPSHOT.jar
CMD java -jar /app/ms-api-service-0.0.1-SNAPSHOT.jar
EXPOSE 8082
```

1. En la terminal, navega al directorio y Construye la Imagen ejecutando:
```ruby
> docker build -t mi-app .
```

> [!NOTE]
> - [ ] -t mi-app imagen: Especifica el nombre de la imagen.
> - [ ] .: Indica que Docker debería buscar el Dockerfile en el directorio actual.
 
### Despliegue de un Contenedor

2. Ejecuta el Contenedor: Crea y ejecuta el contenedor con el siguiente comando:
```ruby
> docker run -d -p 3000:3000 mi-app
```

3. Prueba el Despliegue: Si la aplicación está expuesta en el puerto 3000, abre un navegador o usa curl para verificar:
```ruby
> curl http://localhost:3000
```

> [!WARNING]
> Recuerda que el mapeo de puertos, debe coincidir entre el puerto que expone el contenedor, con el puerto en el cual del host al que se hace referencia en el request.

> [!IMPORTANT]
> EXPOSE 8082


## 2. Contenedores en Google Cloud
Desplegar un contenedor Docker en Google Cloud es un proceso que involucra Google Kubernetes Engine (GKE), Compute Engine, o directamente Google Cloud Run, dependiendo del nivel de control y abstracción que prefieras. A continuación, te describo ampliamente cómo hacerlo utilizando Google Cloud Run (para simplicidad) y Google Kubernetes Engine (GKE) (para mayor control).

### 2.1. Usando Google Cloud Run
Google Cloud Run es un servicio totalmente gestionado que simplifica el despliegue de contenedores. Solo necesitas una imagen Docker y Cloud Run se encarga de la infraestructura.
Crear la Imagen Docker
Desde el directorio del proyecto, crea un archivo Dockerfile
```ruby
FROM registry.access.redhat.com/ubi8/openjdk-11-runtime:1.13-1
WORKDIR /app
COPY ./ms-api-service-0.0.1-SNAPSHOT.jar /app/ms-api-service-0.0.1-SNAPSHOT.jar
CMD java -jar /app/ms-api-service-0.0.1-SNAPSHOT.jar
EXPOSE 8082
```

Construye la imagen Docker:
```ruby
docker build -t gcr.io/[PROJECT_ID]/my-container:v1 .
```

Subir la Imagen a Google Container Registry (GCR)
Configura Docker para autenticarte con GCR:
```ruby
gcloud auth configure-docker
```

Sube la imagen al Container Registry:
```ruby
docker push gcr.io/[PROJECT_ID]/my-container:v1
```

Desplegar el Contenedor en Google Cloud Run
Habilita el servicio de Cloud Run:
```ruby
gcloud services enable run.googleapis.com
```

Despliega el contenedor:
```ruby
gcloud run deploy my-container \
  --image gcr.io/[PROJECT_ID]/my-container:v1 \
  --region [REGION] \
  --platform managed \
  --allow-unauthenticated
```

```
[PROJECT_ID]: ID de tu proyecto de Google Cloud.
[REGION]: Por ejemplo, us-central1.
```

Obtén la URL de tu aplicación:
```ruby
gcloud run services describe my-container --region [REGION] --format="value(status.url)"
```

Ventajas de Cloud Run
Totalmente gestionado (no necesitas administrar infraestructura).
Escala automáticamente según la carga.
Pago por uso (solo pagas cuando el contenedor se ejecuta).
Limitaciones
Menor control sobre la infraestructura comparado con GKE.
Algunas configuraciones avanzadas pueden no estar disponibles.

### 2.2. Usando Google Kubernetes Engine (GKE)
GKE ofrece un control total sobre la ejecución de contenedores en clústeres de Kubernetes.

Autenticarte en Google Cloud:
```ruby
gcloud auth login
gcloud config set project [PROJECT_ID]
```

Crear un Clúster de GKE:
```ruby
gcloud container clusters create [CLUSTER_NAME] \
  --num-nodes=3 \
  --region=[REGION]
```

Configurar kubectl para acceder al clúster:
```ruby
gcloud container clusters get-credentials [CLUSTER_NAME] --region [REGION]
```

Verifica la conectividad:
```ruby
kubectl get nodes
```

Construye y sube la imagen Docker (igual que en Cloud Run):
```ruby
docker build -t gcr.io/[PROJECT_ID]/my-container:v1 .
docker push gcr.io/[PROJECT_ID]/my-container:v1
```

Crea un archivo YAML (por ejemplo, deployment.yaml) para desplegar el contenedor en GKE:
```ruby
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-container
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-container
  template:
    metadata:
      labels:
        app: my-container
    spec:
      containers:
      - name: my-container
        image: gcr.io/[PROJECT_ID]/my-container:v1
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: my-container-service
spec:
  selector:
    app: my-container
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
```

Aplica el manifiesto para crear el despliegue y el servicio:
```ruby
kubectl apply -f deployment.yaml
```

Verifica los pods y servicios:
```ruby
kubectl get pods
kubectl get services
```

Accede a tu aplicación usando la IP externa del servicio LoadBalancer.


## 3.  Kubernetes en GKE
Para desplegar una aplicación en Google Kubernetes Engine (GKE) usando un manifiesto de Kubernetes, necesitas seguir estos pasos clave. Detallo cada etapa desde la preparación del entorno hasta el despliegue:

### 3.1. Configuración Inicial de GKE
#### Autenticarte en Google Cloud Platform (GCP):

Asegúrate de tener configurado el CLI de GCP (gcloud).
Inicia sesión:
```ruby
gcloud auth login
```

Selecciona el proyecto donde desplegarás la aplicación:
```ruby
gcloud config set project [PROJECT_ID]
```

#### 3.2.  Crear un Clúster de GKE:
Configura el clúster de GKE:
```ruby
gcloud container clusters create [CLUSTER_NAME] \
  --num-nodes=3 \
  --region=[REGION]
```

Ejemplo:
```ruby
gcloud container clusters create my-cluster --num-nodes=3 --region=us-central1
```

#### 3.3.  Configurar credenciales locales:

Descarga el archivo de credenciales del clúster para interactuar con Kubernetes:
```ruby
gcloud container clusters get-credentials [CLUSTER_NAME] --region [REGION]
```

Verifica la conectividad al clúster:
```ruby
kubectl get nodes
```

Esto mostrará los nodos del clúster si todo está configurado correctamente.

### 3.4. Preparar el Manifiesto de Kubernetes
Asegúrate de que tu manifiesto YAML esté listo y correctamente configurado. Por ejemplo:

Manifiesto de Despliegue y Servicio
```ruby
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: gcr.io/[PROJECT_ID]/my-image:latest
        ports:
        - containerPort: 80
```
```ruby
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

### 3.5. Construir y Subir la Imagen al Registro de Contenedores
Crear la imagen Docker:
Desde tu directorio de proyecto:
```ruby
docker build -t gcr.io/[PROJECT_ID]/my-image:latest .
```

Subir la imagen al Container Registry:

Asegúrate de autenticar Docker con GCP:
```ruby
gcloud auth configure-docker
```
Sube la imagen:
```ruby
docker push gcr.io/[PROJECT_ID]/my-image:latest
```

### 3.6. Desplegar la Aplicación en GKE
Aplicar el manifiesto:
Usa el comando kubectl apply para desplegar tu aplicación:
```ruby
kubectl apply -f [MANIFEST_FILE].yaml
```

Ejemplo:
```ruby
kubectl apply -f my-app.yaml
```

Verifica el despliegue:
Asegúrate de que los pods estén funcionando:
```ruby
kubectl get pods
```

Revisa el servicio para obtener la dirección IP externa:
```ruby
kubectl get service my-service
```

### 3.7. Probar la Aplicación
Una vez que el servicio esté expuesto, accede a la aplicación desde la dirección IP externa proporcionada por el servicio LoadBalancer.

### 3.8. Limpieza (Opcional)
Si quieres eliminar los recursos desplegados:
```ruby
kubectl delete -f [MANIFEST_FILE].yaml
```
Si no necesitas el clúster de GKE, elimínalo para evitar costos innecesarios:
```ruby
gcloud container clusters delete [CLUSTER_NAME] --region [REGION]
```

### Mejores Prácticas
1. Gestionar Versiones de la Imagen:
Usa etiquetas de versión específicas (my-image:v1.0.0) en lugar de latest para garantizar reproducibilidad.

2. Configurar Recursos y Límites:
Define los recursos de CPU y memoria para los contenedores:
```ruby
resources:
  requests:
    memory: "256Mi"
    cpu: "500m"
  limits:
    memory: "512Mi"
    cpu: "1"
```

3. Probar en Ambientes Locales:
Usa Minikube o Kind para validar el manifiesto antes de desplegar en GKE.

4. Habilitar Autoescalado:
Configura el autoescalador en tu clúster de GKE:
```ruby
gcloud container clusters update [CLUSTER_NAME] \
  --enable-autoscaling \
  --min-nodes=1 \
  --max-nodes=5
```
