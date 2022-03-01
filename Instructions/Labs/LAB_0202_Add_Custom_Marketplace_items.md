---
lab:
    title: 'Lab: Add custom Marketplace Items by using Azure Gallery Packager'
    module: 'Module 2: Provide Services'
---

# Lab - Add custom Marketplace Items by using the Azure Gallery Packager
# Student lab manual

## Lab dependencies

- None 

## Estimated Time

45 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to create custom Azure Stack Marketplace items by using the Azure Gallery Packager tool.

## Objectives

After completing this lab, you will be able to:

- Create custom Azure Stack Hub Marketplace items by using the Azure Gallery Packager

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

You will download and install software necessary to create custom Azure Stack Marketplace items in the course of this lab.

### Exercise 1: Customize and publish Azure Stack Hub Marketplace items

In this exercise, you will customize and publish Azure Marketplace items by using the Azure Gallery Packager tool.

1. Download the Azure Gallery Packager tool and sample packages
1. Modify an existing Azure Gallery Packager package
1. Upload the package to an Azure Stack Hub storage account 
1. Publish the package to Azure Stack Hub Marketplace
1. Verify availability of the published Azure Stack Hub Marketplace item

#### Task 1: Download the Azure Gallery Packager tool and sample packaging files

In this task, you will:

- Download the Azure Gallery Packager tool and sample packaging files

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzS-HOST1**, open a web browser window, navigate to the [Azure Gallery Packager tool download page](https://aka.ms/azsmarketplaceitem) and download the **Microsoft Azure Stack Gallery Packaging Tool and Sample 3.0.zip** archive file to the **Downloads** folder.
1. Once the download completes, extract the **Packager** folder within the zip file into the **C:\\Downloads** folder (create the folder if needed).


#### Task 2: Modify an existing Azure Gallery Packager package

In this task, you will:

- Modify an existing Azure Gallery Packager package, such that it will use a Windows Server 2019 image (rather than Windows Server 2016).

1. Within the Remote Desktop session to **AzS-HOST1**, in File Explorer, navigate to the **C:\\Downloads\\Packager\\Samples for Packager** folder, copy the **Sample.SingleVMWindowsSample.1.0.0.azpkg** package to the **C:\\Downloads** folder, and rename its extension to **.zip**.
1. Extract the content of the **Sample.SingleVMWindowsSample.1.0.0.zip** archive to the **C:\\Downloads\\SamplePackage** folder (you will need to create it first).
1. In File Explorer, navigate to the **C:\\Downloads\\SamplePackage\\DeploymentTemplates** folder and open the **createuidefinition.json** file in Notepad.
1. Modify the **sku** parameter in the **imageReference** section of the file such that it references a Windows Server 2019 Datacenter Core image which was downloaded earlier to the ASDK instance:

    ```json
    "imageReference": {
      "publisher": "MicrosoftWindowsServer",
      "offer": "WindowsServer",
      "sku": "2019-Datacenter-Core-smalldisk"
    },
    ```

1. Save the changes and close Notepad.
1. In File Explorer, navigate to the **C:\\Downloads\\SamplePackage\\strings** folder and open the **resources.resjson** file in Notepad.
1. Modify the content of the **resources.resjson** file by setting the following values in the key-value pairs :

    - displayName: **Custom Windows Server 2019 Core VM**
    - publisherDisplayName: **Contoso**
    - summary: **Custom Windows Server 2019 Core VM (small disk)**
    - longSummary: **Custom Contoso Windows Server 2019 Core VM (small disk)**
    - description: **<p>Sample customized Windows Server 2019 Azure Stack Hub VM</p><p>Based on Azure Stack Hub Quickstart template</p>**
    - documentationLink: **Documentation**

    >**Note**: This should yield the following content:

    ```json
    {
      "displayName": "Custom Windows Server 2019 Core VM",
      "publisherDisplayName": "Contoso",
      "summary": "Custom Windows Server 2019 Core VM (small disk)",
      "longSummary": "Custom Contoso Windows Server 2019 Core VM (small disk)",
      "description": "<p>Sample customized Windows Server 2019 Azure Stack Hub VM</p><p>Based on Azure Stack Hub Quickstart template</p>",
      "documentationLink": "Documentation"
    }
    ```

1. Save the changes and close Notepad.
1. In File Explorer, navigate to the **C:\\Downloads\\SamplePackage** folder and open the **manifest.json** file in Notepad.
1. Modify the content of the **manifest.json** file by setting the following values in the key-value pairs :

    - name: **CustomVMWindowsSample**
    - publisher: **Contoso**
    - version: **1.0.1**
    - displayName: **Custom Windows Server 2019 Core VM**
    - publisherDisplayName: **Contoso**
    - publisherLegalName: **Contoso Inc.**
    - uri of the documentation link: **https://docs.contoso.com**

    >**Note**: This should yield the following content (starting with the line 2 of the file):

    ```json
        "$schema": "https://gallery.azure.com/schemas/2015-10-01/manifest.json#",
        "name": "CustomVMWindowsSample",
        "publisher": "Contoso",
        "version": "1.0.1",
        "displayName": "Custom Windows Server 2019 Core VM",
        "publisherDisplayName": "Contoso",
        "publisherLegalName": "Contoso Inc.",
        "summary": "ms-resource:summary",
        "longSummary": "ms-resource:longSummary",
        "description": "ms-resource:description",
        "longDescription": "ms-resource:description",
	    "uiDefinition": {
		"path": "UIDefinition.json"
	},
        "links": [
            { "displayName": "ms-resource:documentationLink", "uri": "https://docs.contoso.com/" }
        ], 
    ```

1. Save the changes and close Notepad.


#### Task 3: Generate the customized Azure Gallery Packager package

In this task, you will:

- Regenerate the newly customized Azure Gallery Packager package.

1. Within the Remote Desktop session to **AzS-HOST1**, start **Command Prompt**.

1. From the **Command Prompt**, run the following to change the current directory:

    ```cmd
    cd C:\Downloads\Packager
    ```

1. From the **Command Prompt**, run the following to generate a new package based on the content you modified in the previous task:

    ```cmd
    AzureStackHubGallery.exe package -m C:\Downloads\SamplePackage\manifest.json -o C:\Downloads\
    ```

1. Verify that the **Contoso.CustomVMWindowsSample.1.0.1.azpkg** package was automatically saved to the **C:\\Downloads** folder.


#### Task 4: Upload the package to an Azure Stack Hub storage account

In this task, you will:

- Upload the Azure Stack Hub Marketplace item package to an Azure Stack Hub storage account.

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the web browser window displaying the Azure Stack Hub administrator portal, click **+ Create a resource**. 
1. On the **New** blade, click **Data + Storage**.
1. On the **Data + Storage** blade, click **Storage account**.
1. On the **Basics** tab of the **Create storage account** blade, specify the following settings:

    - Subscription: **Default Provider Subscription**
    - Resource group: the name of a new resource group **marketplace-pkgs-RG**
    - Name: a unique name consisting of between 3 and 24 lower case letters or digits
    - Location: **local**
    - Performance: **Standard**
    - Account kind: **Storage (general purpose v1)**
    - Replication: **Locally-redundant storage (LRS)**

1. On the **Basics** tab of the **Create storage account** blade, click **Next: Advanced >**.
1. On the **Advanced** tab of the **Create storage account** blade, leave the default settings in place and click **Review + create**.
1. On the **Review + create** tab of the **Create storage account** blade, click **Create**.

    >**Note**: Wait until the storage account is provisioned. This should take about one minute.

1. In the web browser window displaying the Azure Stack Hub administrator portal, in the hub menu, select **Resource groups**.
1. On the **Resource group** blade, in the list of resource groups, click the **marketplace-pkgs-RG** entry.
1. On the **marketplace-pkgs-RG** blade, click the entry representing the newly created storage account.
1. On the storage account blade, click **Containers**.
1. On the Containers blade, click **+ Container**.
1. On the **New container** blade, in the **Name** textbox, type **gallerypackages**, in the **Public access level** drop down list, select **Blob (anonymous read access for blobs only)**, and click **Create**.
1. Back on the Containers blade, click the **gallerypackages** entry representing the newly created container.
1. On the **gallerypackages** blade, click **Upload**.
1. On the **Upload blob** blade, click the folder icon next to the **Select a file** text box. 
1. In the **Open** dialog box, navigate to the **C:\Downloads** folder, select the **Contoso.CustomVMWindowsSample.1.0.1.azpkg** package file and click **Open**.
1. Back on the **Upload blob** blade, click **Upload**.


#### Task 5: Publish the package to Azure Stack Hub Marketplace

In this task, you will:

- Publish the package to Azure Stack Hub Marketplace.

1. Within the Remote Desktop session to **AzS-HOST1**, start Windows PowerShell as administrator.
1. Within the Remote Desktop session to **AzS-HOST1**, from the **Administrator: Windows PowerShell** prompt, run the following to install the Azure Stack Hub PowerShell modules required for this lab: 

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force
    Install-AzProfile -Profile 2020-09-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.2.0 
    ```

    >**Note**: Disregard any error messages regarding already available commands.

1. From the **Administrator: Windows PowerShell** prompt, run the following to register your Azure Stack Hub operator environment:

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

1. From the **Administrator: Windows PowerShell** prompt, run the following to sign in to your Azure Stack Hub operator PowerShell environment with the AzureStack\CloudAdmin credentials by leveraging your existing browser session to the Azure Stack Hub administrator portal:

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin'
    ```

    >**Note**: This will automatically open another browser tab displaying the message informing you about successful authentication.

1. Close the browser tab, switch back to the **Administrator: Windows PowerShell** window and verify that you have successfully authenticated as **CloudAdmin@azurestack.local**.
1. From the Administrator: **Administrator: Windows PowerShell**, run the following to publish the package to Azure Stack Hub Marketplace (where the `<storage_account_name>` placeholder represents the name of the storage account you assigned in the previous task):

    ```powershell
    Add-AzsGalleryItem -GalleryItemUri `
    https://<storage_account_name>.blob.local.azurestack.external/gallerypackages/Contoso.CustomVMWindowsSample.1.0.1.azpkg -Verbose
    ```

1. Review the output of the **Add-AzsGalleryItem** command to verify that the command completes successfully.


#### Task 6: Verify availability of the published Azure Stack Hub Marketplace item

In this task, you will:

- Verify availability of the published Azure Stack Hub Marketplace item.

1. Within the Remote Desktop session to **AzS-HOST1**, open the web browser window displaying the [Azure Stack Hub user portal](https://portal.local.azurestack.external).
1. In the web browser window displaying the Azure Stack Hub administrator portal, click **Marketplace**. 
1. On the **Marketplace** blade, click **Compute** and then click **See More**.
1. On the **Marketplace** blade, ensure that the **Custom Windows Server 2019 Core VM** appears in the list of choices. 

    >**Note**: In order to ensure that the new Azure Stack Hub Marketplace item is functioning as intended, you would also need to ensure that all of the prerequisites for its deployment, such as OS images referenced by its template, are in place. This is beyond the scope of this lab.

>**Review**: In this exercise, you have created a custom Azure Stack Hub Marketplace item by using the Azure Gallery Packager and published it by using the **Add-AzsGalleryItem** cmdlet.
