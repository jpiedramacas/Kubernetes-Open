Aquí tienes la estructura revisada con los números de paso:

**Antes de comenzar:**

1. Antes de comenzar, necesitas tener un clúster de Kubernetes configurado y la herramienta de línea de comandos kubectl debe estar configurada para comunicarse con tu clúster. Se recomienda ejecutar este tutorial en un clúster con al menos dos nodos que no actúen como hosts del plano de control. Si aún no tienes un clúster, puedes crear uno utilizando Minikube o puedes usar alguno de estos playgrounds de Kubernetes:

2. Para verificar la versión, ingresa el comando 'kubectl version'

3. El ejemplo mostrado en esta página funciona con kubectl versión 1.27 o superior.

**Pasos a seguir:**

4. Descarga los siguientes archivos de configuración:

   - [mysql-deployment.yaml](link)
   - [wordpress-deployment.yaml](link)

5. Crea un archivo kustomization.yaml e incluye un generador de Secretos utilizando el siguiente comando. Asegúrate de reemplazar YOUR_PASSWORD con la contraseña que deseas utilizar:

    ```yaml
    secretGenerator:
    - name: mysql-pass
      literals:
      - password=mysql1234
    resources:
      - mysql-deployment.yaml
      - wordpress-deployment.yaml
    ```

**Pasos para gestionar WordPress en Kubernetes:**

6. **Aplicar configuración inicial:**
   Utiliza el siguiente comando para aplicar la configuración inicial y desplegar WordPress en el clúster.

   ```bash
   kubectl apply -k ./
   ```

7. **Acceder a WordPress:**
   Para acceder a WordPress, ejecuta el siguiente comando y visita localhost en tu navegador.

   ```bash
   kubectl port-forward service/wordpress 80:80
   ```

8. **Realizar cambios en WordPress:**
   Inicia sesión en WordPress y realiza cambios como añadir páginas o modificar texto según sea necesario.

9. **Modificar la configuración:**
   Elimina las líneas de código mencionadas en wordpress-deployment.yaml y mysql-deployment.yaml para permitir cambios en los volúmenes persistentes.

   **mysql-deployment.yaml**
   ```yaml
   volumeMounts:
   - name: mysql-persistent-storage
     mountPath: /var/lib/mysql
   ```

   **wordpress-deployment.yaml**
   ```yaml
   volumeMounts:
   - name: wordpress-persistent-storage
     mountPath: /var/www/html
   ```

10. **Eliminar los despliegues actuales:**
    Usa el siguiente comando para eliminar los despliegues actuales y los pods.

    ```bash
    kubectl delete deployment --all
    ```

11. **Volver a desplegar:**
    Vuelve a aplicar la configuración para desplegar WordPress con los cambios en los volúmenes persistentes.

    ```bash
    kubectl apply -k ./
    ```

12. **Verificar cambios:**
    Accede a WordPress nuevamente utilizando el siguiente comando y verifica que los cambios realizados anteriormente estén presentes.

    ```bash
    kubectl port-forward service/wordpress 80:80
    ```

13. **Restaurar configuración original:**
    Repite los pasos 10-12, pero esta vez restaura las líneas de código eliminadas en los archivos 'wordpress-deployment.yaml' y 'mysql-deployment.yaml' para volver a la configuración original.

Al seguir estos pasos, podrás gestionar eficazmente una instancia de WordPress en un clúster de Kubernetes y realizar cambios en los volúmenes persistentes según sea necesario.
