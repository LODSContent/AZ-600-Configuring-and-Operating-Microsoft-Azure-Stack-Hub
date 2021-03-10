---
lab:
    title: 'Lab: Connect to Azure Stack Hub via PowerShell'
    module: 'Module 4: Manage Infrastructure'
---

# Lab - Connect to Azure Stack Hub via PowerShell
# Student lab manual

## Lab dependencies

- None

## Estimated Time

30 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to be able to manage your environment by using PowerShell. You also need to be able to ocasionally connect to the Azure Stack Hub via PowerShell as a user. 

## Objectives

In this lab, you will be able to:

- Connect to the ASDK operator and user environments via PowerShell

## Lab Environment

This lab uses an ADSK instance integrated with Active Directory Federation Services (AD FS) (backed up Active Directory as the identity provider). 

>**Note**: For information regarding connecting to Azure Stack Hub integrated with Azure Active Directory (Azure AD), refer to [Connect to Azure Stack Hub with PowerShell](https://docs.microsoft.com/en-us/azure-stack/operator/azure-stack-powershell-configure-admin?view=azs-2008&tabs=az1%2Caz2%2Caz3#connect-with-azure-ad).

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

## Instructions

### Exercise 1: Connecting to ASDK via Azure PowerShell

In this exercise, you will connect to the Admin ARM endpoint of ASDK from the ASDK host via PowerShell:

1. Install Azure Stack Hub compatible Az PowerShell modules
1. Download Azure Stack Hub tools.
1. Configure and connect to the Azure Stack Hub operator environment via PowerShell
1. Configure and connect to the Azure Stack Hub user environment via PowerShell

#### Task 1: Install Azure Stack Hub compatible Azure PowerShell modules

In this task, you will:

- Remove any pre-exisiting Azure and Az PowerShell modules.
- Install and configure prerequisites for Azure Stack Hub compatible Az PowerShell modules.
- Install Azure Stack Hub compatible Az PowerShell modules.

>**Note**: All versions of the Azure Resource Manager (AzureRM) PowerShell module are outdated, although not out of support. However, the Az PowerShell module is now the recommended PowerShell module for interacting with Azure and Azure Stack Hub. 

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzS-HOST1**, start **Windows PowerShell** as Administrator.
1. From the **Administrator: Windows PowerShell** prompt, run the following to remove all existing versions of Azure PowerShell and Az PowerShell modules:

    ```powershell
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    Get-Module -Name Az.* -ListAvailable | Uninstall-Module -Force -Verbose -ErrorAction Continue
    ```

    >**Note**: If you receive an error message regarding in-use modules, close the Windows PowerShell session, re-open it, and rerun the above commands.

1. Within the Remote Desktop session to **AzS-HOST1**, start File Explorer and delete all the folders which names start with **Azure**, **Az** or **Azs** from the **C:\\Program Files\\WindowsPowerShell\\Modules** and **C:\\Users\\AzureStackAdmin\\Documents\\PowerShell\\Modules\\** folders.
1. Within the Remote Desktop session to **AzS-HOST1**, start Microsoft Edge, navigate to the [PowerShell releases page](https://github.com/PowerShell/PowerShell/releases/tag/v7.1.2). 
1. From the [PowerShell releases page](https://github.com/PowerShell/PowerShell/releases/tag/v7.1.2) page, download and install the latest release of PowerShell. 
1. Within the Remote Desktop session to **AzS-HOST1**, start PowerShell 7 as administrator.
1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to configure PowerShell Gallery as a trusted repository

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to install PowerShellGet:

    ```powershell
    Install-Module PowerShellGet -MinimumVersion 2.2.3 -Force
    ```

    >**Note**: Disregard any warning messages regarding in-use modules.

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to install the PowerShell Az module for Azure Stack Hub:

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**Note**: Disregard any error messages regarding already available commands.


#### Task 2: Download Azure Stack Hub tools 

In this task, you will:

- Download Azure Stack Hub tools from GitHub

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to download the Azure Stack Tools:

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

    >**Note**: This step copies the archive containing the GitHub repository hosting the Azure Stack Hub tools to the local computer and expands the archive to the **C:\\AzureStack-Tools-master** folder. The tools contain PowerShell modules that offer a range of features, including identifying Azure Stack Hub capabilities, managing Azure Stack Hub VM infrastructure and images, configuring Resource Manager policies, registering Azure Stack Hub with Azure, Azure Stack Hub deployment, connectivity to Azure Stack Hub, Azure Stack Hub tenant management, and validation of Azure Stack Hub Resource Manager templates. 


#### Task 3: Configure and connect to the Azure Stack Hub operator environment via PowerShell

In this task, you will:

- Configure the Azure Stack Hub operator environment via PowerShell
- Connect to the Azure Stack Hub operator environment via PowerShell
- Verify connection to the Azure Stack Hub operator environment via PowerShell

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to register your Azure Stack Hub operator PowerShell environment:

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

    >**Note**: Verify that the command returns the following output:

    ```powershell
    Name            Resource Manager Url                              ActiveDirectory Authority
    ----            --------------------                              -------------------------
    AzureStackAdmin https://adminmanagement.local.azurestack.external https://adfs.local.azurestack.external/adfs/
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to sign in to your Azure Stack Hub operator PowerShell environment with the AzureStack\CloudAdmin credentials:

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin' -UseDeviceAuthentication
    ```

1. In the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, review the resulting message, open a web browser window, navigate to the [adfs.local.azurestack.external](https://adfs.local.azurestack.external/adfs/oauth2/deviceauth) page, type the code included in the reviewed message, and click **Continue**. 
1. When prompted, sign in by using the following credentials:

    - Username: **CloudAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Switch back to the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window and verify that you have successfully authenticated as **CloudAdmin@azurestack.local**.
1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to list the Azure Stack Hub admin subscriptions

    ```powershell
    Get-AzSubscription
    ```

    >**Note**: Verify that the output includes **Default Provider Subscription**, **Metering Subscription**, and **Consumption Subscription**.

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to verify the corresponding PowerShell environment context:

    ```powershell
    Get-AzContext
    ```


#### Task 4: Configure and connect to the Azure Stack Hub user environment via PowerShell

In this task, you will:

- Configure the Azure Stack Hub user environment via PowerShell
- Connect to the Azure Stack Hub user environment via PowerShell
- Verify connection to the Azure Stack Hub user environment via PowerShell

1. Within the Remote Desktop session to **AzS-HOST1**, start PowerShell 7.
1. From the **C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to register an Azure Resource Manager environment that targets your Azure Stack Hub user environment:

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint 'https://management.local.azurestack.external'
    ```

    >**Note**: Verify that the command returns the following output:

    ```powershell
    Name            Resource Manager Url                              ActiveDirectory Authority
    ----            --------------------                              -------------------------
    AzureStackUser https://management.local.azurestack.external https://adfs.local.azurestack.external/adfs/
    ```

1. From the **C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to sign in to your Azure Stack Hub PowerShell environment.

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackUser'
    ```

    >**Note**: This will automatically open a web browser window, prompting you for the AzureStack\CloudAdmin credentials.

1. When prompted, sign in by using the following credentials:

    - Username: **CloudAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to list the Azure Stack Hub admin subscriptions

    ```powershell
    Get-AzSubscription
    ```

    >**Note**: Verify that the output does **not** include **Default Provider Subscription**, **Metering Subscription**, and **Consumption Subscription**.

>**Review**: In this exercise, you have connected to the Azure Stack Hub operator and user environments via PowerShell.
