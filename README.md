# Despliegue de WordPress y MySQL en Kubernetes

Este tutorial te muestra cómo desplegar un sitio de WordPress y una base de datos MySQL utilizando Minikube en Kubernetes. Ambas aplicaciones utilizan PersistentVolumes y PersistentVolumeClaims para almacenar datos.

Un PersistentVolume (PV) es un fragmento de almacenamiento en el clúster que ha sido provisionado manualmente por un administrador o provisionado dinámicamente por Kubernetes utilizando un StorageClass. Un PersistentVolumeClaim (PVC) es una solicitud de almacenamiento realizada por un usuario que puede ser satisfecha por un PV. Los PersistentVolumes y PersistentVolumeClaims son independientes de los ciclos de vida de los Pods y conservan los datos a través de reinicios, reprogramaciones e incluso eliminaciones de Pods.

## Objetivos

1. Crear PersistentVolumeClaims y PersistentVolumes.
2. Crear un kustomization.yaml con
   - Un generador de Secret
   - Configuraciones de recursos de MySQL
   - Configuraciones de recursos de WordPress
3. Aplicar el directorio kustomization mediante `kubectl apply -k ./`.
4. Limpieza.

## Antes de comenzar

Para verificar la versión, introduce el comando `kubectl version`. El ejemplo mostrado en esta página funciona con kubectl 1.27 y versiones posteriores.

Descarga los siguientes archivos de configuración:

