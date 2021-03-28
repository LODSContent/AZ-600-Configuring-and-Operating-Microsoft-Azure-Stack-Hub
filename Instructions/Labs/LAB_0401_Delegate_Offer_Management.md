---
lab:
    title: 'Lab: Delegate Offer Management in Azure Stack Hub'
    module: 'Module 4: Manage Identity and Access'
---

# Lab - Delegate Offer Management in Azure Stack Hub
# Student lab manual

## Lab dependencies

- None

## Estimated Time

45 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to delegate management of offers to your internal technical support staff.

## Objectives

After completing this lab, you will be able to:

- Implement Azure Stack Hub offer delegation.

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

1. Create a delegated admin user account (as a cloud operator)
1. Create a tenant user account (as a cloud operator)


#### Task 1: Create a delegated admin user account (as a cloud operator)

In this task, you will:

- Create a delegated admin user account (as a cloud operator)

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzS-HOST1**, click **Start**, in the Start menu, click **Windows Administrative Tools**, and, in the list of administrative tools, double-click **Active Directory Administrative Center**.
1. In the **Active Directory Administrative Center** console, click **azurestack (local)**.
1. In the details pane, double-click the **Users** container.
1. In the **Tasks** pane, in the **Users** section, click **New -> User**.
1. In the **Create User** window, specify the following settings and click **OK**: 

    - Full name: **DP1**
    - User UPN logon: **dp1@azurestack.local**
    - User SamAccountName: **azurestack\dp1**
    - Password: **Pa55w.rd**
    - Password options: **Other password options -> Password never expires**


#### Task 2: Create a tenant user account (as a cloud operator)

In this task, you will:

