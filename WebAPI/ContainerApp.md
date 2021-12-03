**Add container image to ACR**
$ACR_NAME="aircond"
az acr login --name $ACR_NAME
docker tag fieldapi $ACR_NAME/fieldapi:v1
docker push $ACR_NAME.azurecr.io/fieldapi:v1

**Pre Requisites**
az extension add --source https://workerappscliextension.blob.core.windows.net/azure-cli-extension/containerapp-0.2 0-py2.py3-none-any.whl
az provider register --namespace Microsoft.Web

$RESOURCE_GROUP="my-containerapps"
$LOCATION="canadacentral"
$LOG_ANALYTICS_WORKSPACE="containerapps-logs"
$CONTAINERAPPS_ENVIRONMENT="containerapps-env"

az group create --name $RESOURCE_GROUP --location "$LOCATION"

**Create an environment**
az monitor log-analytics workspace create --resource-group $RESOURCE_GROUP --workspace-name $LOG_ANALYTICS_WORKSPACE

$LOG_ANALYTICS_WORKSPACE_CLIENT_ID=(az monitor log-analytics workspace show --query customerId -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv)

$LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET=(az monitor log-analytics workspace get-shared-keys --query primarySharedKey -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv)

az containerapp env create `
  --name $CONTAINERAPPS_ENVIRONMENT `
  --resource-group $RESOURCE_GROUP `
  --logs-workspace-id $LOG_ANALYTICS_WORKSPACE_CLIENT_ID `
  --logs-workspace-key $LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET `
  --location "$LOCATION"

**Create Container App**
az containerapp create `
  --name my-container-app-fieldapi `
  --resource-group $RESOURCE_GROUP `
  --environment $CONTAINERAPPS_ENVIRONMENT `
  --image aircond.azurecr.io/fieldapi:v1 `
  --target-port 80 `
  --ingress 'external' `
  --query configuration.ingress.fqdn