- [mysql-deployment.yaml](https://k8s.io/examples/application/wordpress/mysql-deployment.yaml)
- [wordpress-deployment.yaml](https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml)

## Crear un kustomization.yaml

Añade un generador de Secret al kustomization.yaml. Un Secret es un objeto que almacena datos sensibles como una contraseña o clave. Desde la versión 1.14, kubectl admite la gestión de objetos de Kubernetes utilizando un archivo kustomization. Puedes crear un Secret mediante generadores en kustomization.yaml.

**Archivo:** `kustomization.yaml`

```bash
secretGenerator:
- name: mysql-pass
  literals:
  - password=mysql1234
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml
```

## Agregar configuraciones de recursos para MySQL y WordPress

El siguiente manifiesto describe un Deployment de MySQL de instancia única. El contenedor MySQL monta el PersistentVolume en /var/lib/mysql. La variable de entorno MYSQL_ROOT_PASSWORD establece la contraseña de la base de datos desde el Secret.

**Archivo:** `mysql-deployment.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql-service
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:8.0
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        - name: MYSQL_DATABASE
          value: wordpress
        - name: MYSQL_USER
          value: wordpress
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

Verificamos 

```bash
kubectl apply -f .\mysql-deployment.yaml
```

El siguiente manifiesto describe un Deployment de WordPress de instancia única. El contenedor de WordPress monta el PersistentVolume en /var/www/html para los archivos de datos del sitio web. La variable de entorno WORDPRESS_DB_HOST establece el nombre del Servicio MySQL definido anteriormente, y WordPress accederá a la base de datos a través del Servicio. La variable de entorno WORDPRESS_DB_PASSWORD establece la contraseña de la base de datos desde el Secret generado por kustomize.

**Archivo:** `wordpress-deployment.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:6.2.1-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        - name: WORDPRESS_DB_USER
          value: wordpress
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

Verificamos 

```bash
kubectl apply -f .\wordpress-deployment.yaml
```

## Agregar recursos al archivo kustomization.yaml

Agrégalos al archivo kustomization.yaml.

```bash
secretGenerator:
- name: mysql-pass
  literals:
  - password=mysql1234
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml
```

## Aplicar y verificar

El archivo `kustomization.yaml` contiene todos los recursos para desplegar un sitio de WordPress y una base de datos MySQL. Puedes aplicar el directorio con el siguiente comando:

```bash
kubectl apply -k ./
```

Ahora puedes verificar que todos los objetos existen.

Verifica que el Secret existe ejecutando el siguiente comando:

```bash
kubectl get secrets
```

La respuesta debería ser similar a esta:

```
NAME                    TYPE                                  DATA   AGE
mysql-pass-c57bb4t7mf   Opaque                                1      9s
```

Verifica que un PersistentVolume se haya provisionado dinámicamente:

```bash
kubectl get pvc
```

**Nota:** Puede tardar unos minutos en que los PVs se provisionen y se vinculen.

La respuesta debería ser similar a esta:

```
NAME             STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
mysql-pv-claim   Bound     pvc-8cbd7b2e-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
wp-pv-claim      Bound     pvc-8cd0df54-4044-11e9-b2bb-42010a800002   20Gi       RWO            standard           77s
```

Verifica que el Pod esté en ejecución ejecutando el siguiente comando:

```bash
kubectl get pods
```

**Nota:** Puede tardar unos minutos en que el estado del Pod sea RUNNING.

La respuesta debería ser similar a esta:

```
NAME                               READY     STATUS    RESTARTS   AGE
wordpress-mysql-1894417608-x5dzt   1/1       Running   0          40s
```

Verifica que el Servicio esté en ejecución ejecutando el siguiente comando:

```bash
kubectl get services wordpress
```

La respuesta debería ser similar a esta:

```
NAME        TYPE            CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
wordpress   LoadBalancer    10.0.0.89    <pending>     80:32406/TCP   4m
```

**Nota:** Minikube solo puede exponer Servicios a través de NodePort. El EXTERNAL-IP siempre está pendiente.

Ejecuta el siguiente comando para obtener la dirección IP del Servicio de WordPress:

```bash
minikube service wordpress 
```

La respuesta debería ser similar a esta:

```
http://1.2.3.4:32406
```

## Limpieza

Ejecuta el siguiente comando para eliminar tu Secret, Deployments, Services y PersistentVolumeClaims:

```bash
kubectl delete -k ./
```

## Exploración de Volúmenes Persistentes en Kubernetes con Minikube

Proporciona los comandos y pasos necesarios para explorar y verificar los volúmenes persistentes utilizados por WordPress y MySQL en un entorno Kubernetes usando Minikube.

### Comandos Utilizados

#### 1. Listar PersistentVolumeClaims (PVC)
Para listar todos los PersistentVolumeClaims en el clúster:

```bash
kubectl get pvc
```

Salida esperada:

```
NAME             STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
mysql-pv-claim   Bound    pvc-1134154a-3e57-40ab-9bd2-1939c6fb4a72   20Gi       RWO            standard       <unset>                 79m
wp-pv-claim      Bound    pvc-500bb251-7b41-4fb1-8f1e-e3a747ad4caa   20Gi       RWO            standard       <unset>                 78m
```

#### 2. Acceder a Minikube a través de SSH
Para acceder a la instancia de Minikube:

```bash
minikube ssh
```

Nota: Puedes ver una advertencia relacionada con la resolución del contexto Docker CLI, pero no afecta el acceso SSH.

#### 3. Verificar la existencia de los PVC en el sistema de archivos
Una vez dentro de la instancia de Minikube, puedes listar los archivos y directorios en tu directorio home para confirmar que estás en el entorno correcto:

```bash
ls -la
```

#### 4. Navegar a los directorios de los Pods
Para acceder a los directorios de los pods en el sistema de archivos de Minikube, primero necesitas permisos de superusuario:

```bash
sudo su
cd /var/lib/kubelet/pods
```
Esto listará los directorios correspondientes a cada pod, identificados por sus UUIDs.

#### 5. Verificar los directorios de los Pods
Para ver los detalles de los directorios de los Pods:

```bash
ls -la
```

Salida esperada:

```bash
total 44
drwxr-x--- 11 root root 4096 May 29 15:46 .
drwx------  9 root root 4096 May 29 15:08 ..
drwxr-x---  5 root root 4096 May 29 15:08 063d6b9688927e601f52fd818d1305c5
drwxr-x---  5 root root 4096 May 29 15:08 3c555f828409b009ebee39fdbedfcac0
drwxr-x---  5 root root 4096 May 29 15:09 5d2a7ca1-e020-456c-a9c3-6a02422ef9a3
drwxr-x---  5 root root 4096 May 29 15:08 7fd44e8d11c3e0ffe6b1825e2a1f2270
drwxr-x---  5 root root 4096 May 29 15:09 999015e5-3033-4576-84e5-a3002c1a5d5e
drwxr-x---  5 root root 4096 May 29 15:09 dbba026e-ffc0-4f9e-802e-649bc7afdc71
drwxr-x---  5 root root 4096 May 29 15:46 e54dd7bd-7c2c-4a81-8c42-703ff039caa5
drwxr-x---  5 root root 4096 May 29 15:46 f48d2937-02cb-4968-8c05-cb48713a642c
drwxr-x---  5 root root 4096 May 29 15:08 f9c8e1d0d74b1727abdb4b4a31d3a7c1
```

### Conclusión
Siguiendo estos pasos, puedes verificar y explorar los volúmenes persistentes asociados a tus despliegues de WordPress y MySQL en Kubernetes utilizando Minikube. Esto te permitirá asegurarte de que tus datos se almacenan de manera persistente y correcta en el clúster.
