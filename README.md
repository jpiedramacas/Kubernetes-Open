
### 1. Archivo de Deployment

Crea un archivo llamado `httpd-deployment.yaml` con el siguiente contenido:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-deploy
  labels:
    app: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - name: httpd
        image: public.ecr.aws/ubuntu/apache2:2.4-20.04_beta
        ports:
        - containerPort: 80
```

### 2. Archivo de Service

Crea un archivo llamado `httpd-service.yaml` con el siguiente contenido:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: httpd-service
spec:
  selector:
    app: httpd
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

### Pasos para Desplegar los Archivos en Kubernetes

1. **Aplicar el Deployment**:

    ```bash
    kubectl apply -f httpd-deployment.yaml
    ```

2. **Aplicar el Service**:

    ```bash
    kubectl apply -f httpd-service.yaml
    ```

3. **Verificar el Estado del Deployment y el Service**:

    - Verifica el Deployment:

        ```bash
        kubectl get deployments
        ```

    - Verifica el Service:

        ```bash
        kubectl get services
        ```

    Si el `EXTERNAL-IP` del `httpd-service` aparece como `<pending>`, sigue los pasos adicionales para configurar el balanceador de carga en tu entorno (como se describe en la sección de solución de problemas en la respuesta anterior).

4. **Acceder al Servidor Web**:

    Una vez que el `EXTERNAL-IP` esté disponible, abre un navegador web y navega a la dirección `http://<EXTERNAL-IP>:80`. Deberías ver la página de bienvenida de Apache HTTPD.

    Si estás utilizando Minikube, puedes acceder al servicio utilizando el comando `minikube service`:

    ```bash
    minikube service httpd-service
    ```

### Notas Adicionales

- **EXTERNAL-IP pendiente**: Si el `EXTERNAL-IP` del `httpd-service` permanece como `<pending>`, asegúrate de que tu entorno Kubernetes soporta servicios de tipo `LoadBalancer`. Si estás utilizando Minikube, ejecuta `minikube tunnel` para asignar una IP externa.
- **Acceso usando NodePort**: Como alternativa, puedes modificar el tipo de servicio a `NodePort` para acceder a la aplicación a través de la IP del nodo y un puerto específico.

Guarda estos archivos y sigue los comandos para desplegar y verificar tu servidor Apache HTTPD en Kubernetes.
