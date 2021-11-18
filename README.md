# Kubernetes Workshop

## Crear Cluster
Para este paso tenemos la opcion de usar la terminal Cloud Shell que nos proporciona GCP con todas la herramientas instaladas o instalar Cloud SDK de manera local 

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
### Creando Servicios
```
kubectl apply -f Services.yaml -n kcd-ns
```
