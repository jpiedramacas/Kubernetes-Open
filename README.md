# README: Configuración de un Pod para Utilizar un PersistentVolume para Almacenamiento

Este documento proporciona instrucciones paso a paso para configurar un Pod en Kubernetes que utilice un PersistentVolumeClaim para almacenamiento.

## Introducción

En este tutorial, aprenderás cómo:
1. Crear un PersistentVolume respaldado por almacenamiento físico.
2. Crear un PersistentVolumeClaim que se vincule automáticamente a un PersistentVolume adecuado.
3. Crear un Pod que utilice el PersistentVolumeClaim para almacenamiento.

### Requisitos Previos

- Debes tener un clúster de Kubernetes con un solo nodo y la herramienta de línea de comandos `kubectl` configurada para comunicarse con tu clúster. Si no tienes un clúster de un solo nodo, puedes crear uno usando Minikube.
- Familiarízate con el material en [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/).

## Paso 1: Crear un archivo index.html en tu Nodo

1. Abre una shell en el único nodo de tu clúster. Si estás usando Minikube, abre una shell con:
   ```sh
   minikube ssh
   ```

2. En la shell del nodo, crea un directorio `/mnt/data`:
   ```sh
   sudo mkdir /mnt/data
   ```

3. En el directorio `/mnt/data`, crea un archivo `index.html`:
   ```sh
   sudo sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
   ```

4. Verifica que el archivo `index.html` existe:
   ```sh
   cat /mnt/data/index.html
   ```
   La salida debe mostrar:
   ```
   Hello from Kubernetes storage
   ```

## Paso 2: Crear un PersistentVolume

1. Crea un archivo de configuración `pv-volume.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: task-pv-volume
     labels:
       type: local
   spec:
     storageClassName: manual
     capacity:
       storage: 10Gi
     accessModes:
       - ReadWriteOnce
     hostPath:
       path: "/mnt/data"
   ```

2. Aplica el archivo de configuración para crear el PersistentVolume:
   ```sh
   kubectl apply -f pv-volume.yaml
   ```

3. Verifica el estado del PersistentVolume:
   ```sh
   kubectl get pv task-pv-volume
   ```
   La salida debe mostrar que el estado es `Available`:
   ```
   NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS      CLAIM     STORAGECLASS   REASON    AGE
   task-pv-volume   10Gi       RWO           Retain          Available             manual                   4s
   ```

## Paso 3: Crear un PersistentVolumeClaim

1. Crea un archivo de configuración `pv-claim.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: v1
   kind: PersistentVolumeClaim
   metadata:
     name: task-pv-claim
   spec:
     storageClassName: manual
     accessModes:
       - ReadWriteOnce
     resources:
       requests:
         storage: 3Gi
   ```

2. Aplica el archivo de configuración para crear el PersistentVolumeClaim:
   ```sh
   kubectl apply -f pv-claim.yaml
   ```

3. Verifica el estado del PersistentVolumeClaim y el PersistentVolume:
   ```sh
   kubectl get pv task-pv-volume
   kubectl get pvc task-pv-claim
   ```
   La salida debe mostrar que ambos están en estado `Bound`:
   ```
   NAME             CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                   STORAGECLASS   REASON    AGE
   task-pv-volume   10Gi       RWO           Retain          Bound     default/task-pv-claim   manual                   2m

   NAME            STATUS    VOLUME           CAPACITY   ACCESSMODES   STORAGECLASS   AGE
   task-pv-claim   Bound     task-pv-volume   10Gi       RWO           manual         30s
   ```

## Paso 4: Crear un Pod

1. Crea un archivo de configuración `pv-pod.yaml` con el siguiente contenido:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: task-pv-pod
   spec:
     volumes:
       - name: task-pv-storage
         persistentVolumeClaim:
           claimName: task-pv-claim
     containers:
       - name: task-pv-container
         image: nginx
         ports:
           - containerPort: 80
             name: "http-server"
         volumeMounts:
           - mountPath: "/usr/share/nginx/html"
             name: task-pv-storage
   ```

2. Aplica el archivo de configuración para crear el Pod:
   ```sh
   kubectl apply -f pv-pod.yaml
   ```

3. Verifica que el contenedor en el Pod esté corriendo:
   ```sh
   kubectl get pod task-pv-pod
   ```

## Paso 5: Verificación de la configuración

1. Ejecuta un shell en el contenedor que está corriendo en tu Pod:
   ```sh
   kubectl exec -it task-pv-pod -- /bin/bash
   ```

2. En tu shell, verifica que nginx está sirviendo el archivo `index.html` desde el volumen hostPath:
   ```sh
   apt update
   apt install -y curl
   curl http://localhost/
   ```
   La salida debe mostrar el texto que escribiste en el archivo `index.html`:
   ```
   Hello from Kubernetes storage
   ```

## Paso 6: Limpiar

1. Elimina el Pod, el PersistentVolumeClaim y el PersistentVolume:
   ```sh
   kubectl delete pod task-pv-pod
   kubectl delete pvc task-pv-claim
   kubectl delete pv task-pv-volume
   ```

2. Abre una shell en tu nodo y elimina el archivo y el directorio que creaste:
   ```sh
   minikube ssh
   sudo rm /mnt/data/index.html
   sudo rmdir /mnt/data
   ```

¡Listo! Has configurado con éxito un Pod para utilizar almacenamiento desde un PersistentVolumeClaim en Kubernetes.
