# Kubernetes en la nube (EKS, GKE, AKS)

Las principales plataformas de nube ofrecen servicios gestionados de Kubernetes, facilitando la creación y administración de clústeres. En esta guía, se detallan los pasos necesarios para crear y configurar clústeres en Amazon EKS, Google Kubernetes Engine (GKE) y Azure Kubernetes Service (AKS).

## Amazon EKS (Elastic Kubernetes Service)

### 1. Configuración preliminar

1.1. **Instalar AWS CLI**: Si aún no tienes instalada la AWS CLI, sigue las instrucciones en la [documentación oficial](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html).

1.2. **Configurar AWS CLI**: Configura tus credenciales de AWS usando el comando:
   ```sh
   aws configure
   ```
   Se te pedirá que ingreses tu `AWS Access Key ID`, `AWS Secret Access Key`, `region` y `output format`.

### 2. Crear un clúster EKS

2.1. **Crear un rol IAM**: Necesitas un rol IAM con las políticas necesarias para crear y gestionar recursos de EKS. Puedes hacerlo a través de la consola de IAM en AWS.

2.2. **Crear el clúster**:
   ```sh
   aws eks create-cluster --name <cluster-name> --region <region-code> --role-arn <iam-role-arn> --resources-vpc-config subnetIds=<subnet-ids>,securityGroupIds=<security-group-ids>
   ```

2.3. **Esperar la creación del clúster**: La creación del clúster puede tardar varios minutos.

### 3. Configurar kubectl para acceder al clúster

3.1. **Instalar kubectl**: Sigue las instrucciones de la [documentación oficial](https://kubernetes.io/docs/tasks/tools/install-kubectl/) para instalar kubectl.

3.2. **Configurar kubectl**:
   ```sh
   aws eks --region <region-code> update-kubeconfig --name <cluster-name>
   ```

3.3. **Verificar la configuración**:
   ```sh
   kubectl get nodes
   ```

## Google Kubernetes Engine (GKE)

### 1. Configuración preliminar

1.1. **Instalar Google Cloud SDK**: Si aún no tienes instalado el Google Cloud SDK, sigue las instrucciones en la [documentación oficial](https://cloud.google.com/sdk/docs/install).

1.2. **Configurar Google Cloud SDK**:
   ```sh
   gcloud init
   ```
   Se te pedirá que selecciones un proyecto y configures tu cuenta.

### 2. Crear un clúster GKE

2.1. **Habilitar la API de Kubernetes Engine**:
   ```sh
   gcloud services enable container.googleapis.com
   ```

2.2. **Crear el clúster**:
   ```sh
   gcloud container clusters create <cluster-name> --zone <zone-name> --project <project-id>
   ```

2.3. **Esperar la creación del clúster**: La creación del clúster puede tardar varios minutos.

### 3. Configurar kubectl para acceder al clúster

3.1. **Instalar kubectl**: Si no está instalado, puedes instalarlo utilizando el Google Cloud SDK:
   ```sh
   gcloud components install kubectl
   ```

3.2. **Configurar kubectl**:
   ```sh
   gcloud container clusters get-credentials <cluster-name> --zone <zone-name> --project <project-id>
   ```

3.3. **Verificar la configuración**:
   ```sh
   kubectl get nodes
   ```

## Azure Kubernetes Service (AKS)

### 1. Configuración preliminar

1.1. **Instalar Azure CLI**: Si aún no tienes instalada la Azure CLI, sigue las instrucciones en la [documentación oficial](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

1.2. **Iniciar sesión en Azure**:
   ```sh
   az login
   ```

### 2. Crear un clúster AKS

2.1. **Crear un grupo de recursos**:
   ```sh
   az group create --name <resource-group-name> --location <location>
   ```

2.2. **Crear el clúster**:
   ```sh
   az aks create --resource-group <resource-group-name> --name <cluster-name> --node-count <node-count> --enable-addons monitoring --generate-ssh-keys
   ```

2.3. **Esperar la creación del clúster**: La creación del clúster puede tardar varios minutos.

### 3. Configurar kubectl para acceder al clúster

3.1. **Instalar kubectl**: Si no está instalado, puedes instalarlo utilizando la Azure CLI:
   ```sh
   az aks install-cli
   ```

3.2. **Configurar kubectl**:
   ```sh
   az aks get-credentials --resource-group <resource-group-name> --name <cluster-name>
   ```

3.3. **Verificar la configuración**:
   ```sh
   kubectl get nodes
   ```

## Conclusión

Cada una de las principales plataformas en la nube ofrece soluciones gestionadas de Kubernetes que simplifican la administración de clústeres. Al seguir los pasos detallados en esta guía, puedes crear y configurar clústeres en Amazon EKS, Google Kubernetes Engine (GKE) y Azure Kubernetes Service (AKS) de manera eficiente. Asegúrate de consultar la documentación oficial de cada proveedor para obtener más detalles y actualizaciones sobre sus servicios.
