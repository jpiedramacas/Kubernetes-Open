
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
kubectl get pods
kubectl get deployments
kubectl get pvc
```

### Explicación de los Archivos YAML

#### persistent-volume.yaml

Este archivo define un Persistent Volume (PV) en Kubernetes. Un PV es una unidad de almacenamiento que ha sido aprovisionada por un administrador. Es independiente del ciclo de vida de los pods y representa una pieza de almacenamiento en el clúster.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

**Componentes del archivo:**

- `apiVersion: v1`: Especifica la versión de la API de Kubernetes que se está utilizando.
- `kind: PersistentVolume`: Indica que se está definiendo un Persistent Volume.
- `metadata`: Información sobre el PV, como su nombre.
  - `name: pv-demo`: Nombre del PV.
- `spec`: Especificaciones del PV.
  - `capacity`: Define la capacidad del volumen.
    - `storage: 1Gi`: Establece el tamaño del volumen a 1 GiB.
  - `accessModes`: Define cómo puede ser accedido el volumen.
    - `ReadWriteOnce`: El volumen puede ser montado como lectura-escritura por un solo nodo a la vez.
  - `hostPath`: Define la ruta en el host donde se almacena el volumen.
    - `path: "/mnt/data"`: Especifica la ruta en el sistema de archivos del host.

#### persistent-volume-claim.yaml

Este archivo define una Persistent Volume Claim (PVC) en Kubernetes. Una PVC es una petición de almacenamiento por parte de un usuario que especifica cuánta capacidad necesita y qué modos de acceso se requieren. Kubernetes enlaza esta PVC con un PV que cumpla con las características solicitadas.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

**Componentes del archivo:**

- `apiVersion: v1`: Especifica la versión de la API de Kubernetes que se está utilizando.
- `kind: PersistentVolumeClaim`: Indica que se está definiendo una Persistent Volume Claim.
- `metadata`: Información sobre la PVC, como su nombre.
  - `name: pvc-demo`: Nombre de la PVC.
- `spec`: Especificaciones de la PVC.
  - `accessModes`: Define cómo puede ser accedido el volumen solicitado.
    - `ReadWriteOnce`: El volumen puede ser montado como lectura-escritura por un solo nodo a la vez.
  - `resources`: Especifica los recursos solicitados.
    - `requests`: Define las solicitudes de recursos.
      - `storage: 1Gi`: Solicita un volumen de 1 GiB de tamaño.

### ¿Cómo Afectan Estos Archivos a los Despliegues de MySQL y WordPress?

Cuando se despliegan aplicaciones como MySQL y WordPress en Kubernetes, necesitan almacenamiento persistente para guardar sus datos. Esto es crucial porque los contenedores son efímeros por naturaleza, y cualquier dato almacenado solo dentro de un contenedor se perdería si el contenedor se elimina o se reinicia.

#### Para MySQL:

- **Persistent Volume (`task-pv-claim` en `mysql-deployment.yaml`)**:
  - Define un volumen persistente de 5GiB en `/data/pv0001/` del host.
- **Persistent Volume Claim (`task-pv-claim` en `mysql-deployment.yaml`)**:
  - Reclama 3GiB del volumen persistente para que el contenedor de MySQL lo use.
- **Deployment de MySQL**:
  - Monta el volumen persistente en `/var/lib/mysql`, asegurando que los datos de la base de datos se almacenen de manera persistente.

#### Para WordPress:

- **Persistent Volume (`task-pv-claim2` en `wordpress-deployment.yaml`)**:
  - Define un volumen persistente de 10GiB en `/data/pv0002/` del host.
- **Persistent Volume Claim (`task-pv-claim2` en `wordpress-deployment.yaml`)**:
  - Reclama 10GiB del volumen persistente para que el contenedor de WordPress lo use.
- **Deployment de WordPress**:
  - Monta el volumen persistente en `/var/www/html`, asegurando que los datos del sitio web se almacenen de manera persistente.

En resumen, los archivos `persistent-volume.yaml` y `persistent-volume-claim.yaml` proporcionan la infraestructura necesaria para el almacenamiento persistente, asegurando que los datos críticos no se pierdan cuando los contenedores de MySQL y WordPress se reinicien o reprogramen.

### Conclusión

Este proyecto configura y despliega un entorno de WordPress con una base de datos MySQL en un clúster de Kubernetes, utilizando volúmenes persistentes para asegurar que los datos de la base de datos y del sitio web persistan a través de ciclos de vida de los contenedores. Al usar PV y PVC, nos aseguramos de que los datos importantes no se pierdan cuando los contenedores se reinicien o reprogramen.
