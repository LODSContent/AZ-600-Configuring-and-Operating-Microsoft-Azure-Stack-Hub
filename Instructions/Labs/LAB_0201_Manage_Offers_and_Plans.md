---
lab:
    title: 'Lab: Manage offers and plans in Azure Stack Hub'
    module: 'Module 2: Provide Services'
---

# Lab - Manage offers and plans in Azure Stack Hub
# Student lab manual

## Lab dependencies

- None

## Estimated Time

60 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to implement offers and plans for your users. You need to ensure that all users have the same set of base services but want to provide access to additional services only to some users.

## Objectives

After completing this lab, you will be able to:

- Implement Azure Stack Hub offers and plans by using the Azure Stack Hub administrator portal

## Lab Environment 

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

You will create additional user accounts in the course of this lab.


### Exercise 0: Prepare for the lab

In this exercise, you will create Active Directory user accounts that you will be using in this lab:

1. Create user accounts (as a cloud operator)


#### Task 1: Create the first user account (as a cloud operator)

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

1. Repeat the steps 5-6 to create another user account with the following settings:

    - Full name: **T1U2**
    - User UPN logon: **t1u2@azurestack.local**
    - User SamAccountName: **azurestack\t1u2**
    - Password: **Pa55w.rd**
    - Password options: **Other password options -> Password never expires**

>**Review**: In this exercise, you have created the Active Directory user accounts you will use in this lab.

### Exercise 1: Create offers (as a cloud operator)

In this exercise, you will act as a cloud operator and create a plan consisting of the compute, storage, and network services as well as an offer containing this plan. Next, you will make the offer public, allowing users to create subscriptions based on this offer. 
The exercise consists of the following tasks:

1. Create plans consisting of the compute, storage, and network services (as a cloud operator).
1. Create a public offer based on the plan (as a cloud operator)
1. Create a private offer based on the plan (as a cloud operator)

#### Task 1: Create plans consisting of the compute, storage, and network services (as a cloud operator)

In this task, you will:

- Create two plans consisting of the compute, storage, and network services (as a cloud operator).

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**.
1. On the **New** blade, click **Offers + Plans** and then click **Plan**.
1. On the **Basics** tab of the **New plan** blade, specify the following settings:

    - Display name: **base-plan1**
    - Resource name: **base-plan1**
    - Resource group: the name of a new resource group **base-plans-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.Compute**, **Microsoft.Storage**, and **Microsoft.Network** checkboxes.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, specify the following settings

    - Microsoft.Compute: **Default Quota**
    - Microsoft.Storage: **Default Quota**
    - Microsoft.Network: **Default Quota**

1. Click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

1. In the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**.
1. On the **New** blade, click **Offers + Plans** and then click **Plan**.
1. On the **Basics** tab of the **New plan** blade, specify the following settings:

    - Display name: **base-plan2**
    - Resource name: **base-plan2**
    - Resource group: the name of a new resource group **base-plans-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.Compute**, **Microsoft.Storage**, and **Microsoft.Network** checkboxes.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, next to the **Microsoft.Compute** drop-down list, click **Create New**.
1. On the **Create Compute quota** blade, specify the following settings and click **OK**":

    - Name: **base-plan1-compute-quota**
    - Number of virtual machines: **40**
    - Number of virtual machine cores: **100**
    - Number of availability sets: **20**
    - Number of virtual machine scale sets: **40**
    - Capacity (GB) of standard managed disks: **4096**
    - Capacity (GB) of premium managed disks: **4096**

1. Back on the **Quotas** tab of the **New plan** blade, next to the **Microsoft.Storage** drop-down list, click **Create New**.
1. On the **Create Storage quota** blade, specify the following settings and click **OK**":

    - Name: **base-plan1-storage-quota**
    - Maximum capacity (GB): **4096**
    - Total number of storage accounts: **40**

1. Back on the **Quotas** tab of the **New plan** blade, next to the **Microsoft.Network** drop-down list, click **Create New**.
1. On the **Create Network quota** blade, specify the following settings and click **OK**":

    - Name: **base-plan1-network-quota**
    - Max virtual networks: **100**
    - Max virtual network gateways: **2**
    - Network connections: **4**
    - Max public IPs: **100**
    - Max NICs: **200**
    - Max load balancers: **100**
    - Max network security groups: **100**

1. Click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.


#### Task 2: Create a public offer based on the plan (as a cloud operator)

In this task, you will:

