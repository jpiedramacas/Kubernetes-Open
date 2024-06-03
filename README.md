
# README: Despliegue y Modificación de un Pod en Kubernetes

Este README proporciona instrucciones detalladas sobre cómo desplegar un Pod en un clúster de Kubernetes, acceder a él, modificar el archivo `index.html` y ver los cambios utilizando `kubectl port-forward`.

## Paso 1: Despliegue del Pod en Kubernetes

1. **Creación del archivo YAML del Pod**:
   Crea un archivo `pod-definition.yaml` con la definición del Pod. Por ejemplo:
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
<style>
  body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background-color: #000;
    color: #fff;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
  }

  .container {
    max-width: 800px;
    padding: 20px;
    text-align: center;
    background-color: rgba(255, 255, 255, 0.1);
    border-radius: 8px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
  }

  h1 {
    font-size: 36px;
    color: #007bff;
  }

  p {
    font-size: 18px;
    line-height: 1.6;
  }

  a {
    color: #007bff;
    text-decoration: none;
  }

  a:hover {
    text-decoration: underline;
  }
</style>
</head>
<body>
<div class="container">
  <h1>Bienvenido a Proyecto GIO-JP</h1>
  <p>¡Gracias por visitar nuestro sitio! Somos un equipo apasionado comprometido con la excelencia y la innovación en el desarrollo de software.</p>
  <p>Este es el comienzo de un emocionante viaje tecnológico. Estamos aquí para proporcionar soluciones creativas y eficientes para tus necesidades.</p>
  <p>No dudes en ponerte en contacto con nosotros si necesitas ayuda o más información.</p>
  <p><a href="http://giogp.com/" target="_blank">Visita nuestro sitio web</a> para más detalles.</p>
  <p><em>¡Gracias por ser parte de nuestro proyecto!</em></p>
</div>
</body>
</html>
```

## Paso 4: Verificación de los cambios utilizando kubectl port-forward

1. **Ejecución de kubectl port-forward**:
   Para ver los cambios en el archivo `index.html` en tu navegador local, ejecuta el siguiente comando para reenviar el puerto del contenedor al puerto local:
   ```sh
   kubectl port-forward my-pod 80:80
   ```

2. **Acceso al sitio web**:
   Abre tu navegador web y accede a `http://localhost:80` para ver los cambios reflejados en el archivo `index.html` del contenedor.

## Paso 5: 

