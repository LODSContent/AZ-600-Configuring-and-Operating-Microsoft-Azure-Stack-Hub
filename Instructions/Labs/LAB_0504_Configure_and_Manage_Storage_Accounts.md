---
lab:
    title: 'Lab: Configure and manage Azure Stack Hub Storage Accounts'
    module: 'Module 5: Manage Infrastructure'
---

# Lab - Configure and manage Azure Stack Hub Storage Accounts
# Student lab manual

## Lab dependencies

- None

## Estimated Time

30 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to configure and manage its storage accounts.

## Objectives

After completing this lab, you will be able to:

- Configure Azure Stack Hub storage
- Manage Azure Stack Hub storage accounts

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

You will create an additional user accounts in the course of this lab.

### Exercise 1: Configure Azure Stack Hub storage

In this exercise, you will configure Azure Stack Hub storage:

1. Configure storage account retention
1. Create a plan containing Storage services
1. Create an offer based on the plan

#### Task 1: Configure storage account retention

In this task, you will:

- Configure storage account retention

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the web browser window displaying the Azure Stack Hub administrator portal, click **All services**.
1. On the **All services** blade, select the **Administration** section and click **Region management**.
1. On the **local** blade, click **Resource providers**.
1. On the **Resource providers** blade, click **Storage**.
1. On the **Storage** blade, click **Configuration**.
1. On the **Storage – Configuration** blade, in the **Retention period for deleted storage accounts (days)** text box, type 10 and click **Save**.


#### Task 2: Create a plan containing Storage services

In this task, you will:

- Create a plan containing Storage services.

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Plan**.
1. On the **Basics** tab of the **New plan** blade, specify the following settings:

    - Display name: **storage-plan1**
    - Resource name: **storage-plan1**
    - Resource group: the name of a new resource group **storage-plans-RG**

1. Click **Next: Services >**.
1. On the **Services** tab of the **New plan** blade, select the **Microsoft.Storage** checkbox.
1. Click **Next: Quotas>**.
1. On the **Quotas** tab of the **New plan** blade, next to the **Microsoft.Storage** drop-down list, click **Create New**.
1. On the **Create Storage quota** blade, specify the following settings and click **OK**":

    - Name: **storage-plan1-storage-quota**
    - Maximum capacity (GB): **200**
    - Total number of storage accounts: **5**

1. Click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.


#### Task 3: Create an offer based on the plan

In this task, you will:

- Create an offer based on the plan. 

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Offers + Plans**.
1. On the **Offers + Plans** blade, click **Offer**.
1. On the **Basics** tab of the **Create a new offer** blade, specify the following settings:

    - Display name: **storage-offer1**
    - Resource name: **storage-offer1**
    - Resource group: **storage-offers-RG**
    - Make this offer public: **Yes**

1. Click **Next: Base plans >**. 
1. On the **Base plans** tab of the **Create a new offer** blade, select the checkbox next to the **storage-plan1** entry.
1. Click **Next: Add-on plans >**.
1. Leave **Add-on plans** settings with their default values, click **Review + create** and then click **Create**.

    >**Note**: Wait for the deployment to complete. This should take just a few seconds.

>**Review**: After completing this exercise, you have configured the deleted storage account retention setting and created a public offer containing a **Microsoft.Storage** service-based plan. 


### Exercise 2: Manage Azure Stack Hub storage accounts.

In this exercise, you will manage Azure Stack Hub storage accounts:

1. Create and delete Azure storage accounts (as a tenant user)
1. Recover a deleted storage account (as a cloud operator)
1. Reclaim storage capacity (as a cloud operator)

#### Task 1: Create and delete Azure storage accounts (as a tenant user)

In this task, you will:

- Create and delete Azure storage accounts (as a tenant user) 

