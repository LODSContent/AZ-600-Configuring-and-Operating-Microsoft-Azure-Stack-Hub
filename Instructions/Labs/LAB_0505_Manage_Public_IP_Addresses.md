---
lab:
    title: 'Lab: Manage Public IP Addresses in Azure Stack Hub'
    module: 'Module 5: Manage Infrastructure'
---

# Lab - Manage Public IP Addresses in Azure Stack Hub
# Student lab manual

## Lab dependencies

- None

## Estimated Time

30 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to manage public IP address resources. 

## Objectives

After completing this lab, you will be able to:

- Manage public IP address resources

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

You will install software necessary to manage Azure Stack Hub via PowerShell in the course of this lab. You will also create additional user accounts.

## Instructions

### Exercise 0: Prepare for the lab

In this exercise, you will create an Active Directory user account that you will be using in this lab:

1. Create a user account (as a cloud operator)

#### Task 1: Create a user account (as a cloud operator)

In this task, you will:

- Create a user account (as a cloud operator)

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

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

>**Review**: In this exercise, you have created the Active Directory account you will use in this lab.


### Exercise 1: Create an offer (as a cloud operator)

In this exercise, you will act as a cloud operator. First, you will review the public IP address usage, then you will create a plan consisting of the network services and an offer containing this plan. Next, you will make the offer public, allowing users to create subscriptions based on this offer. The exercise consists of the following tasks:

1. Review public IP address usage (as a cloud operator)
1. Create a plan consisting of the network services (as a cloud operator).
1. Create an offer based on the plan and make the offer public (as a cloud operator)


#### Task 1: Review public IP address usage (as a cloud operator)

In this task, you will:

- Review public IP address usage (as a cloud operator).

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the Azure Stack Hub administrator portal, on the **Dashboard** page, in the **Resource providers** tile, click **Network**. 
1. On the **Network** blade, note the **Public IP pools usage** graph and the numbers of used and free IP addresses.

#### Task 2: Create a plan consisting of the network services (as a cloud operator)

In this task, you will:

- Create a plan consisting of the network services (as a cloud operator).

1. In the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Plan**.
1. On the **Basics** tab of the **New plan** blade, specify the following settings:

    - Display name: **Network-plan1**
    - Resource name: **network-plan1**
    - Resource group: the name of a new resource group **network-plans-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.Network** checkbox.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, select **Create New**.
1. On the **Create Network quota** blade, specify the following settings and click **OK**:

    - Name: **Network-plan1-quota**
    - Max virtual networks: **2**
    - Max virtual network gateways: **2**
    - Max network connections: **2**
    - Max public IPs: **20**
    - Max NICs 20**
    - Max load balancers: **5**
    - Max network security groups: **20**

1. Click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.


#### Task 3: Create an offer based on the plan (as a cloud operator)

In this task, you will:

- Create an offer based on the plan (as a cloud operator)

1. In the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Offer**.
1. On the **Basics** tab of the **Create a new offer** blade, specify the following settings:

    - Display name: **Network-offer1**
    - Resource name: **network-offer1**
    - Resource group: a new resource group named **network-offers-RG**
    - Make this offer public: **Yes**

1. Click **Next: Base plans >**. 
1. On the **Base plans** tab of the **Create a new offer** blade, select the checkbox next to the **Network-plan1** entry.
1. Click **Next: Add-on plans >**.
1. Leave **Add-on plans** settings with their default values, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

>**Review**: In this exercise, you have created a plan and a public offer based on that plan.


### Exercise 2: Create public IP address resources (as a user)

In this exercise, you will act as users who signs up for the offer you created in the first exercise, creates a new subscription, and creates public IP address resources in that subscription. The exercise consists of the following tasks:

1. Sign up for the offer (as a user)
1. Connect to Azure Stack Hub user Azure Resource Manager endpoint (as a user)
1. Create IP address resources (as a user)

#### Task 1: Sign up for the offer (as a user)

In this task, you will:

- Sign up for the offer (as a user)