- Create a public offer based on the plan (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans** and then click **Offer**.
1. On the **Basics** tab of the **Create a new offer** blade, specify the following settings:

    - Display name: **base-offer1**
    - Resource name: **base-offer1**
    - Resource group: **base-offers-RG**
    - Make this offer public: **Yes**

1. Click **Next: Base plans >**. 
1. On the **Base plans** tab of the **Create a new offer** blade, select the checkbox next to the **base-plan1** entry.
1. Click **Next: Add-on plans >**.
1. Leave **Add-on plans** settings with their default values, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.


#### Task 3: Create a private offer based on the plan (as a cloud operator)

In this task, you will:

- Create an private offer based on the plan (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Offer**.
1. On the **Basics** tab of the **Create a new offer** blade, specify the following settings:

    - Display name: **base-offer2**
    - Resource name: **base-offer2**
    - Resource group: **base-offers-RG**
    - Make this offer public: **No**

1. Click **Next: Base plans >**. 
1. On the **Base plans** tab of the **Create a new offer** blade, select the checkbox next to the **base-plan2** entry.
1. Click **Next: Add-on plans >**.
1. Leave **Add-on plans** settings with their default values, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

>**Review**: In this exercise, you have created a plan and two offer based on that plan, with one of them being public and the other private.


### Exercise 2: Create subscriptions based on the offers (as users)

In this exercise, you will create subscriptions based on two offers you created in the previous exercise. The exercise consists of the following tasks:

1. Sign up for the offer (as the first user)
1. Assign a subscription to the second user (as an operator)

#### Task 1: Sign up for the offer (as the first user)

In this task, you will:

- Sign up for the offer (as the first user)

1. Within the Remote Desktop session to **AzS-HOST1**, start an InPrivate session of the web browser.
1. In the web browser window, navigate to the [Azure Stack Hub user portal](https://portal.local.azurestack.external) and sign in as **t1u1@azurestack.local** with the password **Pa55w.rd**.
1. In the Azure Stack Hub user portal, on the Dashboard, click the **Get a subscription** tile.
1. On the **Get a subscription** blade, in the **Name** text box, type **t1u1-base-subscription1**.
1. In the list of offers, select **base-offer1** and click **Create**.
1. When presented with the message **Your subscription has been created. You must refresh the portal to start using your subscription**, click **Refresh**. 
1. In the Azure Stack Hub tenant portal, in the hub menu, click **All services**.
1. In the list of services, click **Subscriptions**.
1. On the **Subscriptions** blade, click **t1u1-base-subscription1**.
1. On the **t1u1-base-subscrption1** blade, click **Resources**.
1. On the **t1u1-base-subscrption1 - Resources** blade, click **+ Add**.
1. On the **New** blade, click **Security + Identity** and then click **Key Vault**.
1. On the **Basics** tab of the **Create key vault** blade, specify the following settings (leave others with their default values):

    - Resource group: the name of a new resource group **kv-RG**
    - Key vault name: any unique name consisting of between 3 and 24 alphanumeric characters and dashes, starting with a letter

1. Click **Review + create** and then click **Create**.
1. Verify that the deployment failed. 

    >**Note**: Review the error message **The resource namespace 'Microsoft.KeyVault' is invalid**. You cannot create the key vault since it was not included in the offer.

1. Sign out from the Azure Stack Hub user portal and close the InPrivate session of the web browser.

#### Task 2: Assign a subscription to the second user (as an operator)

In this task, you will:

- Assign a subscription to the second user (as an operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) where you are signed in as CloudAdmin@azurestack.local, select **All resources**.
1. On the **All resources** blade, search for and select **base-offer2**.
1. On the **base-offer2** blade, select **Subscriptions**.
1. On the **base-offer2 \| Subscriptions** blade, click **+ Add**.
1. On the **Create a user subscription** blade, specify the following settings and click **Create**.

    - Name: **t1u2-base-subscription1**
    - User: **t1u2@azurestack.local**
    - Directory tenant: **ADFS.azurestack.local**
    - Offer name: **base-offer2**

1. Within the Remote Desktop session to **AzS-HOST1**, start an InPrivate session of the web browser.
1. In the web browser window, navigate to the [Azure Stack Hub user portal](https://portal.local.azurestack.external) and sign in as **t1u2@azurestack.local** with the password **Pa55w.rd**.
1. In the Azure Stack Hub user portal, in the hub menu, select **All services** and then, on the **All services** blade, **Subscriptions**.
1. On the **Subscriptions** blade, select **t1u2-base-subscription1**.
1. On the **t1u2-base-subscrption1** blade, click **Resources**.
1. On the **t1u2-base-subscrption1 - Resources** blade, click **+ Add**.
1. On the **New** blade, click **Security + Identity** and then click **Key Vault**.
1. On the **Basics** tab of the **Create key vault** blade, specify the following settings (leave others with their default values):

    - Resource group: the name of a new resource group **kv-RG**
    - Key vault name: any unique name consisting of between 3 and 24 alphanumeric characters and dashes, starting with a letter

1. Click **Review + create** and then click **Create**.
1. Verify that the deployment failed. 

    >**Note**: Review the error message **The resource namespace 'Microsoft.KeyVault' is invalid**. You cannot create the key vault since it was not included in the offer.

1. Sign out from the Azure Stack Hub user portal and close the InPrivate session of the web browser.

>**Review**: In this exercise, you have subscribed to the offer as two users, creating this way two new user subscriptions.


### Exercise 3: Create an add-on plan and assign it to a user subscription (as a cloud operator)

In this exercise, you will act as a cloud operator and create an add-on plan consisting of the Key Vault service. Next, you will assign it to one of the user subscriptions. 

1. Create an add-on plan including the Key Vault service (as a cloud operator).
1. Assign the add-on plan to an existing offer (as a cloud operator)

#### Task 1: Create an add-on plan including the Key Vault service (as a cloud operator)

In this task, you will:

- Create an add-on plan consisting of the Key Vault service (as a cloud operator).

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal where you are signed in as CloudAdmin@azurestack.local, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Plan**.
1. On the **New plan** blade, specify the following settings:

    - Display name: **add-on-plan2**
    - Resource name: **add-on-plan2**
    - Resource group: the name of a new resource group **add-on-plans-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.KeyVault** checkbox.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, in the **Microsoft.KeyVault** drop-down list, select **Unlimited**.
1. Click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.


#### Task 2: Assign the add-on plan to an existing offer (as a cloud operator)

In this task, you will:

- Assign the add-on plan to an existing offer (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal where you are signed in as CloudAdmin@azurestack.local, click **All resources**. 
1. In the list of resources, search for and select **base-offer2**.
1. On the **base-offer2** blade, select **Add-on plans**.
1. On the **base-offer2 \| Add-on plans** blade, click **+ Add**.
1. On the **Plan** blade, click the **add-on-plan2** and click **Select**. 

>**Review**: In this exercise, you have created an add-on plan and assigned it to an existing offer associated with the subscription of the second user.


### Exercise 4: Check availability of the add-on plan (as the second user)

In this exercise, you will act as users who signed up for the offer you created in the first exercise and created new subscriptions. The exercise consists of the following tasks:

1. Modify the subscription by adding the add-on plan (as the second user)
1. Verify the functionality of the add-on plan (as the second user)

#### Task 1: Modify the subscription by adding the add-on plan (as the second user)

In this task, you will:

- Check availability of the add-on plan (as the second user)

1. Within the Remote Desktop session to **AzS-HOST1**, start an InPrivate session of the web browser.
1. In the web browser window, navigate to the [Azure Stack Hub user portal](https://portal.local.azurestack.external) and sign in as **t1u2@azurestack.local** with the password **Pa55w.rd**.
1. In the hub menu of the Azure Stack Hub user portal, click **All services**.
1. In the list of services, click **Subscriptions**.
1. On the **Subscriptions** blade, click **t1u2-base-subscription1**.
1. On the **t1u2-base-subscription1** blade, click **+ Add plan**.
1. On the **Add plan** blade, select **add-on-plan2**. If propmted, refresh the web browser page displaying the Azure Stack user portal.
1. Back on the **t1u2-base-subscription1** blade, click **Add-on plans**.
1. On the **t1u2-base-subscription2 | Add-on plans** blade, note the entry representing the **add-on-plan2**.

#### Task 2: Verify the functionality of the add-on plan (as the second user)

In this task, you will:

- Check availability of the add-on plan (as the second user)

1. Within the Remote Desktop session to **AzS-HOST1**, in the InPrivate session of a web browser displaying the Azure Stack Hub user portal where you are signed in as t1u2@azurestack.local, 
1. On the **t1u2-base-subscrption1** blade, click **Resources**.
1. On the **t1u2-base-subscrption1 - Resources** blade, click **+ Add**.
1. On the **New** blade, click **Security + Identity** and then click **Key Vault**.
1. On the **Basics** tab of the **Create key vault** blade, specify the following settings (leave others with their default values):

    - Resource group: **kv-RG**
    - Key vault name: any unique name consisting of between 3 and 24 alphanumeric characters and dashes, starting with a letter

1. Click **Review + create** and then click **Create**.
1. Verify that the deployment was successful.
1. Sign out from the Azure Stack Hub user portal and close the InPrivate session of a web browser.

>**Review**: In this exercise, you have verified the functionality of the add-on plan.
