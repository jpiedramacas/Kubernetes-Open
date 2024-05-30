## Antes de comenzar

Antes de comenzar, necesitas tener un clúster de Kubernetes configurado y la herramienta de línea de comandos `kubectl` debe estar configurada para comunicarse con tu clúster. Se recomienda ejecutar este tutorial en un clúster con al menos dos nodos que no actúen como hosts del plano de control. Si aún no tienes un clúster, puedes crear uno utilizando Minikube o puedes usar alguno de estos playgrounds de Kubernetes:

Para verificar la versión, ingresa el comando `kubectl version`. 

El ejemplo mostrado en esta página funciona con `kubectl` versión 1.27 o superior.

### Pasos a seguir:

1. Descarga los siguientes archivos de configuración:
   - [mysql-deployment.yaml](link)
   - [wordpress-deployment.yaml](link)

2. Crea un archivo `kustomization.yaml` e incluye un generador de Secretos utilizando el siguiente comando. Asegúrate de reemplazar `YOUR_PASSWORD` con la contraseña que deseas utilizar:

```bash
secretGenerator:
- name: mysql-pass
  literals:
  - password=mysql1234
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml
```

## Pasos para gestionar WordPress en Kubernetes

### Aplicar configuración inicial
Utiliza el comando para aplicar la configuración inicial y desplegar WordPress en el clúster.

```bash
kubectl apply -k ./
```

### Acceder a WordPress
Para acceder a WordPress, ejecuta el comando y visita `localhost` en tu navegador.

```bash
kubectl port-forward service/wordpress 80:80
```

### Realizar cambios en WordPress
Inicia sesión en WordPress y realiza cambios como añadir páginas o modificar texto según sea necesario.

### Modificar la configuración
Elimina las líneas de código mencionadas en `wordpress-deployment.yaml` y `mysql-deployment.yaml` para permitir cambios en los volúmenes persistentes.

### mysql-deployment.yaml

```yaml
volumeMounts:
- name: mysql-persistent-storage
  mountPath: /var/lib/mysql
```

### wordpress-deployment.yaml

```yaml
volumeMounts:
- name: wordpress-persistent-storage
  mountPath: /var/www/html
```


### Eliminar los despliegues actuales
Usa el comando para eliminar los despliegues actuales y los pods.

```bash
kubectl delete deployment --all
```

### Volver a desplegar
Vuelve a aplicar la configuración para desplegar WordPress con los cambios en los volúmenes persistentes.

```bash
kubectl apply -k ./
```

### Verificar cambios
Accede a WordPress nuevamente utilizando el comadno y verifica que los cambios realizados anteriormente estén presentes.

```bash
kubectl port-forward service/wordpress 80:80
```

### Restaurar configuración original
Repite los pasos 4-7, pero esta vez restaura las líneas de código eliminadas en los archivos `wordpress-deployment.yaml` y `mysql-deployment.yaml` para volver a la configuración original.

Al seguir estos pasos, podrás gestionar eficazmente una instancia de WordPress en un clúster de Kubernetes y realizar cambios en los volúmenes persistentes según sea necesario.
