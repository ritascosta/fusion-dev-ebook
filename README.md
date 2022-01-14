## Fusion Development Workshop

## Prerequisites

You will need the following software on your computer:

- PowerShell
- .NET 5.0
- A Git client
- Visual Studio Code 

You'll also need an Azure subscription and a PowerApps subscription.

## Step-by-step guide

- [Workshop overview](#overview)
- [Clone the sample code on your local computer](#repo)
- [Set up an Azure SQL Database server](#sql)
- [Configure the Web API and deploy it to Azure Container Apps](#api)
- [Expose your Web API with Azure API Management](#apim)
- [Build an interface for your Web API with Power Apps](#powerapps)

## Workshop Overview <a name="overview"></a>

## Clone the sample code on your local computer <a name="repo"></a>

To view and edit the code in this project, use `git` to clone this project on your hard drive:

1. Start PowerShell and navigate to a directory where you want to keep the source code.
1. To clone the source code, enter this command:

    ```powershell
    git clone https://github.com/ritascosta/fusion-dev-ebook
    cd fusion-dev-ebook
    ```

## Set up an Azure SQL Database server <a name="sql"></a>

The Web API requires three SQL databases:

- InventoryDB. This database stores information about boiler parts and their stock numbers.
- KnowledgeBaseDB. This database stores technical tips on each boiler part.
- ScheduleDB. This database stores appointment details.

You can set up these databases in Azure SQL Database by following these steps:

1. Long into the [Azure Portal](https://portal.azure.com) with your usual credentials.
1. Start the **Cloud Shell** and select the **PowerShell** environment.
1. To clone the source code in the Cloud Shell, enter these commands:

    ```powershell
    git clone https://github.com/microsoft/fusion-dev-ebook
    cd fusion-dev-ebook
    ```

1. To create an Azure SQL Database server, run this command. When prompted, enter a resource group name (this resource group will be created), SQL server name, and an Azure location, such as **westus**:

    ```powershell
    .\databasesetup.ps1
    ```

1. In the Azure Portal, and navigate to **All resources**. Select the SQL Server you created in the previous step.
1. Under **Security** select, **Firewalls and virtual networks**.
1. Select **Add client IP**.
1. Under **Allow Azure services and resources to access this server**, select **Yes**, and then select **Save**.
1. In the Cloud Shell, to create the databases, run these commands, substituting `<yourservername>` with the name of the SQL Server you just created:

    ```powershell
    sqlcmd -S <yourservername>.database.windows.net -U sqladmin -P Pa55w.rd -i "./SQLScripts/CreateAllDBs.sql"
    sqlcmd -S <yourservername>.database.windows.net -U sqladmin -P Pa55w.rd -d InventoryDB -i "./SQLScripts/InventoryDB-setup.sql"
    sqlcmd -S <yourservername>.database.windows.net -U sqladmin -P Pa55w.rd -d KnowledgeDB -i "./SQLScripts/KnowledgeDB-setup.sql"
    sqlcmd -S <yourservername>.database.windows.net -U sqladmin -P Pa55w.rd -d SchedulesDB -i "./SQLScripts/SchedulesDB-setup.sql"
    ```

## Configure the Web API <a name="api"></a>

You can either follow the steps in the guide to build the Web API, or you can use the completed version provided in this repository. To build and deploy the completed version, perform the following instructions:

1. In your local **PowerShell** instance, start Visual Studio Code in the WebAPI folder:

    ```powershell
    cd WebAPI
    code .
    ```

1. Open **appsettings.json**. There are three connections strings in the file. In each connection string, replace **fieldengineersqlserver** with the name of the SQL server you created above. Replace the password in each connection string with **Pa55w.rd**
1. Open **appsettings.Development.json**. In each connection string, replace **fieldengineersqlserver** with the name of the SQL server you created above, and provide the password **Pa55w.rd**.

### Deploy the Web API into Azure Container Apps Service 

If you find it more suitable, you can deploy your Web API as a Web App in Azure App Service. In this case we will deploy it as a container to Azure Containers Apps. 

1. Containerize your application using the **Dockerfile** in the Web API folder

    ```powershell
    docker ...
    ```
1. Add your newly created container image to an **Azure Container Registry** (create new if you don't have one).

    ```powershell
    $ACR_NAME="aircond"
    az acr login --name $ACR_NAME
    docker tag fieldapi $ACR_NAME/fieldapi:v1
    docker push $ACR_NAME.azurecr.io/fieldapi:v1
    ```    
1. Create a **Container Apps Environment** to host your Container App.

    ```powershell
    az monitor log-analytics workspace create --resource-group $RESOURCE_GROUP --workspace-name $LOG_ANALYTICS_WORKSPACE

    $LOG_ANALYTICS_WORKSPACE_CLIENT_ID=(az monitor log-analytics workspace show --query customerId -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv)
    $LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET=(az monitor log-analytics workspace get-shared-keys --query primarySharedKey -g $RESOURCE_GROUP -n $LOG_ANALYTICS_WORKSPACE --out tsv)

    az containerapp env create `
      --name $CONTAINERAPPS_ENVIRONMENT `
      --resource-group $RESOURCE_GROUP `
      --logs-workspace-id $LOG_ANALYTICS_WORKSPACE_CLIENT_ID `
      --logs-workspace-key $LOG_ANALYTICS_WORKSPACE_CLIENT_SECRET `
      --location "$LOCATION"
    ```   
1. Deploy your Docker image as a Container App in your recently created Container App Environment

    ```powershell
    az containerapp create `
      --name my-container-app-fieldapi `
      --resource-group $RESOURCE_GROUP `
      --environment $CONTAINERAPPS_ENVIRONMENT `
      --image aircond.azurecr.io/fieldapi:v1 `
      --target-port 80 `
      --ingress 'external' `
      --query configuration.ingress.fqdn
      ```
1. Access your Web API throught the Application URL followed by "/swagger" and test your connection the the SQL database.

Example: https://fieldapi.victorioushill-21a56854.canadacentral.azurecontainerapps.io/swagger

## Expose your Web API through API Management <a name="apim"></a>

## Build an interface for your Web API with Power Apps <a name="powerapps"></a> 
