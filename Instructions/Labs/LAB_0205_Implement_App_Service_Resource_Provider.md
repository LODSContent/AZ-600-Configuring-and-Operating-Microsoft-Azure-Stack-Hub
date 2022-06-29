---
lab:
    title: 'Lab: Implement App Service Resource Provider in Azure Stack Hub'
    module: 'Module 2: Provide Services'
---

# Lab - Implement App Service Resource Provider in Azure Stack Hub
# Student lab manual

## Lab dependencies

- Implement SQL Server Resource Provider in Azure Stack Hub

## Estimated Time

4 hours

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to allow your tenants to deploy App Service apps and Azure functions.

## Objectives

After completing this lab, you will be able to:

 - Implement App Service resource provider in Azure Stack Hub.

## Lab Environment 

This lab uses an ADSK instance integrated with Active Directory Federation Services (AD FS) (backed up Active Directory as the identity provider). 

The lab environment has the following configuration:

- ASDK deployment running on the **AzS-HOST1** server with the following access points:

  - Administrator portal: https://adminportal.local.azurestack.external
  - Admin ARM endpoint: https://adminmanagement.local.azurestack.external
  - User portal: https://portal.local.azurestack.external
  - User ARM endpoint: https://management.local.azurestack.external

- Administrative users:

  - ASDK cloud operator username: **CloudAdmin@azurestack.local**
  - ASDK cloud operator password: **Pa55w.rd1234**
  - ASDK host administrator username: **AzureStackAdmin@azurestack.local**
  - ASDK host administrator password: **Pa55w.rd1234**

You will install software necessary to manage Azure Stack Hub via PowerShell in the course of this lab. 


### Exercise 1: Install the App Service resource provider in Azure Stack Hub

In this exercise, you will install the App Service resource provider in Azure Stack Hub.

1. Provision a SQL Server hosting server
1. Provision a file server
1. Install the App Service resource provider
1. Validate the installation of the App Service resource provider

>**Note**: To minimize the duration of this exercise, some of the tasks necessary to install the Azure Stack Hub App Service resource provider have already been completed, including the following:

- Implementing Azure Marketplace syndication
- Downloading of the following Azure Marketplace items:

  - **[smalldisk] Windows Server 2019 Datacenter Server Core-Bring your own license**
  - **Windows Server 2016 Datacenter-Bring your own license** 
  - **Custom Script Extension**


#### Task 1: Provision a SQL Server hosting server

In this task, you will:

- Provision a SQL Server hosting server

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the hub menu of the Azure Stack Hub administrator portal, click **Create a resource**.
1. On the **New** blade, select **Compute** and, in the list of available resource types, select **{WS-BYOL} Free SQL Server License: SQL Server 2017 Express on Windows Server 2016**.
1. On the **Basics** pane of the **Create virtual machine** blade, specify the following settings and click **OK** (leave others with their default values):

    - Name: **SqlHOST1**
    - VM disk type: **Premium SSD**
    - User name: **sqladmin**
    - Password: **Pa55w.rd**
    - Subscription: **Default Provider Subscription**
    - Resource group: the name of a new resource group **sql.resources-RG**
    - Location: **local**

1. On the **Choose a size** blade, select **DS1_v2** and click **Select**.
1. On the **Settings** pane of the **Create virtual machine** blade, set **Network Security Group** settings to **Advanced** and then click **Network security group (firewall)**.
1. On the **Create network security group** blade, click **+ Add an inbound rule**.
1. On the **Add inbound security rule** blade, specify the following settings and click **Add** (leave others with their default values):

    - Destination port ranges: **1433**
    - Protocol: **TCP**
    - Action: **Allow**
    - Priority: **200**
    - Name: **custom-allow-sql**

1. Back on the **Create network security group** blade, click **OK**.
1. Back on the **Settings** pane of the **Create virtual machine** blade, specify the following settings and click **OK** (leave others with their default values):

    - Boot diagnostics: Disabled
    - Guest OS diagnostics: Disabled

1. On the **SQL Server settings** pane of the **Create virtual machine** blade, specify the following settings and click **OK** (leave others with their default values):

    - SQL connectivity: **Public (Internet)**
    - Port: **1433**
    - SQL Authentication: **Enable**
    - Login name: **SQLAdmin**
    - Password: **Pa55w.rd**
    - Storage configuration: **General**
    - Automated patching: **Disable**
    - Automated backup: **Disable**
    - Azure Key Vault integration: **Disable**