1. Within the Remote Desktop session to **AzS-HOST1**, start an InPrivate session of the web browser.
1. In the web browser window, navigate to the [Azure Stack Hub user portal](https://portal.local.azurestack.external) and sign in as **t1u1@azurestack.local** with the password **Pa55w.rd**.
1. In the Azure Stack Hub user portal, on the Dashboard, click the **Get a subscription** tile.
1. On the **Get a subscription** blade, in the **Name** text box, type **t1u1-storage-subscription1**.
1. In the list of offers, select **storage-offer1** and click **Create**.
1. When presented with the message **Your subscription has been created. You must refresh the portal to start using your subscription**, click **Refresh**. 
1. In the Azure Stack Hub tenant portal, in the hub menu, click **All services**.
1. In the list of services, click **Subscriptions**.
1. On the **Subscriptions** blade, click **t1u1-storage-subscription1**.
1. On the **t1u1-base-subscrption1** blade, click **Resources**.
1. On the **t1u1-storage-subscription1 - Resources** blade, click **+ Add**.
1. On the **New** blade, click **Data + Storage** and then click **Storage account**.
1. On the **Basics** tab of the **Create storage account** blade, specify the following settings (leave others with their default values):

    - Resource group: the name of a new resource group **storage-RG**
    - Name: a unique name consisting of between 3 and 24 lower case letters or digits
    - Location: **local**
    - Performance: **Standard**
    - Account kind: **Storage (general purpose v1)**
    - Replication: **Locally-redundant storage (LRS)**

1. On the **Basics** tab of the **Create storage account** blade, click **Next: Advanced >**.
1. On the **Advanced** tab of the **Create storage account** blade, leave the default settings in place and click **Review + create**.
1. On the **Review + create** tab of the **Create storage account** blade, click **Create**.

    >**Note**: Wait until the storage account is provisioned. This should take about one minute.

1. Create another storage account with the same settings in the same resource group by following the steps 11-16. 

    >**Note**: Next, you will delete both storage accounts. Make sure to record their names first. You will need them later in this lab.

1. In the Azure Stack Hub user portal, in the hub menu, click **All services** and then, on the **All services** blade, click **Storage accounts**.
1. On the **Storage accounts** blade, ensure that the subscription filter includes the **t1u1-storage-subscription1**, in the list of storage accounts, select both storage accounts you created earlier in this task, and click **Delete**.
1. On the **Delete Resources** blade, in the **Confirm delete** text box, type **yes** and click **Delete**.

#### Task 2: Recover a deleted storage account (as a cloud operator) 

In this task, you will:

- Recover a deleted storage account (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, switch to the web browser window displaying the Azure Stack Hub administrator portal, click **All services**, on the **All services** blade, click **Administration**, and, in the list of administration services, click **Storage**.
1. On the **Storage** blade, click **Storage accounts**.
1. On the **Storage \| Storage accounts** blade, in the **Account name** filter text box, type the name of the first storage account you created in the previous task and verify that the storage account name is listed with the **Deleted** status.
1. Click the entry representing the first deleted storage account.
1. On the **Storage account** blade, click **Recover**. When prompted to confirm, click **Yes**.

    >**Note**: Wait for the operation to complete. This should take less than a minute. Keep in mind that it might take some extra time for a recovered storage account to appear in the user portal.

1. Close the **Storage account** blade.


#### Task 3: Reclaim storage capacity (as a cloud operator) 

In this task, you will:

- Reclaim storage capacity (as a cloud operator)

1. Within the Remote Desktop session to **AzS-HOST1**, in the Azure Stack Hub administrator portal, while logged on as CloudAdmin@azurestack.local, on the **Storage \| Storage accounts** blade, click **Refresh**.
1. With the name of the first storage account in the **Account name** filter text box, verify that the storage account is now listed with the **Active** status.
1. On the **Storage \| Storage accounts** blade, in the **Account name** filter text box, type the name of the second storage account you created in the previous task and verify that the storage account name is listed with the **Deleted** status.
1. On the **Storage \| Storage accounts** blade, click **Reclaim space** and, when prompted to confirm, click **Yes**.

    >**Note**: Wait for the operation to complete. This should take less than a minute.

1. Refresh the browser page.
1. Back on the **Storage \| Storage accounts** blade, in the **Account name** filter text box, type the name of the second storage account you created in the previous task and verify that its name no longer appears in the list of search results. 

>**Review**: In this exercise, you have created and deleted storage accounts as a tenant user as well as recovered a deleted storage account and reclaimed storage capacity as a cloud operator.
