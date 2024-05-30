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

3. Luego, continúa con el resto de los pasos según el contenido del archivo original.
## Pasos para gestionar WordPress en Kubernetes

1. **Aplicar configuración inicial**: Utiliza el comando `kubectl apply -k ./` para aplicar la configuración inicial y desplegar WordPress en el clúster.

2. **Acceder a WordPress**: Para acceder a WordPress, ejecuta `kubectl port-forward service/wordpress 80:80` y visita `localhost` en tu navegador.

3. **Realizar cambios en WordPress**: Inicia sesión en WordPress y realiza cambios como añadir páginas o modificar texto según sea necesario.

4. **Modificar la configuración**: Elimina las líneas de código mencionadas en `wordpress-deployment.yaml` y `mysql-deployment.yaml` para permitir cambios en los volúmenes persistentes.

5. **Eliminar los despliegues actuales**: Usa `kubectl delete deployment --all` para eliminar los despliegues actuales y los pods.

6. **Volver a desplegar**: Vuelve a aplicar la configuración con `kubectl apply -k ./` para desplegar WordPress con los cambios en los volúmenes persistentes.

7. **Verificar cambios**: Accede a WordPress nuevamente utilizando `kubectl port-forward service/wordpress 80:80` y verifica que los cambios realizados anteriormente estén presentes.

8. **Restaurar configuración original**: Repite los pasos 4-7, pero esta vez restaura las líneas de código eliminadas en los archivos `wordpress-deployment.yaml` y `mysql-deployment.yaml` para volver a la configuración original.

Al seguir estos pasos, podrás gestionar eficazmente una instancia de WordPress en un clúster de Kubernetes y realizar cambios en los volúmenes persistentes según sea necesario.
