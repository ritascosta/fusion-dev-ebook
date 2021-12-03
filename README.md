# Fusion Development Workshop

## Prerequisites

You will need the following software on your computer:

- PowerShell
- .NET 5.0
- A Git client
- Visual Studio Code together with the following extensions:
    - The C# extension, available at https://aka.ms/AAbvu72.  
    - The Azure App Service extension, available at https://aka.ms/AAbvgm8. 
    - The Azure Account extension, available at https://aka.ms/AAbvgm7.  
    - The The SQL Server extension, available at https://aka.ms/AAbvgm5.  


You'll also need an Azure subscription and a PowerApps subscription.

## Clone the sample code on your local computer

To view and edit the code in this project, use `git` to clone this project on your hard drive:

1. Start PowerShell and navigate to a directory where you want to keep the source code.
1. To clone the source code, enter this command:

    ```powershell
    git clone https://github.com/ritascosta/fusion-dev-ebook
    cd fusion-dev-ebook
    ```

## Set up an Azure SQL Database server

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

## Configure the Web API

You can either follow the steps in the guide to build the Web API, or you can use the completed version provided in this repository. To build and deploy the completed version, perform the following instructions:

1. In your local **PowerShell** instance, start Visual Studio Code in the WebAPI folder:

    ```powershell
    cd WebAPI
    code .
    ```

1. Open **appsettings.json**. There are three connections strings in the file. In each connection string, replace **fieldengineersqlserver** with the name of the SQL server you created above. Replace the password in each connection string with **Pa55w.rd**
1. Open **appsettings.Development.json**. In each connection string, replace **fieldengineersqlserver** with the name of the SQL server you created above, and provide the password **Pa55w.rd**.

### Deploy the Web API in the Azure Container Apps Service

1. In Visual Studio Code, press **CTRL + SHIFT + P**, type **Azure: Sign-In** and then press **Enter**.
1. In the browser window, sign into Azure with your usual credentials, and then close the browser window.
1. To open the Azure Tools, in Visual Studio Code press **CTRL + SHIFT + A** and then select the **APP SERVICE** window.
1. Right-click your subscription, and then select **Create New Web app**.
1. Enter a name for the Web app and then press **Enter**.
1. Select the **.NET 5** runtime stack and then select the **Free** pricing tier. Azure creates the new Web app.
1. Right-click the new Web app, and then select **Deploy to Web App**.
1. In the Visual Studio Code dialog, select **Deploy**. Visual Studio Code deploys the Web API to Azure.