1. On the **Summary** pane of the **Create virtual machine** blade, click **OK**.

    >**Note**: Wait for deployment to complete. This might take about 20 minutes.

1. Once the deployment completes, navigate to the **SqlHOST1** virtual machine blade and, in the **Overview** section, directly under the **DNS name** label, click **Configure**.
1. On the **SqlHOST1-ip \| Configuration** blade, in the **DNS name label (optional)** text box, type **sqlhost1** and click **Save**.

    >**Note**: This makes the **sqlhost1** available via **sqlhost1.local.cloudapp.azurestack.external** DNS name.

1. On the **sqlhost1-ip \| Configuration** blade, set the **Assignment** option to **Static** and click **Save**.

    >**Note**: This will trigger a restart of the **sqlhost1** virtual machine. Wait until the restart completes before you proceed to the next step.

1. Within the Remote Desktop session to **AzSHOST-1**, start a Remote Desktop session to **sqlhost1.local.cloudapp.azurestack.external** and, when prompted, sign in using the following credentials:

    - Username: **SQLAdmin**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **SqlHOST1**, right-click **Start** and, in the right-click menu, select **Command Prompt (Admin)**. 
1. Within the Remote Desktop session to **SqlHOST1**, from the **Administrator: Command Prompt**, run the following to start a SQLCMD session to the local SQL Server instance:

    ```
    sqlcmd
    ```

1. Within the Remote Desktop session to **SqlHOST1**, from the **Administrator: Command Prompt**, run the following to enable contained database authentication for SQL server:

    ```
    sp_configure 'contained database authentication', 1;
    GO
    RECONFIGURE;
    GO
    ```

    > **Note:** This is necessary in order to use this hosting server when implementing App Service Resource Provider later in this lab.

    > **Note:** Leave the Remote Desktop session to **sqlhost1.local.cloudapp.azurestack.external** open. You will use it later in this lab.


#### Task 2: Provision a file server

In this task, you will:

- Provision a file server

