#Instrucciones

#App

  - Para crear la imagen de la app primero se debe realizar un { `docker build -t <Nombre de la imagen que quieras>  app/` } donde está el Dockerfile y se creará la imagen, si haces esto recuerda que el docker-compose se tendrá que editar.

#Docker-compose

  - Para utilizar el docker-compose, que te creará dos contenedores la app y base de datos persistente, solo se debe escribir { `docker-compose up -d` } y se generaran los dos contenedores anteriormente dichos.


#KUBERNETES

  - En kubernetes está hecho para que funcione en minikube y en gke cada una distribuida en una carpeta con su nombre.

  - Versiones utilizadas:
    - `Minikube`: v1.21.2
    - `GKE`: 1.20.9-gke.701

#Para usar en minikube

  - ¡¡¡NECESARIO INSTALAR Y ARRANCAR UN CLUSTER EN MINIKUBE!!! (https://minikube.sigs.k8s.io/docs/start/)

  - Una vez instalado y arrancado minikube lo único que hay que hacer es poner el siguiente comando: { `kubectl apply -f kubernetes-resources/minikube` } y se te instalara la app y la base de datos con un volumen persistente, dejando un mensaje de lo que está creando.

```
$ kubectl apply -f kubernetes-resources/minikube
deployment.apps/app-deployment created
service/app-service-np created
configmap/config-names created
secret/mysecret created
service/dbserver created
deployment.apps/mysql created
persistentvolume/mysql-pv-volume created
persistentvolumeclaim/mysql-pv-claim created
```

  - Descripción de recursos creados:
    - `mysecret`: Secret con las passwords necesarias. Usado tanto por la aplicación app-deployment como la base de datos mysql
    - `config-names`: ConfigMap con los parámetros de configuración para la base de datos usados tanto por la aplicación como la base de datos.
    - `app-deployment`: Deployment con la aplicación NodeJS
    - `mysql`: Deployment de mysql 5.1
    - `app-service-np`: Servicio tipo NodePort para exponer la aplicación en los nodos de Kubernetes
    - `mysql-pv-volume`: Persistent Volumen de tipo hostpath
    - `mysql-pv-claim`: PersistentVolumeClaim que será asociado al volumen creado anteriormente.
    - `dbserver`: Servicio tipo ClusterIP que apunta a los pods del mysql. Utilizado por la aplicación para acceder a la base de datos.

  - Para saber como entrar a la app hay que saber la dirección IP con { `kubectl get nodes -o=wide` } y la IP estará en la INTERNAL-IP y el puerto asignado al servicio NodePort creado anteriormente con { `kubectl get service app-service-np` }.

  - Para desinstalar solo hay que poner: { `kubectl delete -f kubernetes-resources/minikube` }, ¡¡¡ATENCION!!! El persistent volume crea un fichero en el nodo del minikube ("/mnt/data") para que los datos no se pierdan, si se quiere borrar el fichero hay que entrar al minikube { `docker exec -it (id del contenedor) bash` } y { `rm -r /mnt/data` }.

#Para usar en google cloud (GKE)

  - Hay que crear un clúster en google cloud, recomendado con un mínimo de 2 nodos, serie E2 y tipo de máquina e2-small y tamaño de disco 20 GB.

  - Una vez creado el clúster poner: { `gcloud container clusters get-credentials (nombre del clúster) --zone (zona donde hayas especificado) --project (nombre del proyecto)` } y poner { `kubectl config use-context (nombre del contexto del gke)` } para conectarte al clúster.

  - Para instalar hay que poner: { `kubectl apply -f kubernetes-resources/gke` } y te saldrá un mensaje que estará creando todo.
```
deployment.apps/app-deployment created
service/app-service-cl created
Warning: networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.networking.k8s.io/my-ingress created
configmap/config-names created
secret/mysecret created
service/dbserver created
service/mysql-headless created
statefulset.apps/mysql-statefulset created
```

  - Descripción de recursos creados:
    - `mysecret`: Secret con las passwords necesarias. Usado tanto por la aplicación app-deployment como la base de datos mysql
    - `config-names`: ConfigMap con los parámetros de configuración para la base de datos usados tanto por la aplicación como la base de datos.
    - `app-deployment`: Deployment con la aplicación NodeJS.
    - `mysql`: StatefulSet de mysql 5.1 y el volumen se creará automáticamente como un disco en google cloud al crear el PVC.
    - `mysql-headless`: Servicio de tipo Headless (clusterIP: None) necesario para el funcionamiento del statefulset.
    - `app-service-cl`: Servicio tipo ClusterIP para exponer la aplicación internamente. Será utilizado por el ingress.
    - `dbserver`: Servicio tipo ClusterIP que apunta a los pods del mysql. Utilizado por la aplicación para acceder a la base de datos.
    - `my-ingress`: Ingress que utiliza el controlador por defecto de GKE para exponer la aplicación, y creará automáticamente un Load Balancer en google cloud.

  - Para probar la app hay que saber la ip, al estar hecho con ingress solo hay que saber la dirección IP que te ha creada poniendo: { `kubectl get ingress my-ingress` } y poner en el navegador la ADDRESS, ¡¡¡ATENCION!!! tarda varios minutos el que puedas conectarte a la app.

  - Para desinstalar hay que poner { `kubectl apply -f kubernetes-resources/gke` } y se borraran todos los pods pero el disco del mysql no ya que es persistente y mantendrá los datos que hayas guardado, para borrar el disco hay que poner: { `kubectl get pvc` } te mostrará el disco y después poner: { `kubectl delete pvc mysql-data-mysql-statefulset-0` } para eliminar completamente el disco y los datos que tenga. Si se borrase el clúster pero no borrásemos el disco ese disco quedaría huérfano y se podría borrar desde la google cloud y si se volviera a crear un clúster nuevo el disco huérfano no se conectaría a los nuevos pods el apply te crearía un nuevo disco para la base de datos del mysql.

#Info

  - El puerto que escucha la app es el 3000.

  - Mysql se despliega en miikube como un deployment y en GKE como un statefulset ya que quería experimentar con los dos tipo de recursos.

  - Para usar la app que he hecho en GKE la dirección IP es http://34.149.132.12/ realizada con ingress.
