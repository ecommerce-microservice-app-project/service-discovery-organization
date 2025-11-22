# Configuración de Secrets para Stage

## Información que ya tenemos (de Terraform)

Según `terraform-infra-organization/environments/azure-stage`:

- **Resource Group**: `az-k8s-stage-rg`
- **Cluster Name**: `az-k8s-stage-cluster`
- **Location**: `East US 2`

## Lo que necesita Emely (cuenta de Azure Stage)

Emely necesita crear un **Service Principal** en su cuenta de Azure Stage y darte las credenciales.

### Paso 1: Emely debe hacer login en su cuenta de Azure Stage

```bash
az login
az account list  # Ver todas las suscripciones
az account set --subscription "<staging-subscription-id>"  # Seleccionar la suscripción de stage
az account show  # Verificar que está en la suscripción correcta
```

### Paso 2: Emely debe crear el Service Principal

```bash
az ad sp create-for-rbac \
  --name "github-actions-stage" \
  --role "Contributor" \
  --scopes "/subscriptions/<staging-subscription-id>" \
  --sdk-auth
```

**IMPORTANTE**: El flag `--sdk-auth` genera el formato JSON que necesitamos.

### Paso 3: Emely debe darte el output

El comando anterior devuelve un JSON como este:

```json
{
  "clientId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "clientSecret": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}
```

**Emely debe darte este JSON completo.**

## Configurar Secrets en GitHub

Una vez que tengas el JSON de Emely, ve a:

**Repositorio**: `service-discovery-organization`  
**Settings** → **Secrets and variables** → **Actions** → **New repository secret**

### Secret 1: `AZURE_CREDENTIALS_STAGE`

**Valor**: El JSON completo que te dio Emely, en una sola línea:

```json
{"clientId":"xxx","clientSecret":"xxx","subscriptionId":"xxx","tenantId":"xxx"}
```

**IMPORTANTE**: 
- Debe ser el JSON completo
- En una sola línea (sin saltos de línea)
- Con todas las comillas dobles

### Secret 2: `AZURE_RESOURCE_GROUP_STAGE`

**Valor**: `az-k8s-stage-rg`

### Secret 3: `AZURE_AKS_CLUSTER_STAGE`

**Valor**: `az-k8s-stage-cluster`

## Verificación

Después de configurar los secrets, puedes probar el pipeline haciendo push a la rama `stage`.

## Notas Importantes

1. **El Service Principal debe tener rol "Contributor"** en la suscripción de Stage
2. **El JSON debe estar en formato SDK-auth** (por eso usamos `--sdk-auth`)
3. **No compartas las credenciales públicamente** - solo en GitHub Secrets
4. **Si el Service Principal ya existe**, Emely puede reutilizarlo o crear uno nuevo

## Troubleshooting

### Error: "Not all values are present"
- Verifica que el JSON esté completo
- Verifica que no haya saltos de línea
- Verifica que todas las comillas sean dobles

### Error: "Login failed"
- Verifica que el `clientId` y `tenantId` sean correctos
- Verifica que el `clientSecret` no haya expirado
- Verifica que el Service Principal tenga permisos en la suscripción

### Error: "Resource group not found"
- Verifica que el cluster esté desplegado en Azure Stage
- Verifica que el nombre del resource group sea correcto: `az-k8s-stage-rg`