1. Switch to the Remote Desktop session to **AzSHOST-1** and, in the Azure Stack administrator portal, in the hub menu, click **All services**.
1. In the list of services, click **Marketplace management**
1. On the **Marketplace management - Marketplace items** blade, search for the **[smalldisk] Windows Server 2019 Datacenter Server Core-Bring your own license** item and ensure that it is available.
1. Within the Remote Desktop session to **AzSHOST-1**, open a new tab in a browser window and navigate to (https://aka.ms/appsvconmasdkfstemplate).
1. On the **AzureStack-QuickStart-Templates / appservice-fileserver-standalone** page, click **azuredeploy.json** and then click **Raw**.
1. Select the entire content of the page and copy it to Clipboard.
1. Switch back to the Azure Stack administrator portal and click **+ Create a resource**.
1. On the **New** blade, click **Custom** and then click **Template deployment**.
1. On the **Custom deployment** blade, select **Build your own template in the editor**. 
1. On the **Edit template** blade, replace the precreated template with the content of Clipboard.
1. On the **Custom Deployment** blade, click **Edit template**.
1. On the **Edit template** blade, in the **Parameters** section, set the following values:

    - **defaultValue** of **imageReference**: set to **MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest**
    - **allowedValues** of **imageReference**: set to **MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest**
    - **defaultValue** of **fileServerVirtualMachineSize**: set to **Standard_A1_v2**
    - **allowedValues** of **fileServerVirtualMachineSize**: set to **Standard_A1_v2**
    - **defaultValue** of **adminPassword**: set to **Pa55w.rd1234**
    - **defaultValue** of **fileShareOwnerPassword**: set to **Pa55w.rd1234**
    - **defaultValue** of **fileShareUserPassword**: set to **Pa55w.rd1234**

    > **Note:** This will result in the following content of the **Parameters** section:

    ```json
      "parameters": {
        "fileServerVirtualMachineSize": {
          "type": "string",
          "defaultValue": "Standard_A1_v2",
          "allowedValues": [
            "Standard_A1_v2",
          ],
          "metadata": {
            "description": "Size of vm"
          }
        },
        "imageReference": {
          "type": "string",
          "defaultValue": "MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest",
          "allowedValues": [
            "MicrosoftWindowsServer | WindowsServer | 2019-Datacenter-Core-smalldisk | latest"
          ],
          "metadata": {
            "description": "Please ensure the image is available. publisher: MicrosoftWindowsServer | offer: WindowsServer | sku: 2016-Datacenter"
          }
        },
        "dnsNameForPublicIP": {
          "type": "string",
          "defaultValue": "appservicefileshare",
          "maxLength": 63,
          "metadata": {
            "description": "Unique DNS Name for the Public IP used to access the file share.It must be lowercase. It should match the following regular expression, or it will raise an error: ^[a-z][a-z0-9-]{1,61}[a-z0-9]$"
          }
        },
        "adminUsername": {
          "type": "string",
          "defaultValue": "fileshareowner",
          "metadata": {
            "description": "File server Admin user"
          }
        },
        "adminPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "File server Admin password"
          }
        },
        "fileShareOwner": {
          "type": "string",
          "defaultValue": "fileshareowner",
          "metadata": {
            "description": "fileshare owner username"
          }
        },
        "fileShareOwnerPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "fileshare owner password"
          }
        },
        "fileShareUser": {
          "type": "string",
          "defaultValue": "fileshareuser",
          "metadata": {
            "description": "fileshare user"
          }
        },
        "fileShareUserPassword": {
          "type": "securestring",
          "defaultValue": "Pa55w.rd1234",
          "metadata": {
            "description": "fileshare user password"
          }
        },
        "vmExtensionScriptLocation": {
          "type": "string",
          "defaultValue": "https://raw.githubusercontent.com/Azure/azurestack-quickstart-templates/master/appservice-fileserver-standalone",
          "metadata": {
            "description": "File Server extension script Url"
          }
        }
      },
    ```

1. On the **Edit template** blade, in the **Variables** section, replace **"sku": "2016-Datacenter",** with **"sku": "2019-Datacenter-Core-smalldisk",** and select **Save**.
1. Back on the **Custom Deployment** blade, in the **Subscription** drop-down list, select **Default Provider Subscription** and, in the **Resource group** section, select **sql.resources-RG**.
1. On the **Custom deployment** blade, click **Review + create** and then click **Create**.

    > **Note:** Wait for the deployment to complete. This should take about 15 minutes.


#### Task 3: Install the App Service resource provider

In this task, you will:

- Install the App Service resource provider

1. Within the Remote Desktop session to **AzSHOST-1**, in the Azure Stack administrator portal, in the hub menu, click **All services**.
1. In the list of services, click **Marketplace management**.
1. On the **Marketplace management - Marketplace items** blade, search for the **Windows Server 2016 Datacenter-Bring your own license** item and ensure that it is available.
1. On the **Marketplace management - Marketplace items** blade, search for the **Custom Script Extension** item and ensure that it is available.
1. Within the Remote Desktop session to **AzSHOST-1**, start the web browser and navigate to (https://aka.ms/appsvconmasinstaller) to download **AppService.exe** and, once the download completes, copy the file to the **C:\\Downloads\\AppServiceRP** folder (create the folder if needed).
1. In the web browser, navigate to (https://aka.ms/appsvconmashelpers) to download **AppServiceHelperScripts.zip** and, once the download completes, extract its content to the **C:\\Downloads\\AppServiceRP** folder.
1. Within the Remote Desktop session to **AzSHOST-1**, start Windows PowerShell as administrator.
1. From the **Administrator: Windows PowerShell** window, run the following to create certificates required by App Service on Azure Stack:

    ```powershell
    Set-Location -Path C:\Downloads\AppServiceRP
    Get-ChildItem -Path '.\' -File -Recurse | Unblock-File

    $pfxPass = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    .\Create-AppServiceCerts.ps1 `
	-pfxPassword $pfxPass `
	-DomainName 'local.azurestack.external'
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to create an Azure Resource Manager root certificate for the Azure Stack Hub equired to install the App Service provider:

    ```
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'

    # Add the cloudadmin credential that's required for privileged endpoint access.
    $cloudAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $cloudAdminCreds = New-Object System.Management.Automation.PSCredential ("CloudAdmin@$domain", $cloudAdminPass)

    .\Get-AzureStackRootCert.ps1 -PrivilegedEndpoint $privilegedEndpoint -CloudAdminCredential $cloudAdminCreds
    ```

    > **Note:** The script creates in the local folder a file named AzureStackCertificationAuthority.cer, containing the Azure Resource Manager root certificate for the Azure Stack Hub.

1. From the **Administrator: Windows PowerShell** window, run the following to create an AD FS app required to install the App Service provider:

    ```
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'
    $adminArmEndpoint = 'adminmanagement.local.azurestack.external'
    $certificateFilePath = 'C:\Downloads\AppServiceRP\sso.appservice.local.azurestack.external.pfx'
    $certificatePassword = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    .\Create-ADFSIdentityApp.ps1 `
       -AdminArmEndpoint $adminArmEndpoint `
       -PrivilegedEndpoint $privilegedEndpoint `
       -CloudAdminCredential $cloudAdminCreds `
       -CertificateFilePath $certificateFilePath `
       -CertificatePassword $certificatePassword
    ```

1. In the output of the script, copy the GUID representing the ID of the generated AD FS application. 

    > **Note:** Make sure to record this GUID. You will need it later in this task.

1. From the **Administrator: Windows PowerShell** window and, from the **Administrator: Windows PowerShell** window,  run the following to start AppService.exe:

    ```
    .\AppService.exe
    ```

    > **Note:** This will start the Microsoft Azure App Service Setup wizard.

1. Click **Deploy App Service or upgrade to the latest version**.
1. On the **MICROSOFT SOFTWARE SUPPLEMENTARY LICENSE TERMS** page, review the content, click the checkbox **I have read, understood, and agreed to these license terms** checkbox and click **Next**.
1. On the page displaying the third party license terms, review the content, click the checkbox **I have read, understood, and agreed to these license terms checkbox** and click **Next**.
1. On the page displaying the Admin and Tenant ARM endpoints, verify that the information is correct and click **Next**.
1. On the Azure Stack App Service cloud information page, ensure that the **Credential** option is selected and click **Connect**.
1. When prompted, sign in as **CloudAdmin@AzureStack.local** with the **Pa55w.rd1234** password.
1. Back on the Azure Stack App Service cloud information page, in the **Azure Stack Subscriptions** drop-down list, select the **Default Provider Subscription**, in the **Azure Stack Locations** drop-down list, select **local** and click **Next**.
1. On the **Virtual Network Configuration**, accept the default settings and click **Next**.
1. On the next page, specify the following information and click **Next**:

    - File Share UNC Path: **\\appservicefileshare.local.cloudapp.azurestack.external\websites**
    - File Share Owner: **fileshareowner**
    - File Share Owner Password: **Pa55w.rd1234**
    - File Share User: **fileshareuser**
    - File Share User Password: **Pa55w.rd1234**

1. On the next page, specify the settings that identify the application ID you generated earlier in this task and click **Next**:

    - Identity Application Id: the GUID you copied earlier in this task
    - Identity Application certificate file (*.pfx): **C:\Downloads\AppServiceRP\sso.appservice.local.azurestack.external.pfx**
    - Identity Application certificate file (*.pfx) password: **Pa55w.rd1234pfx**
    - Azure Resource Manager (ARM) root certificate file (*.cer): **C:\Downloads\AppServiceRP\AzureStackCertificationAuthority.cer**

1. On the next page, specify the settings that identify the certificate files and their respective passwords:

    - App Service default SSL certificate file (*.pfx): **C:\Downloads\AppServiceRP\_.appservice.local.azurestack.external.pfx**
    - App Service default SSL certificate (*.pfx) password: **Pa55w.rd1234pfx**
    - App Service API SSL certificate file (*.pfx): **C:\Downloads\AppServiceRP\api.appservice.local.azurestack.external.pfx**
    - App Service API SSL certificate (*.pfx) password: **Pa55w.rd1234pfx**
    - App Service Publisher certificate file (*.pfx): **C:\Downloads\AppServiceRP\ftp.appservice.local.azurestack.external.pfx**
    - App Service Publisher SSL certificate (*.pfx) password: **Pa55w.rd1234pfx**

1. On the next page, specify the SQL Server settings:

    - SQL Server Name: **sqlhost1.local.cloudapp.azurestack.external**
    - SQL sysadmin login: **SQLAdmin**
    - SQL sysadmin password: **Pa55w.rd1234**

1. On the next page, specify the number and SKU of instances of the App Service deployment:

    - Controller Role: **2 instances - Standard_A1_v2 - [1 Core(s), 2048 MB]**
    - Management Role: **1 instance - Standard_A2_v2 - [2 Core(s), 4096 MB]**
    - Publisher Role: **1 instance - Standard_A1_v2 - [1 Core(s), 2048 MB]**
    - FrontEnd Role: **1 instance - Standard_A1_v2 - [1 Core(s), 2048 MB]**
    - Shared Worker Role: **1 instance - Standard_A1_v2 - [1 Core(s), 2048 MB]**

1. click **Next**.
1. On the next page, in the **Select Platform Image** drop-down list, select the **2016 Datacenter - latest** image and click **Next**.
1. On the next page, specify the following Admin credentials for the deployment:

    - Worker Role Virtual Machine(s) Admin: **SAWorkerAdmin**
    - Worker Role Virtual Machine(s) Password: **Pa55w.rd1234**
    - Confirm Password: **Pa55w.rd1234**
    - Other Roles Virtual Machine(s) Admin: **SAORoleAdmin**
    - Other Roles Virtual Machine(s) Password: **Pa55w.rd1234**
    - Confirm Password: **Pa55w.rd1234**

1. Click **Next**.
1. On the Summary page, click the checkbox **Select and click next to start the deployment** and, to start the deployment, click **Next**.

    > **Note:** Wait for the installation to complete. This might take 2-3 hours.

1. Once the installation completes, click **Exit**.


#### Task 4: Validate the installation of the App Service resource provider

In this task, you will:

- Validate the installation of the App Service resource provider

1. Within the Remote Desktop session to **AzSHOST-1**, in the web browser displaying the Azure Stack administrator portal, in the hub menu, select **All services**, on the **All services** blade, select **Administration**, and, in the list of services, click **App Service**. 

    > **Note:** You might need to refresh the browser page for the **App Service** entry to become available.

1. On the **App Service** blade, in the **Essentials** section, verify that the **All roles are ready** message appears under the **Status** label.

    > **Note:** Wait until all roles are successfully started. This might take additional 15-20 minutes.

>**Review**: In this exercise, you have installed the App Service resource provider on Azure Stack Hub.


### Exercise 2: Explore management tasks of App Service resource provider on Azure Stack Hub

In this exercise, you will explore management tasks of App Service resource provider on Azure Stack Hub.

1. Explore the scaling functionality of App Service resouces
1. Explore backup settings of App Service resource provider


### Task 1: Explore the scaling functionality of App Service resources

In this task, you will:

- Review the scaling functionality of App Service resources
- Review the backup settings of App Service resource provider

1. Within the Remote Desktop session to **AzSHOST-1**, in the web browser displaying the Azure Stack administrator portal, on the **App Service** blade, click **Roles**.
1. On the **App Service | Roles** blade, review the list of roles and the corresponding instances.
1. On the **App Service | Roles** blade, in the **Controller** row, click the ellipsis symbol on the right side and, in the drop-down list, note the **Virtual machine** entry.

    > **Note:** The Controller role is implemented by using virtual machines, which is the reason for the choice of 2 instances of the Controller during the installation of the App Service resource provider.

1. On the **App Service | Roles** blade, in the remaining rows, click the ellipsis symbol on the right side and, in the drop-down list, note the **ScaleSet** entry.

    > **Note:** All other roles are implemented by using scale sets, which allows for scaling.

1. On the **App Service | Roles** blade, note that you currently only have Web Worker roles in the **Shared** worker tier. 
1. On the **App Service | Roles** blade, in the vertical menu on the left, select **Worker Tiers**.
1. On the **App Service | Worker Tiers** blade, click **+ Add**. 
1. On the **Create** blade, review the available options, including the **Compute Mode** drop-down list that allows you to choose between **Shared** and **Dedicated**.

    > **Note:** You have the ability to deploy virtual machines in a range of sizes with custom software as virtual machines in the worker tier of your choice.

1. Close the **Create** blade without making any changes.

    > **Note:** The provisioning process might take over an hour.


### Task 2: Explore the backup settings of the App Service resource provider

In this task, you will:

- Review backup settings of the App Service resource provider.

> **Note:** App Service backup on Azure Stack Hub consists of the following main components

- The resource provider infrastructure
- The resource providre secrets 
- The SQL Server instance hosting the metering database
- The user workload content stored on the App Service file share

    > **Note:** The resource provider infrastructure configuration can be recreated from backup during recovery using App Service recovery PowerShell cmdlets. For details regarding the recovery process, refer to [App Service recovery on Azure Stack Hub](https://docs.microsoft.com/en-us/azure-stack/operator/app-service-recover?view=azs-2008).

1. Within the Remote Desktop session to **AzSHOST-1**, in the web browser displaying the Azure Stack administrator portal, on the **App Service** blade, click **Secrets**.
1. On the **App Service \| Secrets** blade, click **Download Secrets** and then click **Save**.
1. Verify that the **SystemSecrets.json** file was downloaded to the **Downloads** folder on **AzSHOST-1**.

    > **Note:** You should copy the **SystemSecrets.json** file to a safe location and repeat this process whenever the secrets are rotated. 

    > **Note:** The recommended approach to back up the **Appservice_hosting** and **Appservice_metering** databases involves the use of SQL Server maintenance plans of Azure Backup Server, but you can also back them up by using the SQL Server PowerShell module cmdlets.

1. Within the Remote Desktop session to **AzSHOST-1**, switch to the Remote Desktop session to **sqlhost1.local.cloudapp.azurestack.external**. 
1. Within the Remote Desktop session to **SqlHOST1**, start Windows PowerShell as Administrator.
1. Within the Remote Desktop session to **SqlHOST1**, from the **Administrator: Windows PowerShell** prompt, run the following to perform a local backup of the App Service databases:

    ```powershell
    $date = Get-Date -Format 'yyyyMMdd'
    New-Item -ItemType Directory -Path 'C:\Backups'
    Backup-SqlDatabase -ServerInstance 'localhost' -Database 'appservice_hosting' -BackupFile "C:\Backups\appservice_hosting_$date.bak" -CopyOnly
    Backup-SqlDatabase -ServerInstance 'localhost' -Database 'appservice_metering' -BackupFile "C:\Backups\appservice_metering_$date.bak" -CopyOnly
    ```

    > **Note:** App Service stores tenant app information on its designated file share. The recommended approach to back up the file share involves the use of Azure Backup Server, but you can use for this purpose any file copy utility.

1. Switch to the Remote Desktop session to **AzSHOST-1** and, within the Remote Desktop session to **AzSHOST-1**, in the web browser displaying the Azure Stack administrator portal, on the **App Service** blade, click **System configuration**.
1. In the web browser displaying the Azure Stack administrator portal, on the **App Service \| System configuration** blade, note the full path of the **File share** (**\\\\appservicefileshare.local.cloudapp.azurestack.external\\websites**)
1. Switch to the Remote Desktop session to **AzSHOST-1** and from the **Administrator: Windows PowerShell** window, run the following to copy the content of the App Service file share to the local file system:

    ```powershell
    $source = '\\appservicefileshare.local.cloudapp.azurestack.external\websites'
    $date = Get-Date -Format 'yyyyMMdd'
    $destination = "C:\Backups\FileShare\$date"
    New-Item -ItemType Directory -Path $destination -Force

    $fileshareusername = 'fileshareowner'
    $fileshareuserpassword = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $fileshareusercreds = New-Object System.Management.Automation.PSCredential ($fileshareusername, $fileshareuserpassword)
    New-PSDrive -Name 'S' -Root $source -PSProvider 'FileSystem' -Credential $fileshareusercreds
    robocopy $source $destination /e /r:1 /w:1
    Remove-PSDrive -Name 'S'
    ```

>**Review**: In this exercise, you have explored management tasks of App Service resource provider on Azure Stack Hub.


### Exercise 3: Deliver App Service resources in Azure Stack Hub

In this exercise, you will deliver App Service resources to Azure Stack Hub users.

1. Make App Service resources available to users (as a cloud operator)
1. Create a Web app (as a user)


### Task 1: Make App Service resources available to users (as a cloud operator)

In this task, you will:

- Make App Service resources available to users (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, switch to the browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/).
1. In the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**.
1. On the **New** blade, click **Offers + Plans** and then click **Plan**.
1. On the **Basics** tab of the **New plan** blade, specify the following settings:

    - Display name: **app-service-plan1**
    - Resource name: **app-service-plan1**
    - Resource group: the name of a new resource group **app-service-plans-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.Web** checkbox.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, select **Create New** and on the **Create** blade, specify the following settings, and click **OK**:

    - Name: **app-service-quota1**
    - App Service Plans: **Custom** **20**
    - Shared App Service Plans: **Custom** **10**
    - Dedicated App Service Plans: **Custom** **10**
    - Pricing SKUs: **Custom** **2 selected** (**Free** and **Shared**)
    - Consumption Plan: **Enabled**

    >**Note**: To offer Azure Functions in the consumption plan model, you need to deploy shared web workers.

1. Back on the **Quotas** tab of the **New plan** blade, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, back on the **New** blade, click **Offer**.
1. On the **Basics** tab of the **Create a new offer** blade, specify the following settings:

    - Display name: **app-service-offer1**
    - Resource name: **app-service-offer1**
    - Resource group: **app-service-offers-RG**
    - Make this offer public: **Yes**

1. Click **Next: Base plans >**. 
1. On the **Base plans** tab of the **Create a new offer** blade, select the checkbox next to the **app-service-plan1** entry.
1. Click **Next: Add-on plans >**.
1. Leave **Add-on plans** settings with their default values, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.


#### Task 2: Create a Web app (as a user)

In this task, you will:

- Create a test user account
- Create a Web app (as a user)

1. Within the Remote Desktop session to **AzS-HOST1**, click **Start**, in the Start menu, click **Windows Administrative Tools**, and, in the list of administrative tools, double-click **Active Directory Administrative Center**.
1. In the **Active Directory Administrative Center** console, click **azurestack (local)**.
1. In the details pane, double-click the **Users** container.
1. In the **Tasks** pane, in the **Users** section, click **New -> User**.
1. In the **Create User** window, specify the following settings and click **OK**: 

    - Full name: **T1U1**
    - User UPN logon: **t1u1@azurestack.local**
    - User SamAccountName: **azurestack\t1u1**
    - Password: **Pa55w.rd**
    - Password options: **Other password options -> Password never expires**

1. Within the Remote Desktop session to **AzS-HOST1**, start an InPrivate session of the web browser.
1. In the web browser window, navigate to the [Azure Stack Hub user portal](https://portal.local.azurestack.external) and sign in as **t1u1@azurestack.local** with the password **Pa55w.rd**.
1. In the Azure Stack Hub user portal, on the Dashboard, click the **Get a subscription** tile.
1. On the **Get a subscription** blade, in the **Name** text box, type **t1u1-app-service-subscription1**.
1. In the list of offers, select **app-service-offer1** and click **Create**.
1. When presented with the message **Your subscription has been created. You must refresh the portal to start using your subscription**, click **Refresh**. 
1. In the Azure Stack Hub tenant portal, in the hub menu, click **+ Create a resource**.
1. In the list of services, click **Web + Mobile** and then click **Web App**. 
1. On the **Web App** blade, specify the following settings:

    - Subscription: **t1u1-app-service-subscription1**
    - App name: **t1u1webapp1**
    - Resource Group: the name of a new resource group **webapps-RG**

1. On the **Web App** blade, click **App Service plan/Location** and, on the **App Service plan** blade, click **+ Create new**. 
1. On the **New App Service plan** blade, specify the following settings:

    - App Service plan: **appserviceplan1**
    - Location: **local**

1. On the **New App Service plan** blade, click **Pricing tier**.
1. On the **Spec Picker** blade, select the **F1** pricing tier and click **Apply**.
1. Back on the **New App Service plan** blade, click **OK**.
1. Back on the **Web App** blade, click **Create**.

    >**Note**: Wait for the deployment to complete. This should take less than a minute.

1. Within the Remote Desktop session to **AzS-HOST1**, in the InPrivate session of the web browser displaying the Azure Stack Hub user portal, in the hub menu, select **All resources**.
1. On the **All resources** blade, in the subscription filter drop down list, select the **t1u1-app-service-subscription1** entry and then click **Refresh**.
1. On the **All resources** blade, in the list of resources, click the **t1u1webapp1** entry.
1. On the **t1u1webapp1** blade, click **Browse**.

    >**Note**: This should open another browser tab displaying the default home page of the newly provisioned web app.

>**Review**: In this exercise, you have made the App Service available to users and created a Web app as a tenant user.
