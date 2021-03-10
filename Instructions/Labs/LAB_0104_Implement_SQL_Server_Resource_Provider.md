---
lab:
    title: 'Lab: Implement SQL Server Resource Provider in Azure Stack Hub'
    module: 'Module 1: Provide Services'
---

# Lab - Implement SQL Server Resource Provider in Azure Stack Hub
# Student lab manual

## Lab dependencies

- None

## Estimated Time

150 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to allow your tenants to deploy SQL Server databases. 

## Objectives

After completing this lab, you will be able to:

- Implement SQL Server resource provider in Azure Stack Hub.

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


### Exercise 1: Install the SQL Server resource provider in Azure Stack Hub

In this exercise, you will install the SQL Server resource provider in Azure Stack Hub.

1. Download SQL Server resource provider binaries 
1. Install the SQL Server resource provider
1. Verify installation of the SQL Server resource provider

>**Note**: To minimize the duration of this exercise, some of the tasks necessary to install the Azure Stack Hub SQL Server resource provider have already been completed, including the following:

- Implementing Azure Marketplace syndication
- Downloading **Microsoft AzureStack Add-On RP Windows Server** from Azure Marketplace

#### Task 1: Download SQL Server resource provider binaries

In this task, you will:

- Download SQL Server resource provider binaries

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. Within the Remote Desktop session to **AzSHOST-1**, in the web browser displaying the Azure Stack administrator portal, in the hub menu, click **All services**.
1. On the **All services** blade, search for and select **Marketplace management**
1. On the Marketplace management blade, verify that **Microsoft AzureStack Add-On RP Windows Server** appears in the list of available services.
1. Within the Remote Desktop session to **AzSHOST-1**, start another web browser window, download the SQL Resource Provider self-extracting executable from (https://aka.ms/azshsqlrp11931) and extract its content into the **C:\\Downloads\\SQLRP** folder (you will need to create the folder first).


#### Task 2: Install the SQL Server resource provider

In this task, you will:

- Install SQL Server resource provider

1. Within the Remote Desktop session to **AzSHOST-1**, start Windows PowerShell as administrator.

    > **Note:** Make sure to start a new PowerShell session.

1. From the **Administrator: Windows PowerShell** prompt, run the following to configure PowerShell Gallery as a trusted repository

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. From the **Administrator: Windows PowerShell** prompt, run the following to install the version of the AzureRM.Bootstrapper module required by the SQL Server resource provider:

    ```powershell
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose

    Install-Module -Name AzureRm.BootStrapper -RequiredVersion 0.5.0 -Force
    Install-Module -Name AzureStack -RequiredVersion 1.6.0
    ```

1. From the **Administrator: Windows PowerShell** prompt, run the following to register your Azure Stack Hub operator PowerShell environment:

    ```powershell
    Add-AzureRmEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

1. From the **Administrator: Windows PowerShell** prompt, run the following to set the current environment:

    ```powershell
    Set-AzureRmEnvironment -Name 'AzureStackAdmin'
    ```

1. From the **Administrator: Windows PowerShell** prompt, run the following to authenticate to the current environment (when prompted, sign in as the **CloudAdmin@azurestack.local** user with the **Pa55w.rd1234** as its password):

    ```powershell
    Connect-AzureRmAccount -EnvironmentName 'AzureStackAdmin'
    ```

1. From the **Administrator: Windows PowerShell** prompt, run the following to verify that the authentiation was successful and the corresponding context is set:

    ```powershell
    Get-AzureRmContext
    ```

1. From the **Administrator: Windows PowerShell** prompt, run the following to set the variables necessary to install the SQL Server Resource Provider:

    ```powershell
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'
    $downloadDir = 'C:\Downloads\SQLRP'

    # Set the AzureStack\AzureStackAdmin credentials
    $serviceAdmin = 'AzureStackAdmin@azurestack.local'
    $serviceAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $serviceAdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $serviceAdminPass)

    # Set the AzureStack\CloudAdmin credentials
    $cloudAdminName = 'AzureStack\CloudAdmin'
    $cloudAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $cloudAdminCreds = New-Object PSCredential($cloudAdminName, $cloudAdminPass)

    # Set credentials for the new resource provider VM local admin account
    $vmLocalAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ('sqlrpadmin', $vmLocalAdminPass)

    # Set a password that will protect the private key of a self-signed certificate generated to secure the SQL Server resource provider
    $pfxPass = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    # Update the PowerShell module path environment variable to include the SQL Server resource provider modules
    $rpModulePath = Join-Path -Path $env:ProgramFiles -ChildPath 'SqlMySqlPsh'
    $env:PSModulePath = $env:PSModulePath + ';' + $rpModulePath 
    ```

1. From the **Administrator: Windows PowerShell** prompt, change the current directory to the location of the extracted SQL Server resource provider installation files and run the DeploySQLProvider.ps1 script:

    ```powershell
    Set-Location -Path 'C:\Downloads\SQLRP'

    .\DeploySQLProvider.ps1 `
        -AzCredential $serviceAdminCreds `
        -VMLocalCredential $vmLocalAdminCreds `
        -CloudAdminCredential $cloudAdminCreds `
        -PrivilegedEndpoint $privilegedEndpoint `
        -DefaultSSLCertificatePassword $pfxPass
    ```

    > **Note:** Wait for the installation to complete. This might take about an hour.

