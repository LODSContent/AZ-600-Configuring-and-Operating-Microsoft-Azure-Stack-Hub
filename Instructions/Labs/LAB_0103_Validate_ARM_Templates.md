---
lab:
    title: 'Lab: Validate Azure Resource Manager (ARM) Templates with Azure Stack Hub'
    module: 'Module 1: Provide Services'
---

# Lab - Validate Azure Resource Manager (ARM) Templates with Azure Stack Hub
# Student lab manual

## Lab dependencies

- None

## Estimated Time

30 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to use your existing Azure Resource Manager (ARM) templates to automate deployment of Azure Stack Hub resources. 

## Objectives

After completing this lab, you will be able to:

- Validate ARM templates for Azure Stack Hub deployments.

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


### Exercise 1: Validate an ARM template with Azure Stack Hub

In this exercise, you will validate an ARM template with Azure Stack Hub.

1. Build a cloud capabilities file 
1. Run a successful template validation
1. Run a failed template validation
1. Remediate template issues


#### Task 1: Build a cloud capabilities file

In this task, you will:

- Build a cloud capabilities file

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

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

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to download the Azure Stack Tools:

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

    >**Note**: This step copies the archive containing the GitHub repository hosting the Azure Stack Hub tools to the local computer and expands the archive to the **C:\\AzureStack-Tools-master** folder. The tools contain PowerShell modules that offer a range of features, including validation of Azure Stack Hub Resource Manager templates. 

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to import the AzureRM.CloudCapabilities PowerShell module:

    ```powershell
    Import-Module .\CloudCapabilities\Az.CloudCapabilities.psm1
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to generate a cloud capabilities JSON file: 

    ```powershell
    $path = 'C:\Templates'
    New-Item -Path $path -ItemType Directory -Force
    Get-AzCloudCapability -Location 'local' -OutputPath $path\AzureCloudCapabilities.Json -Verbose
    ```

1. Within the Remote Desktop session to **AzS-HOST1**, start File Explorer, navigate to the **C:\\Templates** folder and verify that the file **AzureCloudCapabilities.Json** has been successfully created.


#### Task 2: Run a successful template validation

In this task, you will:

- Run template validator against an Azure Stack Hub Quickstart template

1. Within the Remote Desktop session to **AzS-HOST1**, start a web browser and navigate to the Azure Stack Hub QuickStart Templates repository [**MySql Server on Windows for AzureStack** page](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/mysql-standalone-server-windows). 
1. On the **MySql Server on Windows for AzureStack** page, click **azuredeploy.json**.
1. On the [AzureStack-QuickStart-Templates/mysql-standalone-server-windows/azuredeploy.json](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/mysql-standalone-server-windows/azuredeploy.json) page, click **Raw**.
1. Select the entire content of the web page, right click the selection, and click **Copy**.
1. Switch to the File Explorer window displaying the content of the **C:\\Templates** folder, create a new file named **sampletemplate1.json**, paste the copied content into it, and save the file.

    >**Note**: Alternatively, you can run the following command in order to directly download the content of the QuickStart template and save it as the **sampletemplate1.json** file.

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/master/mysql-standalone-server-windows/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate1.json
    ```

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to import the AzureRM.TemplateValidator PowerShell module:

    ```powershell
    Set-Location -Path 'C:\AzureStack-Tools-az\TemplateValidator'
    Import-Module .\AzureRM.TemplateValidator.psm1
    ```
  
1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to validate the template:

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate1.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate1validationreport.html `
        -Verbose
    ```

1. Review the output of the validation and verify that there are no issues.

    >**Note**: The output should have the following format:

    ```
    Validation Summary:
        Passed: 1
        NotSupported: 0
        Exception: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate1validationreport.html
    ```

#### Task 3: Run a failed template validation

In this task, you will:

- Run template validator against an Azure Quickstart template

