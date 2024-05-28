Aquí tienes una lista de comandos fundamentales de Kubernetes utilizando `kubectl` para revisar y administrar diferentes componentes en el clúster:

## Pods
- **Listar todos los pods en todos los namespaces:**
  ```bash
  kubectl get pods --all-namespaces
  ```
- **Listar todos los pods en un namespace específico (por ejemplo, default):**
  ```bash
  kubectl get pods -n default
  ```
- **Ver los detalles de un pod específico:**
  ```bash
  kubectl describe pod <pod-name> -n <namespace>
  ```
- **Ver los logs de un pod específico:**
  ```bash
  kubectl logs <pod-name> -n <namespace>
  ```

## Replication Controllers
- **Listar todos los replication controllers en todos los namespaces:**
  ```bash
  kubectl get rc --all-namespaces
  ```
- **Listar todos los replication controllers en un namespace específico:**
  ```bash
  kubectl get rc -n default
  ```
- **Ver los detalles de un replication controller específico:**
  ```bash
  kubectl describe rc <rc-name> -n <namespace>
  ```

## Deployments
- **Listar todos los deployments en todos los namespaces:**
  ```bash
  kubectl get deployments --all-namespaces
  ```
- **Listar todos los deployments en un namespace específico:**
  ```bash
  kubectl get deployments -n default
  ```
- **Ver los detalles de un deployment específico:**
  ```bash
  kubectl describe deployment <deployment-name> -n <namespace>
  ```
- **Ver el historial de revisiones de un deployment:**
  ```bash
  kubectl rollout history deployment <deployment-name> -n <namespace>
  ```
- **Realizar un rollback de un deployment a una revisión específica:**
  ```bash
  kubectl rollout undo deployment <deployment-name> --to-revision=<revision-number> -n <namespace>
  ```

## Services
- **Listar todos los servicios en todos los namespaces:**
  ```bash
  kubectl get services --all-namespaces
  ```
- **Listar todos los servicios en un namespace específico:**
  ```bash
  kubectl get services -n default
  ```
- **Ver los detalles de un servicio específico:**
  ```bash
  kubectl describe service <service-name> -n <namespace>
  ```

## ConfigMaps
- **Listar todos los ConfigMaps en todos los namespaces:**
  ```bash
  kubectl get configmaps --all-namespaces
  ```
- **Listar todos los ConfigMaps en un namespace específico:**
  ```bash
  kubectl get configmaps -n default
  ```
- **Ver los detalles de un ConfigMap específico:**
  ```bash
  kubectl describe configmap <configmap-name> -n <namespace>
  ```

## Secrets
- **Listar todos los Secrets en todos los namespaces:**
  ```bash
  kubectl get secrets --all-namespaces
  ```
- **Listar todos los Secrets en un namespace específico:**
  ```bash
  kubectl get secrets -n default
  ```
- **Ver los detalles de un Secret específico:**
  ```bash
  kubectl describe secret <secret-name> -n <namespace>
  ```

## Namespaces
- **Listar todos los namespaces:**
  ```bash
  kubectl get namespaces
  ```
- **Ver los detalles de un namespace específico:**
  ```bash
  kubectl describe namespace <namespace-name>
  ```
- **Crear un nuevo namespace:**
  ```bash
  kubectl create namespace <namespace-name>
  ```
- **Eliminar un namespace:**
  ```bash
  kubectl delete namespace <namespace-name>
  ```

Estos comandos te permiten gestionar y obtener información detallada sobre los diferentes componentes en tu clúster de Kubernetes, lo cual es esencial para la administración efectiva del clúster.