#### Task 3: Verify installation of the SQL Server resource provider

In this task, you will:

- Verify installation of the SQL Server resource provider

1. Within the Remote Desktop session to **AzS-HOST1**, switch to the web browser window displaying the Azure Stack administrator portal and, in the hub menu, click **Resource groups**. 
1. On the **Resource groups** blade, click **system.local.sqladapter**.
1. On the **system.local.sqladapter** blade, review the **Deployments** entry and verify that all deployments were successful.
1. In the Azure Stack administrator portal, navigate to the **Virtual machines** and verify that the SQL resource provider VM was successfully created and is running.
1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub tenant portal](https://portal.local.azurestack.external/) and, if prompted, sign in as CloudAdmin@azurestack.local.
1. In the hub menu of the Azure Stack Hub tenant portal, click **Create a resource**.
1. On the **New** blade, select **Data + Storage** and verify that **SQL Database** appears in the list of available resource types.

>**Review**: In this exercise, you have installed the SQL Server resource provider in Azure Stack Hub


### Exercise 2: Configure SQL Server resource provider in Azure Stack Hub

In this exercise, you will configure SQL Server resource provider in Azure Stack Hub.

1. Create a plan, offer, and subscription for a SQL Server hosting server (as a cloud operator)
1. Deploy an Azure Stack Hub VM that will become a SQL Server hosting server (as a cloud operator)
1. Add a SQL hosting server (as a cloud operator)
1. Make SQL databases available to users (as a cloud operator)
1. Crate a SQL database (as a user)

>**Note**: To minimize the duration of this exercise, some of the tasks that facilitate configuration of the Azure Stack Hub SQL Server resource provider have already been completed, including the following:

- Downloading a SQL Server image from Azure Marketplace
- Downloading **Sql IaaS VM extension** from Azure Marketplace


#### Task 1: Create a plan, offer, and subscription for a SQL Server hosting server (as a cloud operator)

In this task, you will:

- Create a plan, offer, and subscription for SQL Server hosting server (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**.
1. On the **New** blade, click **Offers + Plans** and then click **Plan**.
1. On the **Basics** tab of the **New plan** blade, specify the following settings:

    - Display name: **sql-server-hosting-plan1**
    - Resource name: **sql-server-hosting-plan1**
    - Resource group: the name of a new resource group **sql-server-hosting-plans-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.Compute**, **Microsoft.Storage**, and **Microsoft.Network** checkboxes.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, specify the following settings:

    - Microsoft.Compute: **Default Quota**
    - Microsoft.Network: **Default Quota**
    - Microsoft.Storage: **Default Quota**

1. Click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, back on the **New** blade, click **Offer**.
1. On the **Basics** tab of the **Create a new offer** blade, specify the following settings:

    - Display name: **sql-server-hosting-offer1**
    - Resource name: **sql-server-hosting-offer1**
    - Resource group: **sql-server-hosting-offers-RG**
    - Make this offer public: **No**

1. Click **Next: Base plans >**. 
1. On the **Base plans** tab of the **Create a new offer** blade, select the checkbox next to the **sql-server-hosting-plan1** entry.
1. Click **Next: Add-on plans >**.
1. Leave **Add-on plans** settings with their default values, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, back on the **New** blade, click **Subscription**.
1. On the **Create a user subscription** blade, specify the following settings and click **Create**.

    - Name: **sql-server-hosting-subscription1**
    - User: **cloudadmin@azurestack.local**
    - Directory tenant: **ADFS.azurestack.local**
    - Offer name: **sql-server-hosting-offer1**

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

1. Leave the Azure Stack Hub administrator portal window open.


#### Task 2: Deploy an Azure Stack Hub VM that will become a SQL Server hosting server (as a cloud operator)

In this task, you will:

- Deploy an Azure Stack Hub VM that will become a SQL Server hosting server (as a cloud operator)

    >**Note**: An Azure Stack Hub VM operating as a SQL Server hosting server should created in a billable user subscription.

1. Within the Remote Desktop session to **AzS-HOST1**, switch to the web browser window displaying the [Azure Stack Hub tenant portal](https://portal.local.azurestack.external/).
1. In the hub menu of the Azure Stack Hub tenant portal, click **Create a resource**.
1. On the **New** blade, select **Compute** and, in the list of available resource types, select **{WS-BYOL} Free SQL Server License: SQL Server 2017 Express on Windows Server 2016**.
1. On the **Basics** pane of the **Create virtual machine** blade, specify the following settings and click **OK** (leave others with their default values):

    - Name: **sql-host-vm0**
    - VM disk type: **Premium SSD**
    - User name: **sqladmin**
    - Password: **Pa55w.rd**
    - Subscription: **sql-server-hosting-subscription1**
    - Resource group: the name of a new resource group **sql-server-hosting-RG**
    - Location: **local**

1. On the **Choose a size** blade, select **DS1_v2** and click **Select**.
1. On the **Settings** pane of the **Create virtual machine** blade, set **Network Security Group** settings to **Advanced** and then click **Network security group (firewall)**.
1. On the **Create network security group** blade, click **+ Add an inbound rule**.
1. On the **Add inbound security rule** blade, specify the following settings and click **Add** (leave others with their default values):

    - Destination port ranges: **1433**
    - Protocol: **TCP**
    - Action: **Allow**
    - Name: **SQL**

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

1. Once the deployment completes, navigate to the **sql-host-vm0** virtual machine blade and, in the **Overview** section, directly under the **DNS name** labe, click **Configure**.
1. On the **sql-host-vm0-ip \| Configuration** blade, in the **DNS name label (optional)** text box, type **sql-host-vm0** and click **Save**.

    >**Note**: This makes the **sql-host-vm0** available via **sql-host-vm0.local.cloudapp.azurestack.external** DNS name.

1. On the **sql-host-vm0-ip \| Configuration** blade, set the **Assignment** option to **Static** and click **Save**.

    >**Note**: This will trigger a restart of the **sql-host-vm0** virtual machine.


#### Task 3: Add a SQL hosting server (as a cloud operator)

In this task, you will:

- Add a SQL hosting server (as a cloud operator)

1. Within the Remote Desktop session to **AzSHOST-1**, in the the web browser displaying the Azure Stack administrator portal, click **All services** and, in the **Administrative resources** section, click **SQL Hosting Servers**.

    > **Note:** You might need to refresh the browser page displaying the Azure Stack administrator portal for the **SQL Hosting Servers** resource type to appear.

1. On the **SQL Hosting Servers** blade, click **+ Add**.
1. On the **Add a SQL Hosting Server** blade, specify the following settings:

    - SQL Server Name: **sql-host-vm0.local.cloudapp.azurestack.external**
    - Username: **sqladmin**
    - Password: **Pa55w.rd**
    - Size of Hosting Server in GB: **50**
    - Always On Availability Group: unchecked
    - Subscription: **Default Provider Subscription**
    - Resource group: the name of a new resource group **sql.resources-RG**
    - Location: **local**

1. On the **Add a SQL Hosting Server** blade, click **SKU**, on the **SKUs** blade, click **Create new SKU** and, on the **Create SKU** blade, specify the following settings:

    - Name: **MSSQL2017Exp**
    - Family: **SQL Server 2017**
    - Tier: **Standalone**
    - Edition: **Express**

1. On the **Create SKU** blade, click **OK** and back on the **Add a SQL Hosting Server** blade, click **Create**.

    > **Note:** Wait for the operation to complete. This should take less than 1 minute.

1. On the **SQL Hosting Servers** blade, click **Refresh** and verify that the **sqlhost1.local.cloudapp.azurestack.external** appears on the list of servers.


#### Task 4: Make SQL databases available to users (as a cloud operator)

In this task, you will:

- Make SQL databases available to users (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**.
1. On the **New** blade, click **Offers + Plans** and then click **Plan**.
1. On the **Basics** tab of the **New plan** blade, specify the following settings:

    - Display name: **sql-server-2017-express-db-plan1**
    - Resource name: **sql-server-2017-express-db-plan1**
    - Resource group: the name of a new resource group **sqldb-plans-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.SQLAdapter** checkbox.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, next to the **Microsoft.SQLAdapter** entry, click **Create New**.
1. On the **Create Quota* blade, specify the following settings and click **Create**:

    - Quota Name: **sql-server-2017-express-db-quota1**
    - Maximum size of all Databases (GB): **2**
    - Maximum number of databases: **20**

1. Click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, back on the **New** blade, click **Offer**.
1. On the **Basics** tab of the **Create a new offer** blade, specify the following settings:

    - Display name: **sql-server-2017-express-db-offer1**
    - Resource name: **sql-server-2017-express-db-offer1**
    - Resource group: **sqldb-offers-RG**
    - Make this offer public: **Yes**

1. Click **Next: Base plans >**. 
1. On the **Base plans** tab of the **Create a new offer** blade, select the checkbox next to the **sql-server-2017-express-db-plan1** entry.
1. Click **Next: Add-on plans >**.
1. Leave **Add-on plans** settings with their default values, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.


#### Task 5: Crate a SQL database (as a user)

In this task, you will:

- Create a test user account
- Crate a SQL database by using the newly created user account

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
1. On the **Get a subscription** blade, in the **Name** text box, type **t1u1-sqldb-subscription1**.
1. In the list of offers, select **sql-server-2017-express-db-offer1** and click **Create**.
1. When presented with the message **Your subscription has been created. You must refresh the portal to start using your subscription**, click **Refresh**. 
1. In the Azure Stack Hub tenant portal, in the hub menu, click **All services**.
1. In the list of services, click **SQL Databases**.
1. On the **SQL Databases** blade, click **+ Add**.
1. On the **Create Database** blade, specify the following settings:

    - Database Name: **sqldb1**
    - Collation: **SQL_Latin1_General_CP1_CI_AS**
    - Max Size in MB: **200**
    - Subscription: ****t1u1-sqldb-subscription1**
    - Resource Group: the name of a new resource group **sqldb-RG**
    - Location: **local**
    - SKU: **MSSQL2017Exp**

    >**Note**: You might have to wait before a newly created SKU becomes available in the tenant portal.

1. On the **Create Database** blade, click **Login**.
1. On the **Select a Login** blade, click **Create a new login**.
1. On the **New Login** blade, specify the following settings and click **OK**:

    - Database login: **dbAdmin**
    - Password: **Pa55w.rd**

1. Back on the **Create Database** blade, click **Create**.

    >**Note**: Wait for the deployment to complete. This should take less than a minute. 

>**Review**: In this exercise, you have added a SQL Server hosting server to Azure Stack Hub, made it available to tenants, and deployed a SQL database as a tenant user.