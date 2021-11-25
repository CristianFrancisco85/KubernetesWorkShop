# Kubernetes Workshop

## Crear Cluster
Para este paso tenemos la opcion de usar la terminal Cloud Shell que nos proporciona GCP con todas la herramientas instaladas o instalar Cloud SDK de manera local.



### Iniciar Cluster
```
gcloud container clusters create kcd-cluster --num-nodes=3 --region=us-central1-a --project={PROJECT-ID}
```
### Obtener credenciales
Este paso solo es necesario si trabajamos de manera local
```
gcloud container clusters get-credentials kcd-cluster --zone us-central1-a --project {PROJECT-ID}
```
### Visualizar Nodos
```
kubectl get nodes -o wide
```

## Crear Namespace
Un namespace es una manera de abstraer secciones de nuestro cluster, con esto conseguimos organizar nuestros recursos
```
kubectl create namespace kcd-ns
```

## Pods
Los Pods son las unidades de computación desplegables más pequeñas que se pueden crear y gestionar en Kubernetes.

### Creando Pod
```
kubectl apply -f Pod.yaml -n kcd-ns
```
### Visualizar Pod
```
kubectl get pods -n kcd-ns

NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          2m44s
```
### Eliminar Pod
```
kubectl delete pod nginx-pod -n kcd-ns
```

 
 ## Deployments
Un Deployment proporciona actualizaciones declarativas para los Pods.
Cuando se describe el estado deseado con un Deployment, el controlador del Deployment se encarga de cambiar el estado actual al estado deseado de forma controlada.  
### Creando Deployment
```
kubectl apply -f Deployment.yaml -n kcd-ns
```
### Visualizar Pods
Se crearan dos Pods ya que en la configuracion de replicas indicamos que deseamos dos
```
kubectl get pods -n kcd-ns

NAME                               READY   STATUS    RESTARTS   AGE  
nginx-deployment-8b4556b7d-lplfr   1/1     Running   0          2m53s
nginx-deployment-8b4556b7d-sr7ks   1/1     Running   0          2m53s
```
### Eliminar Pod
Al eliminar un pod de un deployment veremos como se crea uno nuevo debido a la configuracion de replicas
```
kubectl delete pod nginx-pod -n kcd-ns
```
### Eliminar Deployment
```
kubectl delete -f Deployment.yaml -n kcd-ns
```
 ## DaemonSets
Un DaemonSet garantiza que todos de los nodos ejecuten una copia de un Pod. Conforme se añade más nodos al clúster, nuevos Pods son añadidos a los mismos. Conforme se elimina nodos del clúster, dichos Pods se destruyen. Al eliminar un DaemonSet se limpian todos los Pods que han sido creados.
### Creando DaemonSet
```
kubectl apply -f DaemonSet.yaml -n kcd-ns
```
### Visualizar Pods
Se crearan un pod por cada nodo en nuestro cluster
```
kubectl get pods -n kcd-ns

NAME                    READY   STATUS    RESTARTS   AGE   
nginx-daemonset-478nt   1/1     Running   0          64s   
nginx-daemonset-jqlfn   1/1     Running   0          64s   
nginx-daemonset-wn4h9   1/1     Running   0          64s

```
### Eliminar Pod
Al eliminar un pod de un deployment veremos como se crea uno nuevo para cumplir con el DaemonSet
```
kubectl delete pod nginx-pod -n kcd-ns
```
### Eliminar DaemonSet
```
kubectl delete -f DaemonSet.yaml -n kcd-ns
```

