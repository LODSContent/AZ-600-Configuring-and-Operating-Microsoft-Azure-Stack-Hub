---
lab:
    title: 'Lab: Manage Service Principals in Azure Stack Hub'
    module: 'Module 4: Manage Identity and Access'
---

# Lab - Manage Service Principals in Azure Stack Hub
# Student lab manual

## Lab dependencies

- None

## Estimated Time

30 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You plan to use an internally develop application to manage Azure Stack Hub. To allow for the application to authenticate, you need to create a service principal and assign to it the Contributor role in the Default Provider Subscription.

>**Note**: An application that needs to deploy or configure resources through Azure Resource Manager must be represented by its own identity. Just as a user is represented by a security principal called a user principal, an app is represented by a service principal. The service principal provides an identity for your app, allowing you to delegate only the necessary permissions to the app.

An app must present credentials during authentication. This authentication consists of two elements:

- An Application ID, sometimes referred to as a Client ID. A GUID that uniquely identifies the app's registration in your Active Directory tenant.
- A secret associated with the application ID. You can either generate a client secret string (equivalent to a password), or specify an X509 certificate.

In this lab, you will use a secret. For details regarding authentication by using certificates, refer to [Use an app identity to access Azure Stack Hub resources](https://docs.microsoft.com/en-us/azure-stack/operator/azure-stack-create-service-principals?view=azs-2008&tabs=az1%2Caz2&pivots=state-disconnected).

## Objectives

After completing this lab, you will be able to:

- Create and manage service principals in Azure Stack Hub AD FS integrated scenarios.

## Lab Environment 

This lab uses an ADSK instance integrated with Active Directory Federation Services (AD FS) (backed up Active Directory as the identity provider). 

The lab environment consists of the following components:

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

The approach described in this lab is specific to the AD FS integrated scenarios. For information regarding implementing service principal-based auhentication when using Azure Active Directory (Azure AD) as the identity provider, refer to [Use an app identity to access Azure Stack Hub resources](https://docs.microsoft.com/en-us/azure-stack/operator/azure-stack-create-service-principals?view=azs-2008&tabs=az1%2Caz2&pivots=state-connected).


### Exercise 1: Create and configure a service principal in Azure Stack Hub

In this exercise, you will establish a PowerShell Remoting session to the privileged endpoint to create a service principal and use the Azure Stack Hub administrator portal to assign to it the Contributor role. The exercise consists of the following tasks:

1. Create a service principal
1. Assign the Contributor role to the service principal


#### Task 1: Create a service principal

In this task, you will:

- Connect to the privileged endpoint via PowerShell and create a service principal.

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzS-HOST1**, start Windows PowerShell as administrator.
1. From the **Administrator: Windows PowerShell** window, run the following to establish a PowerShell Remoting session to the privileged endpoint:

    ```powershell
    $session = New-PSSession -ComputerName AzS-ERCS01 -ConfigurationName PrivilegedEndpoint
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to create the new app registration (and service principal object) and store the reference to it in the **$spObject** variable:

    ```powershell
    $spObject = Invoke-Command -Session $session -ScriptBlock {New-GraphApplication -Name 'azsmgmt-app1' -GenerateClientSecret}
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to retrieve the Azure Stack Hub stamp information and store the reference to it in the **$azureStackInfo** variable:

    ```powershell
    $azureStackInfo = Invoke-Command -Session $session -ScriptBlock {Get-AzureStackStampInformation}
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to terminate the PowerShell Remoting session ot the privileged endpoint:

    ```powershell
    $session | Remove-PSSession
    ```

    >**Note**: In general, you should use the **Close-PrivilegedEndpoint** cmdlet to close the privileged endpoint session. We are not following this practice for the sake of simplicity to avoid the need to set up a file share to host the session transcript logs.

1. From the **Administrator: Windows PowerShell** window, run the following to use the Azure Stack Hub stamp information you retrieved in earlier in this task to set values of the variables you will use to configure the service principal, referencing, respectively, the Azure Stack Hub endpoint used for Azure Resource Manager user operations, audience for acquiring an OAuth token used to access Graph API, and GUID of the identity provider:

    ```powershell
    $armUseEndpoint = $azureStackInfo.TenantExternalEndpoints.TenantResourceManager
    $graphAudience = "https://graph." + $azureStackInfo.ExternalDomainFQDN + "/"
    $tenantID = $azureStackInfo.AADTenantID
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to register and set the Azure Stack Hub user environment:

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint $armUseEndpoint
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to sign in as the service principal into the AzureStackUser envrionment:

    ```powershell
    $securePassword = $spObject.ClientSecret | ConvertTo-SecureString -AsPlainText -Force
    $credential = New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $spObject.ClientId, $securePassword
    $spUserSignIn = Connect-AzAccount -Environment 'AzureStackUser' -ServicePrincipal -Credential $credential -TenantId $tenantID
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to verify that the sign in was successful:

    ```powershell
    $spUserSignIn
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to remove the current authentication context:

    ```powershell
    Remove-AzAccount -Username $credential.UserName
    ```

    >**Note**: Now you will repeat the equivalent sequence of steps to validate that you can authenticate to the Azure Stack Hub administrator environment:

1. From the **Administrator: Windows PowerShell** window, run the following to use the Azure Stack Hub stamp information you retrieved in earlier in this task to set values of the variables you will use to configure the service principal, referencing, respectively, the Azure Stack Hub endpoint used for Azure Resource Manager administrative operations, audience for acquiring an OAuth token used to access Graph API, and GUID of the identity provider:

    ```powershell
    $armAdminEndpoint = $azureStackInfo.AdminExternalEndpoints.AdminResourceManager
    $graphAudience = "https://graph." + $azureStackInfo.ExternalDomainFQDN + "/"
    $tenantID = $azureStackInfo.AADTenantID
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to register and set the Azure Stack Hub admin environment:

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint $armAdminEndpoint
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to sign in as the service principal into the AzureStackAdmin envrionment::

    ```powershell
    $spAdminSignIn = Connect-AzAccount -Environment 'AzureStackAdmin' -ServicePrincipal -Credential $credential -TenantId $tenantID
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to verify that the sign in was successful:

    ```powershell
    $spAdminSignIn
    ```

1. From the **Administrator: Windows PowerShell** window, run the following to display the properties of the new service principal:

    ```powershell
    $spObject
    ```

    >**Note**: The output should have the following format:

    ```
    ApplicationIdentifier : S-1-5-21-2657257302-3827180852-1812683747-1510
    ClientId              : 6a3fb4ad-838a-47b1-a93c-f3e4a1b683c8
    Thumbprint            :
    ApplicationName       : Azurestack-azsmgmt-app2-53508ec5-d7d9-4761-876a-3602542c2965
    ClientSecret          : 3fKPtUg37YraCk1IaFtdqeyTpVplXDqc25Dj3bUs
    PSComputerName        : AzS-ERCS01
    RunspaceId            : 6b142339-b67f-490e-a258-40983c0cd8ea
    ```

    >**Note**: Record the value of the **ApplicationName** property. You will need it in the next task. In addition, you should record the value of the **ClientSecret** property and provide it to developers who implement the management application.


#### Task 2: Assign the Contributor role to the service principal

In this task, you will:

- Assign the Contributor role to the service principal by using the Azure Stack Hub administrator portal.

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the web browser displaying the Azure Stack Hub administrator portal, in the hub menu, select **All services**. 
1. On the **All services** blade, select **General** and, in the list of services, select **Subscriptions**.
1. On the **Subscriptions** blade, select **Default Provider Subscription**.
1. On the **Default Provider Subscription** blade, select **Access control (IAM)**.
1. On the **Default Provider Subscription | Access control (IAM)** blade, click **+ Add** and, in the drop-down menu, select **Add role assignment**.
1. On the **Add role assignment** blade, specify the following settings and click **Save**:

    - Role: **Contributor**
    - Assign access to: **User, group, or service principal**
    - Select: search for and select the value of the **ApplicationName** property of the service principal you identified in the previous task.

1. Verify that the role assignment was successful.

>**Review**: In this exercise, you have created a service principal and assigned to it the Contributor role. 