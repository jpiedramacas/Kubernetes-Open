
## Introducción a Persistent Volume (PV) y Persistent Volume Claim (PVC) en Kubernetes

### Descripción General

En Kubernetes, los volúmenes persistentes (Persistent Volumes, PV) y las reclamaciones de volúmenes persistentes (Persistent Volume Claims, PVC) son utilizados para proporcionar almacenamiento duradero a los pods. Este README explicará los conceptos de PV y PVC y detallará cómo se aplican en los despliegues de MySQL y WordPress.

### ¿Qué es un Persistent Volume (PV)?

Un **Persistent Volume (PV)** es un recurso de almacenamiento en el clúster de Kubernetes que ha sido aprovisionado por un administrador. Representa una pieza de almacenamiento físico, como un disco en un servidor o un volumen en un servicio de almacenamiento en la nube. Los PV son independientes del ciclo de vida de los pods que los usan.

### ¿Qué es un Persistent Volume Claim (PVC)?

Una **Persistent Volume Claim (PVC)** es una petición de almacenamiento por parte de un usuario. Un PVC permite que los pods soliciten almacenamiento persistente sin conocer los detalles del almacenamiento físico subyacente. Kubernetes se encarga de enlazar una PVC con un PV que cumpla con los requisitos especificados (como capacidad y modos de acceso).

### Estructura del Proyecto

En este proyecto, utilizamos tres archivos YAML principales:

1. `kustomization.yaml`
2. `mysql-deployment.yaml`
3. `wordpress-deployment.yaml`

### Descripción de los Archivos

#### kustomization.yaml

El archivo `kustomization.yaml` se usa para agrupar y gestionar múltiples recursos de Kubernetes de manera cohesiva. Este archivo permite definir un conjunto de recursos que pueden aplicarse conjuntamente.

```yaml
secretGenerator:
- name: mysql-pass
  literals:
  - password=mysql1234
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml
```

#### mysql-deployment.yaml

Este archivo define el servicio, volumen persistente, reclamación de volumen persistente y el despliegue para MySQL.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-claim
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 5Gi
  hostPath:
    path: /data/pv0001/
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
  volumeName: task-pv-claim
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
          claimName: task-pv-claim
```

#### wordpress-deployment.yaml

Este archivo define el servicio, volumen persistente, reclamación de volumen persistente y el despliegue para WordPress.

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
kind: PersistentVolume
metadata:
  name: task-pv-claim2
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: /data/pv0002/
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim2
  labels:
    app: wordpress
spec:
  storageClassName: ""
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  volumeName: task-pv-claim2
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
          claimName: task-pv-claim2
```

### Aplicación de la Configuración

Para desplegar estas configuraciones en su clúster de Kubernetes, siga estos pasos:

1. Asegúrese de que los archivos `kustomization.yaml`, `mysql-deployment.yaml`, y `wordpress-deployment.yaml` estén en el mismo directorio.

2. Ejecute el siguiente comando para aplicar todos los recursos definidos en `kustomization.yaml`:

```sh
kubectl apply -k .
```

3. Verifique que los recursos se han creado correctamente con los siguientes comandos:

```sh
kubectl get svc
kubectl get pv
kubectl get pvc
kubectl get deployments
kubectl get pods
```

### Conclusión

Este proyecto configura y despliega un entorno de WordPress con una base de datos MySQL en un clúster de Kubernetes, utilizando volúmenes persistentes para asegurar que los datos de la base de datos y del sitio web persistan a través de ciclos de vida de los contenedores. Al usar PV y PVC, nos aseguramos de que los datos importantes no se pierdan cuando los contenedores se reinicien o reprogramen.
