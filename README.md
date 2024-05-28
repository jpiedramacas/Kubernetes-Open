# Desplegar una Aplicación Web con un Volumen Persistente en Kubernetes

Este documento proporciona una guía detallada sobre cómo montar un volumen persistente en un pod de Kubernetes mediante la definición de un PersistentVolumeClaim (PVC) y su asociación con un Deployment. 

## Archivos de Configuración

### PersistentVolumeClaim

Crea un archivo llamado `pvc.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-volume-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Deployment

Crea un archivo llamado `deployment.yaml` con el siguiente contenido:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      volumes:
        - name: my-volume
          persistentVolumeClaim:
            claimName: my-volume-claim
      containers:
        - name: nginx
          image: nginx:latest
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: my-volume
```

## Desplegar los Recursos en Kubernetes

Sigue estos pasos para desplegar los recursos definidos en los archivos YAML.

### 1. Aplicar el PersistentVolumeClaim

Ejecuta el siguiente comando para crear el PersistentVolumeClaim en Kubernetes:

```bash
kubectl apply -f pvc.yaml
```

### 2. Aplicar el Deployment

Ejecuta el siguiente comando para crear el Deployment en Kubernetes:

```bash
kubectl apply -f deployment.yaml
```

### 3. Verificar el Estado de los Recursos

#### Verificar el PersistentVolumeClaim

```bash
kubectl get pvc
```

Deberías ver una salida similar a:

```
NAME             STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
my-volume-claim  Bound    pvc-xyz  1Gi        RWO            standard       1m
```

#### Verificar el Deployment y los Pods

```bash
kubectl get deployments
kubectl get pods
```

Deberías ver una salida similar a:

```
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
webapp    3/3     3            3           2m
```

Y para los pods:

```
NAME                      READY   STATUS    RESTARTS   AGE
webapp-<random-string>    1/1     Running   0          2m
webapp-<random-string>    1/1     Running   0          2m
webapp-<random-string>    1/1     Running   0          2m
```

### 4. Acceder a los Logs de los Pods

Para ver los logs de un pod específico:

```bash
kubectl logs <pod-name>
```

### 5. Ver Detalles del Deployment

Para ver los detalles de un deployment específico:

```bash
kubectl describe deployment webapp
```

## Solución de Problemas

### PersistentVolumeClaim No Bound

Si el `PersistentVolumeClaim` no está en estado `Bound`, verifica los recursos de almacenamiento disponibles en tu clúster:

```bash
kubectl get pv
```

### Pods No Iniciando

Si los pods no están en estado `Running`, revisa los eventos del pod para obtener más detalles:

```bash
kubectl describe pod <pod-name>
```

## Conclusión

Siguiendo estos pasos, has configurado y desplegado una aplicación web (usando Nginx) en un clúster de Kubernetes con un volumen persistente montado. Esto garantiza que los datos en el directorio `/usr/share/nginx/html` persistan incluso si los pods se eliminan y recrean.

Estos comandos y configuraciones proporcionan una base sólida para el manejo de volúmenes persistentes y despliegues en Kubernetes.