1. 1. Within the Remote Desktop session to **AzS-HOST1**, start an InPrivate session of the web browser.
1. In the web browser window, navigate to the [Azure Stack Hub user portal](https://portal.local.azurestack.external) and sign in as **t1u1@azurestack.local** with the password **Pa55w.rd**.
1. In the Azure Stack Hub user portal, on the dashboard, click **Get a subscription**.
1. On the **Get a subscription** blade, in the **Display name** text box, type **T1U1-network-subscription1**.
1. In the list of offers, select **Network-offer1** and then click **Create**.
1. In the message box **Your subscription has been created. You must refresh the portal to start using your subscription**, click **Refresh**.


#### Task 2: Connect to Azure Stack Hub Azure Resource Manager user endpoint (as a user)

In this task, you will:

- Connect to Azure Stack Hub Azure Resource Manager user endpoint (as a user)

1. Within the Remote Desktop session to **AzS-HOST1**, start PowerShell 7 as Administrator.

    >**Note**: For detailed instructions on setting up PowerShell connectivity to Azure Stack Hub, follow instructions in the lab **Connect to Azure Stack Hub via PowerShell**.

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to install the Azure Stack Hub PowerShell modules required for this lab:

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force -AllowPrerelease -AllowClobber
    Install-AzProfile -Profile 2019-03-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.0.2-preview -AllowPrerelease
    ```

    >**Note**: Disregard any error messages regarding already available commands.

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to download and extract the Azure Stack Hub tools:

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to register your Azure Stack Hub user environment:

    ```powershell
    Add-AzEnvironment -Name 'AzureStackUser' -ArmEndpoint 'https://management.local.azurestack.external'
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to initiate authentication to the Azure Stack Hub user environment via a browser session as the **t1u1@azurestack.local** user:

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackUser' -UseDeviceAuthentication
    ```

1. In the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, review the resulting message, open another web browser window in the InPrivate mode, navigate to the [adfs.local.azurestack.external](https://adfs.local.azurestack.external/adfs/oauth2/deviceauth) page, and type in the code included in the reviewed message. If prompted, sign in again as the **t1u1@azurestack.local** user.
1. Switch back to the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window and verify that you have successfully authenticated as the **t1u1@azurestack.local** user.


#### Task 3: Create IP address resources (as a user)

In this task, you will:

- Create IP address resources (as a user)

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to verify that you are using the newly provisioned subscription: 

    ```powershell
    (Get-AzSubscription).Name
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to register the Network Resource Provider within the current subscription:

    ```powershell 
    Register-AzResourceProvider -ProviderNamespace Microsoft.Network
    ```

    >**Note**: You have to register a resource provider in order to create resources managed by that provider.

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to create a resource group that will host public IP address resources:

    ```powershell 
    $rg = New-AzResourceGroup -Name publicIPs-RG -Location local
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** prompt, run the following to create public IP address resources: 

    ```powershell
    1..5 | ForEach-Object {New-AzPublicIpAddress -Name "publicIP$_" -ResourceGroupName $rg.ResourceGroupName -AllocationMethod Static -Location local}
    ```

1. Wait until all IP address resources are provisioned.

>**Review**: After completing this exercise, you have created public IP address resources in a user's subscription.


### Exercise 4: Manage public IP address usage (as a cloud operator)

In this exercise, you will act as a cloud operator, reviewing and managing the public IP address usage. The exercise consists of the following tasks:

1. Review public IP address usage
2. Add a public IP address pool

#### Task 1: Review public IP address usage (as a cloud operator)

In this task, you will:

- Review public IP address usage (as a cloud operator).

1. Switch to the web browser window displaying the Azure Stack Hub administrator portal, where you are signed in as CloudAdmin@azurestack.local.
1. In the Azure Stack Hub administrator portal, in the hub menu, click **Dashboard** and, on the **Resource providers** tile, click **Network**.
1. On the **Network** blade, review again the **Public IP pools usage** graph and the numbers of used and free IP addresses.

    >**Note**: The numbers should have changed, reflecting additional 5 public IP addresses you created in the user subscription (as the user).

#### Task 2: Add a public IP address pool (as a cloud operator)

In this task, you will:

- Add a public IP address pool

1. In the web browser window displaying the Azure Stack Hub administrator portal, on the **Network** blade, click the **Public IP pools usage** tile.

    >**Note**: If the **Public IP pools** blade displays the message **The exclusive operation 'Startup' is in progress. Add node and add IP pool operations are disabled while the operation is running. Click here to view the activity log**, then you will need to wait until the 'Startup' operation completes before you proceed to the next step.

1. On the **Public IP pools** blade, click **+ Add IP pool**. 
1. On the **Add IP pool** blade, specify the following settings and click **Add**.

    - Name: **Public Pool 1**
    - Region: **local**
    - Address range (CIDR block): **192.168.110.0/24**

1. Wait for the change to take effect and navigate back to the **Network** blade.
1. Review the **Public IP pools usage** graph and note the changes in the number of free IP addresses.

>**Review**: In this exercise, you have reviewed and configured public IP address pools.