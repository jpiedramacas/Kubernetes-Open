# Desplegar Nginx en Kubernetes

Este documento describe los pasos necesarios para desplegar Nginx en un clúster de Kubernetes utilizando un Deployment y un Service. A continuación, se proporcionan los comandos y archivos necesarios.

## Prerrequisitos

- Kubernetes instalado y configurado.
- `kubectl` configurado para interactuar con tu clúster de Kubernetes.

## Pasos

### 1. Archivos de configuracion Nginx

Debemos tener dos archivos de configuracion  `nginx-deployment.yaml` y `nginx-service.yaml`

### 2. Desplegar los archivos en Kubernetes

Para desplegar los archivos, utiliza los siguientes comandos:

```bash
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
```

### 3. Verificar el estado del Deployment y el Service

Verifica que el Deployment y el Service se hayan creado correctamente y estén funcionando.

#### Verificar el Deployment

```bash
kubectl get deployments
```

Deberías ver una salida similar a:

```
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   3/3     3            3           1m
```

#### Verificar el Service

```bash
kubectl get services
```

Deberías ver una salida similar a:

```
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP      10.96.0.1       <none>        443/TCP        10m
nginx-service   LoadBalancer   10.97.234.51    <pending>     80:31357/TCP   2m
```

Si el `EXTERNAL-IP` del `nginx-service` aparece como `<pending>`, sigue los pasos adicionales para configurar el balanceador de carga en tu entorno (como se describe en la sección de solución de problemas).

### 4. Comprobar que el servidor web funciona correctamente

Una vez que el `EXTERNAL-IP` esté disponible, abre un navegador web y navega a la dirección `http://<EXTERNAL-IP>:80`. Deberías ver la página de bienvenida de Nginx.

Si estás utilizando Minikube, puedes acceder al servicio utilizando el comando `minikube service`:

```bash
minikube service nginx-service
```

Esto abrirá el navegador con la URL correcta para acceder a Nginx.

## Solución de Problemas

- **EXTERNAL-IP pendiente**: Si el `EXTERNAL-IP` del `nginx-service` permanece como `<pending>`, asegúrate de que tu entorno Kubernetes soporta servicios de tipo `LoadBalancer`. Si estás utilizando Minikube, ejecuta `minikube tunnel` para asignar una IP externa.
- **Acceso usando NodePort**: Como alternativa, puedes modificar el tipo de servicio a `NodePort` para acceder a la aplicación a través de la IP del nodo y un puerto específico.

Guarda estos archivos y sigue los comandos para desplegar y verificar tu servidor Nginx en Kubernetes.