## StatefulSets
Un StatefulSet se usa para gestionar aplicaciones con estado.Cada uno de los Pods creados tiene su propio identificador persistente que mantiene a lo largo de cualquier re-programación. Su uso es ideal para cuando se necesite almacenamiento persitente e identificadores de red estables.
### Creando StatefulSet
```
kubectl apply -f StatefulSet.yaml -n kcd-ns
```
### Visualizar Pod
```
kubectl get pods -n kcd-ns

NAME                  READY   STATUS    RESTARTS   AGE     
mysql-statefulset-0   1/1     Running   0          30s
```
### Entrar al shell del container
```
kubectl exec -it mysql-statefulset-0 -n kcd-ns -- /bin/bash
```
#### Creando BD en MySQL
```
root@mysql-statefulset-0:/# mysql -u root -p
Enter password: 
mysql> SHOW DATABASES;
+---------------------+ 
| Database            | 
+---------------------+ 
| information_schema  | 
| #mysql50#lost+found | 
| mysql               | 
| performance_schema  | 
+---------------------+ 

mysql> CREATE DATABASE kcdDataBase;
Query OK, 1 row affected (0.00 sec)

mysql> SHOW DATABASES;
+---------------------+
| Database            |
+---------------------+
| information_schema  |
| kcdDataBase         |
| #mysql50#lost+found |
| mysql               |
| performance_schema  |
+---------------------+
mysql> exit
Bye
root@mysql-statefulset-0:/# exit
exit

```
### Eliminando Pod
Al eliminar el pod se creara otro automaticamente, sin embargo si realizamos los pasos anteriores nos daremos cuenta que la base de datos *kcdDataBase* existe gracias a la persistencia de datos que nos otorga el uso de volumenes
```
kubectl delete pod mysql-statefulset-0 -n kcd-ns
```

## Services 
Los servicios nos proporcionan una capa de abstraccion para poder acceder a los Pods a traves de la red.

### Creando Servicios
```
kubectl apply -f Services.yaml -n kcd-ns
```
### Probando ClusterIP
```
kubectl get service/mysql-clusterip -n kcd-ns

NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
mysql-clusterip   ClusterIP   10.44.4.4    <none>        3306/TCP   22m

kubectl get svc -n kcd-ns
kubectl exec -it ubuntu -n kcd-ns -- /bin/bash
root@ubuntu:/# apt-get update
root@ubuntu:/# apt-get install mysql-client
root@ubuntu:/# mysql -u root -p -hmysql-clusterip
Enter password: 
mysql> SHOW DATABASES;
+---------------------+
| Database            |
+---------------------+
| information_schema  |
| #mysql50#lost+found |
| mysql               |
| performance_schema  |
+---------------------+
mysql> exit
Bye
root@mysql-statefulset-0:/# exit
exit
```
### Probando NodePort
Desde MySQL Workbench en nuestra maquina local podemos conectarnos mediante la IP de cualquiera de nuestros nodos y el puerto especificado en nuestro NodePort
```
kubectl get nodes -n kcd-ns -o wide

NAME                                         STATUS   ROLES    AGE   VERSION            INTERNAL-IP   EXTERNAL-IP                                   
gke-kcd-cluster-default-pool-ae58e577-dv1w   Ready    <none>   41m   v1.21.5-gke.1302   10.128.0.41   146.148.49.234            
gke-kcd-cluster-default-pool-ae58e577-fbt3   Ready    <none>   41m   v1.21.5-gke.1302   10.128.0.39   104.198.169.42              
gke-kcd-cluster-default-pool-ae58e577-jm5s   Ready    <none>   41m   v1.21.5-gke.1302   10.128.0.40   35.223.114.80             

kubectl get service/mysql-nodeport -n kcd-ns

NAME             TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
mysql-nodeport   NodePort   10.44.9.241   <none>        3306:30000/TCP   25m
```

### Probando LoadBalancer
Desde MySQL Workbench en nuestra maquina local podemos conectarnos mediante la IP y puerto especificado en nuestro LoadBalancer
```
kubectl get service/mysql-loadbalancer -n kcd-ns

NAME                 TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)          AGE
mysql-loadbalancer   LoadBalancer   10.44.14.87   35.192.27.126   3306:31100/TCP   30m
```
## Ingress y ConfigMap

### Instalando NGINX Ingress Controller
Estos comandos se ejecutaron desde Cloud Shell
```
kubectl create ns nginx-ingress
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx 
helm repo update 
helm install nginx-ingress ingress-nginx/ingress-nginx -n nginx-ingress
```
### Creando NGINX con ConfigMap y exponiendolo con Ingress
```
kubectl apply -f ConfigMap.yaml -n kcd-ns
```
### Probando Ingress
Mendiante la IP del LoadBalancer podemos tener acceso al Ingress Controller
```
kubectl get services -n nginx-ingress
NAME                                              TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
nginx-ingess-ingress-nginx-controller             LoadBalancer   10.44.14.208   35.232.199.127   80:32331/TCP,443:30643/TCP   50m
nginx-ingess-ingress-nginx-controller-admission   ClusterIP      10.44.10.254   <none>           443/TCP                      50m
```