- Create a tenant user account (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the **Active Directory Administrative Center** console, in the **Tasks** pane, in the **Users** section, click **New -> User**.
1. In the **Create User** window, specify the following settings and click **OK**: 

    - Full name: **T1U1**
    - User UPN logon: **t1u1@azurestack.local**
    - User SamAccountName: **azurestack\t1u1**
    - Password: **Pa55w.rd**
    - Password options: **Other password options -> Password never expires**

>**Review**: In this exercise, you have created the Active Directory accounts you will use in this lab.


### Exercise 1: Designate the delegated provider (as a cloud operator)

In this exercise, you will act as a cloud operator and create a plan consisting of the **Subscription** service and a corresponding offer. Next, you will create a subscription containing the new offer and designate the delegated provider as its subscriber:

1. Create a plan consisting of the Subscription service (as a cloud operator)
1. Create an offer based on the plan (as a cloud operator)
1. Create a new subscription containing the offer (as a delegated provider)


#### Task 1: Create a plan consisting of the Subscription service (as a cloud operator)

In this task, you will:

- Create an plan consisting of the Subscription service (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Plan**.
1. On the **Basics** tab of the **New plan** blade, specify the following settings:

    - Display name: **co-subscription-plan**
    - Resource name: **co-subscription-plan**
    - Resource group: the name of a new resource group **co-subscription-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.Subscriptions** checkbox.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, in the **Microsoft.Subscriptions** dropdown list, select **delegatedProviderQuota**. 
1. Click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

#### Task 2: Create an offer based on the plan (as a cloud operator)

In this task, you will:

- Create an offer based on the plan (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Offer**.
1. On the **Basics** tab of the **Create a new offer** blade, specify the following settings:

    - Display name: **cotodp-subscription-offer**
    - Resource name: **cotodp-subscription-offer**
    - Resource group: **co-subscription-RG**
    - Make this offer public: **Yes**

1. Click **Next: Base plans >**. 
1. On the **Base plans** tab of the **Create a new offer** blade, select the checkbox next to the **co-subscription-plan** entry.
1. Click **Next: Add-on plans >**.
1. Leave **Add-on plans** settings with their default values, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

#### Task 3: Create a new subscription containing the offer and add the delegated provider as its subscriber (as a cloud operator)

In this task, you will:

- Create a new subscription containing the offer and add the delegated provider as its subscriber (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Subscription**.
1. On the **Create a user subscription** blade, , specify the following settings:

    - Name: **dp1-subscription1**
    - User: **dp1@azurestack.local**
    - Directory tenant: **ADFS.azurestack.local**

1. In the list of public offers, click **cotodp-subscription-offer** and then click **Create**

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

>**Review**: In this exercise, you have designated the delegated provider.


### Exercise 2: Create and delegate a delegated offer to a delegated provider (as a cloud operator)

In this exercise, you will act as a cloud operator and create an offer containing services, which the delegated provider will then offer to its tenants:

1. Create a plan to delegate (as a cloud operator)
1. Create an offer based on the plan (as a cloud operator)
1. Delegate the offer to the delegated provider (as a cloud operator)

#### Task 1: Create a plan to delegate (as a cloud operator)

In this task, you will:

- Create a plan to delegate (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Plan**.
1. On the **Basics** tab of the **New plan** blade, specify the following settings:

    - Display name: **co-services1-plan**
    - Resource name: **co-services1-plan**
    - Resource group: the name of a new resource group **co-services-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.Storage** checkbox.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, next to the **Microsoft.Storage** dropdown list, click **Create New**.
1. On the **Create Storage quota** blade, specify the following settings and click **OK**:

    - Name: **co-services1-storage-quota**
    - Maximum capacity (GB): **200**
    - Total number of storage accounts: **1**

1. Click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.


#### Task 2: Create an offer based on the plan (as a cloud operator)

In this task, you will:

- Create an offer based on the plan (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Offer**.
1. On the **Basics** tab of the **Create a new offer** blade, specify the following settings:

    - Display name: **cotodp-services1-offer**
    - Resource name: **cotodp-services1-offer**
    - Resource group: **co-services-RG**
    - Make this offer public: **Yes**

1. Click **Next: Base plans >**. 
1. On the **Base plans** tab of the **Create a new offer** blade, select the checkbox next to the **co-services1-plan** entry.
1. Click **Next: Add-on plans >**.
1. Leave **Add-on plans** settings with their default values, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.


#### Task 3: Delegate the offer to a delegated provider (as a cloud operator)

In this task, you will:

- Delegate the offer to the delegated provider (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, in the hub menu, click **Offers**. 
1. On the **Offers** blade, click **cotodp-services1-offer**.
1. On the **cotodp-services1-offer** blade, click **Delegated providers**.
1. On the **cotodp-services1-offer \| Delegated providers** blade, click **+ Add**.
1. On the **Delegate offer** blade, specify the following and click **Delegate**:

    - Name: **cotodp-services1-offer**
    - Pick the delegated provider subscription: **dp1-subscription1**

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

>**Review**: In this exercise, you have created and delegated the offer to a delegated provider.


### Exercise 3: Create a delegated offer to a tenant (as a delegated provider)

In this exercise, you will act as a delegated provider and create an offer (based on the offer from the cloud operator) to which your tenants will be able to subscribe:

1. Create a delegated provider offer for a tenant (as a cloud operator)
1. Identify the URL of the delegated provider portal (as a delegated provider)

#### Task 1: Create a delegated provider offer for a tenant (as a delegated provider)

In this task, you will:

- Create an offer for a tenant (as a delegated provider) based on the cloud operator offer for the delegated provider

This will allow tenants to create a subscription based on the offer from the delegated provider

1. Within the Remote Desktop session to **AzS-HOST1**, start an InPrivate session of the web browser.
1. In the web browser window, navigate to the [Azure Stack Hub user portal](https://portal.local.azurestack.external) and sign in as **dp1@azurestack.local** with the password **Pa55w.rd**.
1. In the Azure Stack Hub user portal, in the hub menu, click **All services** and, on the **All services** blade, click **Offers**.
1. On the **Offers** blade, click **+ Add**.
1. On the **Create a new offer** blade, specify the following settings:

    - Display name: **dp1tot-services1-offer**
    - Resource name: **dp1tot-services1-offer**
    - Provider subscription: **dp1-subscription1**
    - Resource group: the name of a new resource group **dp1-RG**

1. Click **Delegated** offer.
1. On the **Delegated offer** blade, click **cotodp-services1-offer**.
1. Back on the **Create a new offer** blade, click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

1. In the Azure Stack Hub user portal, back on **Offers** blade, click **Refresh** and click the newly created offer **dp1tot-services1-offer**.
1. On the **dp1tot-services1-offer** blade, click **Change state** and, in the drop-down list, click **Public**.


#### Task 2: Identify the URL of the delegated provider portal (as a delegated provider)

In this task, you will:

- Identify the URL of the delegated provider portal (as a delegated provider)

1. In the Azure Stack Hub user portal, while signed in as **dp1@azurestack.local**, in the hub menu, click **All services** and, on the **All services** blade, in the list of services, click **Subscriptions**.
1. On the **Subscriptions** blade, click **dp1-subscription1**.
1. On the **dp1-subscription1** delegated provider subscription blade, click **Properties**.
1. On the properties blade, copy the value of the **Portal URL** to clipboard. You will need it in the next exercise of this lab, 
1. In the Azure Stack Hub user portal, in the upper right corner, click the user avatar icon and, in the drop-down menu, click **Sign out**.

    >**Note**: Tenants need to subscribe to the offer from the delegated provider portal URL.

>**Review**: In this exercise, you have created a delegated provider offer.


### Exercise 4: Sign up for a delegated provider’s offer (as a tenant user)

In this exercise, you will act as a tenant user who signs up for a delegated provider’s offer and creates a resource in the new subscription. The exercise consists of the following tasks:

1. Sign up for the delegated provider’s offer (as a tenant user)
1. Create a resource in the new subscription (as a tenant user)

#### Task 1: Sign up for the delegated provider’s offer (as a tenant user)

In this task, you will:

- Sign up for the delegated offer (as a tenant user)

1. Within the Remote Desktop session to **AzS-HOST1**, start an InPrivate session of the web browser.
1. In the web browser window, navigate to the delegated provider portal URL you identified in the previous exercise and sign in as **t1u1@azurestack.local** with the password **Pa55w.rd**.

    >**Note**: Make sure not to use the standard Azure Stack Hub user portal URL in this case.

1. In the Azure Stack Hub user portal, click the **Get a subscription** tile.
1. On the **Get a subscription** blade, in the **Name** text box, type **t1u1-subscription1**.
1. On the **Get a subscription** blade, select the **dp1tot-services1-offer** offer and click **Create**.
1. When presented with the message **Your subscription has been created. You must refresh the portal to start using your subscription**, click **Refresh**.

#### Task 2: Create a resource in the new subscription (as a tenant user)

In this task, you will:

- Create a resource in the new subscription (as a tenant user)

1. Within the Remote Desktop session to **AzS-HOST1**, in the Azure Stack Hub user portal, while signed in as **t1u1@azurestack.local**, in the hub menu, click **All services** and, on the **All services** blade, in the list of services, click **Subscriptions**.
1. On the **Subscriptions** blade, click **t1u1-subscription1**.
1. On the **t1u1-subscription1** blade, click **Resources**.
1. On the **t1u1-subscription1 \| Resources** blade, click **+ Add**.
1. On the **New** blade, click **Data + Storage** and, in the list of resource types, click **Storage account**.
1. On the **Basics** tab of the **Create storage account** blade, specify the following settings:

    - Subscription: **t1u1-subscription1**
    - Resource group: the name of a new resource group **delegated-RG**
    - Storage account name: any unique name consisting of between 3 and 24 lower case letters or digits
    - Location: **local**
    - Performance: **Standard**
    - Account kind: **Storage (general purpose v1)**
    - Replication: **Locally-redundant storage (LRS)**

1. On the **Basics** tab of the **Create storage account** blade, click **Next: Advanced >**.
1. On the **Advanced** tab of the **Create storage account** blade, leave the default settings in place and click **Review + create**.
1. On the **Review + create** tab of the **Create storage account** blade, click **Create**.

    >**Note**: Wait until the storage account is provisioned. This should take about one minute.

1. Verify that the storage account has been successfully created.

>**Review**: In this exercise, you have subscribed to a delegated provider’s offer, creating this way a new subscription and provisioned a storage account in the new subscription.
