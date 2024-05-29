# Despliegue de WordPress y MySQL con Volúmenes Persistentes

Este tutorial te muestra cómo desplegar un sitio de WordPress y una base de datos MySQL usando Minikube. Ambas aplicaciones utilizan PersistentVolumes y PersistentVolumeClaims para almacenar datos.

Un PersistentVolume (PV) es un fragmento de almacenamiento en el clúster que ha sido provisto manualmente por un administrador o provisionado dinámicamente por Kubernetes usando un StorageClass. Un PersistentVolumeClaim (PVC) es una solicitud de almacenamiento por parte de un usuario que puede ser cumplida por un PV. Los PersistentVolumes y PersistentVolumeClaims son independientes del ciclo de vida del Pod y preservan los datos a través de reinicios, reprogramaciones e incluso eliminaciones de Pods.

Advertencia:
Este despliegue no es adecuado para casos de uso en producción, ya que utiliza Pods de WordPress y MySQL de una sola instancia. Considera usar el Helm Chart de WordPress para desplegar WordPress en producción.
Nota:
Los archivos proporcionados en este tutorial utilizan APIs de implementación de GA y son específicos para Kubernetes versión 1.9 y posteriores. Si deseas utilizar este tutorial con una versión anterior de Kubernetes, actualiza apropiadamente la versión de la API o consulta versiones anteriores de este tutorial.

## Objetivos

1. Crear PersistentVolumeClaims y PersistentVolumes.
2. Crear un kustomization.yaml con:
   - Un generador de Secret.
   - Configuraciones de recursos de MySQL.
   - Configuraciones de recursos de WordPress.
3. Aplicar el directorio de kustomization con `kubectl apply -k ./`.
4. Limpieza.

## Antes de empezar

Necesitas tener un clúster de Kubernetes, y la herramienta de línea de comandos `kubectl` debe estar configurada para comunicarse con tu clúster. Se recomienda ejecutar este tutorial en un clúster con al menos dos nodos que no actúen como anfitriones de plano de control. Si aún no tienes un clúster, puedes crear uno usando Minikube o puedes usar uno de estos entornos de juego de Kubernetes:

- [Killercoda](https://www.killercodamaui.com/)
- [Play with Kubernetes](https://labs.play-with-k8s.com/)

Para verificar la versión, ingresa `kubectl version`. El ejemplo mostrado en esta página funciona con kubectl 1.27 y superior.

## Pasos

### 1. Descarga los siguientes archivos de configuración:

- [mysql-deployment.yaml](https://k8s.io/examples/application/wordpress/mysql-deployment.yaml)
- [wordpress-deployment.yaml](https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml)

### 2. Crea PersistentVolumeClaims y PersistentVolumes

Tanto MySQL como Wordpress requieren un PersistentVolume para almacenar datos. Sus PersistentVolumeClaims se crearán en el paso de implementación.

### 3. Crea un kustomization.yaml

#### Agrega un generador de Secret

Un Secret es un objeto que almacena datos sensibles como contraseñas o claves. Desde la versión 1.14, kubectl admite la gestión de objetos de Kubernetes mediante un archivo kustomization. Puedes crear un Secret mediante generadores en kustomization.yaml.

Agrega un generador de Secret en kustomization.yaml con el siguiente comando. Deberás reemplazar `YOUR_PASSWORD` con la contraseña que desees utilizar.

```yaml
secretGenerator:
- name: mysql-pass
  literals:
  - password=YOUR_PASSWORD
```

#### Agrega configuraciones de recursos para MySQL y WordPress

El siguiente manifiesto describe un despliegue de MySQL de instancia única. El contenedor de MySQL monta el PersistentVolume en `/var/lib/mysql`. La variable de entorno `MYSQL_ROOT_PASSWORD` establece la contraseña de la base de datos desde el Secret.

**mysql-deployment.yaml:**

```yaml
# ... (contenido del archivo mysql-deployment.yaml)
```

El siguiente manifiesto describe un despliegue de WordPress de instancia única. El contenedor de WordPress monta el PersistentVolume en `/var/www/html` para los archivos de datos del sitio web. La variable de entorno `WORDPRESS_DB_HOST` establece el nombre del Servicio MySQL definido anteriormente, y WordPress accederá a la base de datos mediante el Servicio. La variable de entorno `WORDPRESS_DB_PASSWORD` establece la contraseña de la base de datos desde el Secret generado por kustomize.

**wordpress-deployment.yaml:**

```yaml
# ... (contenido del archivo wordpress-deployment.yaml)
```

### 4. Aplica y verifica

El kustomization.yaml contiene todos los recursos para desplegar un sitio de WordPress y una base de datos MySQL. Puedes aplicar el directorio con:

```bash
kubectl apply -k ./
```

Ahora puedes verificar que todos los objetos existen.

- Verifica que el Secret existe ejecutando el siguiente comando:
  
  ```bash
  kubectl get secrets
  ```

- Verifica que se haya provisionado dinámicamente un PersistentVolume.

  ```bash
  kubectl get pvc
  ```

- Verifica que el Pod esté en ejecución.

  ```bash
  kubectl get pods
  ```

- Verifica que el Servicio esté en ejecución.

  ```bash
  kubectl get services wordpress
  ```

## Limpieza

Ejecuta el siguiente comando para eliminar tu Secret, Despliegues, Servicios y PersistentVolumeClaims:

```bash
kubectl delete -k ./
```

¡Y eso es todo! Has desplegado con éxito un sitio de WordPress y una base de datos MySQL en tu clúster de Kubernetes utilizando volúmenes persistentes.
