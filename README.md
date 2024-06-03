
# README: Despliegue y Modificación de un Pod en Kubernetes

Este README proporciona instrucciones detalladas sobre cómo desplegar un Pod en un clúster de Kubernetes, acceder a él, modificar el archivo `index.html` y ver los cambios utilizando `kubectl port-forward`.

## Paso 1: Despliegue del Pod en Kubernetes

1. **Creación del archivo YAML del Pod**:
   Crea un archivo `pod.yaml` con la definición del Pod. Por ejemplo:
   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-pod
   spec:
     containers:
     - name: my-container
       image: nginx:latest
       ports:
       - containerPort: 80
   ```

2. **Aplicación del archivo YAML**:
   Aplica el archivo YAML usando el comando `kubectl apply`:
   ```sh
   kubectl apply -f pod.yaml
   ```

## Paso 2: Acceso al Pod en Kubernetes

1. **Listado de Pods**:
   Verifica que el Pod esté en estado `Running`:
   ```sh
   kubectl get pods
   ```

2. **Acceso al contenedor del Pod**:
   Accede al contenedor del Pod usando `kubectl exec`:
   ```sh
   kubectl exec -it my-pod -- /bin/bash
   ```

## Paso 3: Modificación del archivo `index.html`

1. **Actualización del sistema de archivos del contenedor**:
   Antes de editar el archivo `index.html`, actualiza y asegura que Nano esté instalado:
   ```sh
   apt-get update
   apt-get install -y nano
   ```

2. **Ubicación del archivo `index.html`**:
   Dentro del contenedor, encuentra la ubicación del archivo `index.html`:
   ```sh
   find / -name "index.html"
   ```

3. **Edición del archivo `index.html`**:
   Edita el archivo `index.html` según tus necesidades:
   ```sh
   nano /usr/share/nginx/html/index.html
   ```

   Aquí tienes un ejemplo básico de `index.html`:
   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>Proyecto GIO-JP</title>
   </head>
   <body>
   <h1>Bienvenido a Proyecto GIO-JP</h1>
   <p>¡Gracias por visitar nuestro sitio! Somos un equipo apasionado comprometido con la excelencia y la innovación en el desarrollo de software.</p>
   <p>Este es el comienzo de un emocionante viaje tecnológico. Estamos aquí para proporcionar soluciones creativas y eficientes para tus necesidades.</p>
   <p>No dudes en ponerte en contacto con nosotros si necesitas ayuda o más información.</p>
   <p><em>¡Gracias por ser parte de nuestro proyecto!</em></p>
   </body>
   </html>
   ```

4. **Verificación de los cambios**:
   Verifica los cambios accediendo al servicio Nginx a través de tu navegador o usando `curl`.

## Paso 4: Verificación de los cambios utilizando kubectl port-forward

1. **Ejecución de kubectl port-forward**:
   Para ver los cambios en el archivo `index.html` en tu navegador local, ejecuta el siguiente comando para reenviar el puerto del contenedor al puerto local:
   ```sh
   kubectl port-forward my-pod 80:80
   ```

2. **Acceso al sitio web**:
   Abre tu navegador web y accede a `http://localhost:80` para ver los cambios reflejados en el archivo `index.html` del contenedor.

## Paso 5: 

