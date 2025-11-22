# Secrets Requeridos para GitHub Actions

Este documento lista todos los secrets que deben configurarse en GitHub para que los pipelines funcionen correctamente.

## Secrets Comunes (todos los ambientes)

1. **DOCKERHUB_USERNAME**: Usuario de Docker Hub
2. **DOCKERHUB_TOKEN**: Token de Docker Hub
3. **INFRA_REPO**: Nombre del repositorio de infraestructura (ej: `usuario/terraform-infra-organization`)

## Secrets para DEV (Azure)

Estos secrets ya deberían estar configurados y funcionando:

1. **AZURE_CREDENTIALS**: Service Principal JSON de Azure Dev
   ```json
   {
     "clientId": "...",
     "clientSecret": "...",
     "subscriptionId": "...",
     "tenantId": "..."
   }
   ```
2. **AZURE_RESOURCE_GROUP**: Nombre del resource group de Azure Dev
   - Ejemplo: `az-k8s-rg`
   - Verificar con: `terraform output azure_resource_group_name` (desde `terraform-infra-organization/environments/azure-dev`)
3. **AZURE_AKS_CLUSTER**: Nombre del cluster AKS de Dev
   - Ejemplo: `az-k8s-cluster`
   - Verificar con: `terraform output azure_aks_cluster_name` (desde `terraform-infra-organization/environments/azure-dev`)

## Secrets para STAGE (Azure - Cuenta de Emely)

**NUEVOS SECRETS A CONFIGURAR:**

1. **AZURE_CREDENTIALS_STAGE**: Service Principal JSON de Azure Stage (cuenta de Emely)
   ```json
   {
     "clientId": "...",
     "clientSecret": "...",
     "subscriptionId": "...",
     "tenantId": "..."
   }
   ```
   - **Cómo obtenerlo**: Emely debe crear un Service Principal en su cuenta de Azure Stage
   - **Comando**:
     ```bash
     az ad sp create-for-rbac --name "github-actions-stage" --role contributor --scopes /subscriptions/{subscription-id} --sdk-auth
     ```

2. **AZURE_RESOURCE_GROUP_STAGE**: Nombre del resource group de Azure Stage
   - Valor: `az-k8s-stage-rg` (según outputs de terraform)
   - Verificar con: `terraform output resource_group_name` (desde `terraform-infra-organization/environments/azure-stage`)

3. **AZURE_AKS_CLUSTER_STAGE**: Nombre del cluster AKS de Stage
   - Valor: `az-k8s-stage-cluster` (según outputs de terraform)
   - Verificar con: `terraform output aks_cluster_name` (desde `terraform-infra-organization/environments/azure-stage`)

## Secrets para PROD (GCP - Cuenta de tu compañero)

**NUEVOS SECRETS A CONFIGURAR:**

1. **GCP_CREDENTIALS_PROD**: Service Account JSON de GCP Production
   - **Cómo obtenerlo**: Tu compañero debe crear un Service Account en GCP y descargar la key JSON
   - **Pasos**:
     1. Ir a GCP Console → IAM & Admin → Service Accounts
     2. Crear nuevo Service Account (ej: `github-actions-prod`)
     3. Asignar roles: `Kubernetes Engine Developer`, `Container Cluster Admin`
     4. Crear key JSON y descargarla
     5. Copiar el contenido completo del JSON y pegarlo en el secret

2. **GCP_PROJECT_ID_PROD**: ID del proyecto de GCP Production
   - Valor: `tfg-prod-478914` (según outputs de terraform)
   - Verificar con: `terraform output` (desde `terraform-infra-organization/environments/gcp-prod`)

3. **GKE_CLUSTER_NAME_PROD**: Nombre del cluster GKE de Production
   - Valor: `gke-prod-cluster` (según outputs de terraform)
   - Verificar con: `terraform output gke_cluster_name` (desde `terraform-infra-organization/environments/gcp-prod`)

4. **GKE_CLUSTER_LOCATION_PROD**: Ubicación del cluster GKE
   - Valor: `us-central1-a` (según outputs de terraform)
   - Verificar con: `terraform output cluster_location` (desde `terraform-infra-organization/environments/gcp-prod`)

## Cómo Configurar Secrets en GitHub

1. Ir al repositorio en GitHub
2. Settings → Secrets and variables → Actions
3. Click en "New repository secret"
4. Agregar cada secret con su nombre y valor

## Verificación

Para verificar que los secrets están correctos, puedes ejecutar los pipelines manualmente desde la pestaña "Actions" en GitHub.

## Notas Importantes

- **DEV**: Los secrets ya deberían estar configurados y funcionando
- **STAGE**: Necesita que Emely proporcione las credenciales de Azure Stage
- **PROD**: Necesita que tu compañero proporcione las credenciales de GCP