1. Within the Remote Desktop session to **AzS-HOST1**, from the web browser displaying the AzureStack QuickStart Templates repository, navigate to the Azure QuickStart Templates repository [**MySQL Server 5.6 on Ubuntu VM** page](https://github.com/Azure/azure-quickstart-templates/tree/master/mysql-standalone-server-ubuntu)
1. On the **MySQL Server 5.6 on Ubuntu VM** page, click **azuredeploy.json**.
1. On the [azure-quickstart-templates/mysql-standalone-server-ubuntu/azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/mysql-standalone-server-ubuntu/azuredeploy.json) page, click **Raw**.
1. Select the entire content of the web page, right click the selection, and click **Copy**.
1. Switch to the File Explorer window displaying the content of the **C:\\Templates** folder, create a new file named **sampletemplate2.json**, paste the copied content into it.

    >**Note**: Alternatively, you can run the following command in order to directly download the content of the QuickStart template and save it as the **sampletemplate2.json** file.

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mysql-standalone-server-ubuntu/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate2.json
    ```

1. Within the Remote Desktop session to **AzS-HOST1**, in the web browser, navigate to the REST API reference for [Virtual Machines](https://docs.microsoft.com/en-us/rest/api/compute/virtualmachines) and identify the latest Azure API version (**2020-12-01** at the time of authoring this content). 
1. Switch to the **sampletemplate2.json** file and locate the section representing the virtual machine resource in the the `resources` section of the template, which has the following format:

    ```json
    {
      "apiVersion": "2018-03-30",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

1. Set the value of the `apiVersion` key to the latest Azure REST API version for virtual machines you identified earlier in this task and save the change, leaving the file open.

    >**Note**: Record the original value before you make the change. You will need to revert to it in the next task.

    >**Note**: Assuming the REST API version **2020-12-01**, the change should result in the following outcome:

    ```json
    {
      "apiVersion": "2020-12-01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

    >**Note**: This intentionally introduces a configuration which is not supported in Azure Stack Hub.

1. From the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window, run the following to validate the newly modified template:

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. Review the output of the validation and note that this time there is a supportability issue.

    >**Note**: The output should have the following format:

    ```
    Validation Summary:
        Passed: 0
        NotSupported: 1
        Exception: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html
    ```

1. Open the **C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html** file and review the report. 

    >**Note**: The report should contain an entry in the following format: **NotSupported: apiversion (Resource type: Microsoft.Compute/virtualMachines). Not Supported Values - 2020-12-01**.


#### Task 4: Remediate template issues

In this task, you will:

- Remediate template issues.

1. Within the Remote Desktop session to **AzS-HOST1**, in File Explorer, navigate to the **C:\\Templates** folder and open the **AzureCloudCapabilities.json** file.
1. In the **AzureCloudCapabilities.json** file, locate the `"ResourceTypeName": "virtualMachines",` section, which should have the following format:

    ```json
    {
      "ProviderNamespace": "Microsoft.Compute",
      "ResourceTypeName": "virtualMachines",
      "Locations": [
        "local"
      ],
      "ApiVersions": [
        "2020-06-01",
        "2019-12-01",
        "2019-07-01",
        "2019-03-01",
        "2018-10-01",
        "2018-06-01",
        "2018-04-01",
        "2017-12-01",
        "2017-03-30",
        "2016-08-30",
        "2016-03-30",
        "2015-11-01",
        "2015-06-15"
      ],
      "ApiProfiles": [
        "2017-03-09-profile",
        "2018-03-01-hybrid"
      ]
    },
    ```

1. Switch to the **sampletemplate2.json** file and change the value of the REST API version you modified in the previous task to its original value. Ensure that this version matches one of the API versions in the **AzureCloudCapabilities.json** listed above and save the change.
1. Switch to the **Administrator: C:\Program Files\PowerShell\7\pwsh.exe** window and run the following to validate the newly modified template:

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. Review the output of the validation and note that this time there are no issues.

>**Review**: In this exercise, you have created a cloud capabilities file and used it to validate Azure Resource Manager templates. You also modified a template based on the result of the validation.