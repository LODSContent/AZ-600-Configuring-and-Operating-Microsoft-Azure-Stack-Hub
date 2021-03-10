---
lab:
    title: 'Lab: Register Azure Stack Hub with an Azure Subscription'
    module: 'Module 2: Implement Data Center Integration'
---

# Lab - Register Azure Stack Hub with an Azure Subscription
# Student lab manual

## Lab dependencies

- None

## Estimated Time

30 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to register it with your Azure subscription in order to download Azure Marketplace items and set up data reporting to Azure. 

## Objectives

After completing this lab, you will be able to:

- Register Azure Stack Hub with an Azure subscription 

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


### Exercise 1: Register Azure Stack Hub with an Azure subscription.

In this exercise, you will register Azure Stack Hub with an Azure subscription.

1. Register the Azure Stack Hub resource provider
1. Perform Azure Stack Hub registration
1. Verify Azure Stack Hub registration

#### Task 1: Register the Azure Stack Hub resource provider

In this task, you will:

- Register the Azure Stack Hub resource provider in the target Azure subscription.

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzS-HOST1**, start PowerShell 7 as administrator.
1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to install the PowerShell Az module for Azure Stack Hub:

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to authenticate to the Azure subscription you will be using in this lab.

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureCloud'
    ```

1. When prompted, sign in with the credentials of an Azure Active Directory (Azure AD) user with the Contributor role in the Azure subscription you will be using in this lab.
1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to register the Azure Stack resource provider:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.AzureStack
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to determine whether that the registration is completed:

    ```powershell
    Get-AzResourceProvider -ProviderNamespace Microsoft.AzureStack | Where-Object {$_.RegistrationState -eq 'Registered'}
    ```

    >**Note**: Wait for the registration to complete. Re-run the **Get-AzResourceProvider** cmdlet in order to verify the status of the registration.


#### Task 2: Perform Azure Stack Hub registration

In this task, you will:

- Perform Azure Stack Hub registration.

1. Within the Remote Desktop session to **AzSHOST-1**, from the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to establish a privileged endpoint (PEP) session:

    ```powershell
    Enter-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, within the PowerShell Remoting session to the privileged endpoint, run the following to display the the summary configuration of the Azure Stack Hub stamp:

    ```powershell
    Get-AzureStackStampInformation
    ```

1. In the output of the command you ran in the previous step, identify and record the value of the **CloudId** property.
1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, within the PowerShell Remoting session to the privileged endpoint, run the following to exit the session:

    ```powershell
    Exit-PSSession
    ```

    >**Note**: In general, you should not use the **Exit_PSSession** to terminate the session to the privileged endpoint, but use the **Close-PrivilegedEndpoint** cmdlet instead. We are not following this practice for the sake of simplicity to avoid the need to set up a file share to host the session transcript logs.

    >**Note**: You can also identify the value of the stamp **Cloud ID** property from the **local \| Properties** blade in the Azure Stack Hub adminstrator portal.

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to register the Azure Stack PowerShell environment that targets your Azure Stack instance (make sure to replace the `[cloud_ID]` placeholder with the value of the **CloudId** property you identified in the output of the **Get-AzureStackStampInformation** cmdlet):

    ```powershell
    Import-Module .\Registration\RegisterWithAzure.psm1
    $RegistrationName = "[cloud_ID]"
    Set-AzsRegistration `
       -PrivilegedEndpointCredential $adminCredentials `
       -PrivilegedEndpoint 'AzS-ERCS01' `
       -BillingModel 'Development' `
       -RegistrationName $RegistrationName `
       -UsageReportingEnabled:$true
    ```

1. When prompted, sign in with an Azure AD user account with the Contributor role in the Azure subscription.

    > **Note:** Wait for the registration to complete. This registration might take about 20 minutes.


#### Task 3: Verify Azure Stack Hub registration

In this task, you will:

- Verify registration of Azure Stack Hub with the Azure subscription

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the web browser displaying the Azure Stack Hub administrator portal, from the **Dashboard** blade, select the **Region management** tile.
1. On the **local** blade, select **Properties**. 
1. On the **local \| Properties** blade, verify that the **Registration status** is listed as **Registered**. 

    > **Note:** The status can be **Registered**, **Not registered**, or **Expired**.

>**Review**: In this exercise, you have registered Azure Stack Hub with an Azure subscription.