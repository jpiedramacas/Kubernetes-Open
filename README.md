# Despliegue de WordPress y MySQL en Kubernetes

Este tutorial te muestra cómo desplegar un sitio de WordPress y una base de datos MySQL utilizando Minikube en Kubernetes. Ambas aplicaciones utilizan PersistentVolumes y PersistentVolumeClaims para almacenar datos.

Un PersistentVolume (PV) es un fragmento de almacenamiento en el clúster que ha sido provisionado manualmente por un administrador o provisionado dinámicamente por Kubernetes utilizando un StorageClass. Un PersistentVolumeClaim (PVC) es una solicitud de almacenamiento realizada por un usuario que puede ser satisfecha por un PV. Los PersistentVolumes y PersistentVolumeClaims son independientes de los ciclos de vida de los Pods y conservan los datos a través de reinicios, reprogramaciones e incluso eliminaciones de Pods.

**Advertencia:** Este despliegue no es adecuado para casos de uso en producción, ya que utiliza Pods de WordPress y MySQL de instancia única. Considera utilizar el gráfico Helm de WordPress para desplegar WordPress en producción.

**Nota:** Los archivos proporcionados en este tutorial utilizan API de implementación de GA y son específicos de Kubernetes versión 1.9 en adelante. Si deseas utilizar este tutorial con una versión anterior de Kubernetes, actualiza la versión de la API adecuadamente o consulta versiones anteriores de este tutorial.

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

## Crear PersistentVolumeClaims y PersistentVolumes

MySQL y WordPress requieren cada uno un PersistentVolume para almacenar datos. Sus PersistentVolumeClaims se crearán en el paso de despliegue.

Muchos entornos de clúster tienen un StorageClass predeterminado instalado. Cuando no se especifica un StorageClass en el PersistentVolumeClaim, se utiliza el StorageClass predeterminado del clúster.

Cuando se crea un PersistentVolumeClaim, se provisiona dinámicamente un PersistentVolume en función de la configuración del StorageClass.

**Advertencia:** En clústeres locales, el StorageClass predeterminado utiliza el proveedor hostPath. Los volúmenes hostPath solo son adecuados para desarrollo y pruebas. Con volúmenes hostPath, tus datos residen en /tmp en el nodo donde se planifica el Pod y no se mueven entre nodos. Si un Pod muere y se planifica en otro nodo del clúster, o si se reinicia el nodo, los datos se pierden.

**Nota:** Si estás configurando un clúster que necesita utilizar el proveedor hostPath, el flag `--enable-hostpath-provisioner` debe estar activado en el componente controller-manager.

**Nota:** Si tienes un clúster de Kubernetes en Google Kubernetes Engine, sigue [esta guía](https://kubernetes.io/docs/setup/release/notes/#gke).

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

## Conclusión

Este tutorial te ha guiado a través del proceso de despliegue de un sitio de WordPress y una base de datos MySQL utilizando Kubernetes y Minikube. Has aprendido a crear PersistentVolumes y PersistentVolumeClaims, a manejar secretos sensibles y a verificar el estado de tus recursos desplegados. Para entornos de producción, considera utilizar gráficos Helm y configuraciones más robustas.
