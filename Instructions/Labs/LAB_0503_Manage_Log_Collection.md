---
lab:
    title: 'Lab: Manage Log Collection in Azure Stack Hub'
    module: 'Module 5: Manage Infrastructure'
---

# Lab - Manage Log Collection in Azure Stack Hub
# Student lab manual

## Lab dependencies

- None

## Estimated Time

30 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to identify methods to perform log collection.

## Objectives

After completing this lab, you will be able to:

- Perform Azure Stack Hub log collection

## Lab Environment 

The lab environment consists of the following components:

- ASDK deployment running on the **AzSHOST-1** server with the following access points:

  - Administrator portal: https://adminportal.local.azurestack.external
  - Admin ARM endpoint: https://adminmanagement.local.azurestack.external
  - User portal: https://portal.local.azurestack.external
  - User ARM endpoint: https://management.local.azurestack.external

- Administrative users:

  - ASDK cloud operator username: **CloudAdmin@azurestack.local**
  - ASDK cloud operator password: **Pa55w.rd1234**
  - ASDK host administrator username: **AzureStackAdmin@azurestack.local**
  - ASDK host administrator password: **Pa55w.rd1234**


### Exercise 1: Explore Azure Stack Hub diagnostic logs collection capabilities

In this exercise, you will explore different options for managing diagnostic logs collection options.

1. Enable proactive diagnostic log collection
1. Send diagnostic logs on demand
1. Copy diagnostic logs to a local file share

#### Task 1: Enable proactive diagnostic log collection

In this task, you will:

- Enable proactive log collection.

>**Note**: Proactive log collection automatically collects and sends diagnostic logs from Azure Stack Hub to Microsoft before you open a support case. These logs are only collected when a system health alert is raised and are only accessed by Microsoft Support in the context of a support case.

1. If needed, sign in to **AzSHOST-1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzSHOST-1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the web browser window displaying the Azure Stack Hub administrator portal, in the hub menu, click **Help + support**.
1. On the **Overview** blade, click **Log Collection**.
1. On the **Overview \| Log Collection** blade, click **Enable proactive log collection**.

    >**Note**: If the **Enable proactive log collection** option is not available, proceed directly to the next step. 

1. On the **Settings** blade, specify the following settings and click **Save**.

    - Proactive log collection: **Enable**
    - Log location: **Azure (Recommended)**

1. Back on the **Overview \| Log Collection** blade, click **Settings** to verify that the proactive log collection is configured according to settings you specified.


#### Task 2: Send diagnostic logs on demand

In this task, you will:

- Send diagnostic logs on demand by using the Azure Stack Hub administration portal.
- Send diagnostic logs on demand by using Azure Stack Hub PowerShell. 

>**Note**: This functionality is referred to as *Send logs now*

>**Note**: You will start by sending diagnostic logs on demand by using the Azure Stack Hub administration portal.

1. Within the Remote Desktop session to **AzSHOST-1**, in the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/), in the hub menu, click **Help + support**.
1. On the **Overview** blade, click **Log Collection**.
1. On the **Overview \| Log Collection** blade, click **Send logs now**.
1. On the **Send logs now** blade, specify the following settings and click **Collect + upload**.

    - Start: current date and time - 3 hours
    - End: current date and time

    >**Note**: Do not wait for the upload to complete but instead proceed to the next step. The log collection will fail. This is expected since the target Azure subscription is not directly accessible from the lab environment.

    >**Note**: Now you will configure the equivalent functionality by using Azure Stack Hub PowerShell. This requires connecting to the privileged endpoint.

1. Within the Remote Desktop session to **AzSHOST-1**, start PowerShell ISE as administrator.
1. From the **Administrator: Windows PowerShell ISE** console, run the following to identify the IP address of the infrastructure VM running the privileged endpoint:

    ```powershell
    $ipAddress = (Resolve-DnsName -Name AzS-ERCS01).IPAddress
    ```

1. From the **Administrator: Windows PowerShell ISE** window, run the following to add the IP address of the infrastructure VM running the privileged endpoint to the list of WinRM trusted hosts (unless all hosts are already allowed):

    ```powershell
    $trustedHosts = (Get-Item -Path WSMan:\localhost\Client\TrustedHosts).Value
    If ($trustedHosts -ne '*') {
	If ($trustedHosts -ne '') {
		$trustedHosts += ",ipAddress"
	} else {
	$trustedHosts = "$ipAddress"
	}
    }
    Set-Item WSMan:\localhost\Client\TrustedHosts -Value $TrustedHosts -Force
    ```

1. From the **Administrator: Windows PowerShell ISE** window, run the following to store the Azure Stack Hub admin credentials in a variable:

    ```powershell
    $adminUserName = 'CloudAdmin@azurestack.local'
    $adminPassword = 'Pa55w.rd1234' | ConvertTo-SecureString -Force -AsPlainText
    $adminCredentials = New-Object PSCredential($adminUserName,$adminPassword)
    ```

1. From the **Administrator: Windows PowerShell ISE** window, run the following to establish a PowerShell Remoting session to the privileged endpoint:

    ```powershell
    $session = New-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $adminCredentials
    Enter-PSSession -Session $session
    ```

    >**Note**: Verify that the PowerShell Remoting session has been successfully established. The console pane in the Windows PowerShell ISE window should be displaying the prompt starting with the IP address of the infrastructure VM running the privileged endpoint enclosed in square brackets.

1. From the PowerShell Remoting session in the console pane of the **Administrator: Windows PowerShell ISE** window, run the following to send Azure Stack Hub storage diagnostic logs on demand:

    ```powershell
    Send-AzureStackDiagnosticLog -FilterByRole Storage
    ```

    >**Note**: Do not wait for the upload to complete but instead proceed to the next step. The log collection might fail if the target Azure subscription is not directly accessible from the lab environment.

1. From the PowerShell Remoting session in the console pane of the **Administrator: Windows PowerShell ISE** window, press **Ctrl**+**C** to terminate the log collection and then run the following to exit the interactive session (without terminating it):

    ```powershell
    Exit-PSSession
    ```

    >**Note**: You have the option of filtering by role as well as specify the time window for which logs should be collected. For details, refer to [Diagnostic log collection](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-diagnostics).


#### Task 3: Copy diagnostic logs to a local file share

In this task, you will:

- Copy diagnostic logs to a local file share by using the Azure Stack Hub administration portal.
- Copy diagnostic logs to a local file share by using Azure Stack Hub PowerShell. 

>**Note**: You will start by creating a file share to store logs.

1. Within the Remote Desktop session to **AzSHOST-1**, start File Explorer. 
1. In File Explorer, create a new folder **C:\\AzSHLogs**.
1. In File Explorer, right-click the **AzSHLogs** folder and, in the right-click menu, click **Properties**.
1. In the **AzSHLogs Properties** window, click the **Sharing** tab and then click **Advanced Sharing**.
1. In the **Advanced Sharing** dialog box, click **Share this folder** and then click **Permissions**.
1. In the **Permissions for AzSHLogs** window, ensure that the **Everyone** entry is selected and then click **Remove**.
1. Click **Add**, in the **Select Users, Computers, Service Accounts, or Groups** dialog box, type **CloudAdmins** and click **OK**.
1. Ensure that the **CloudAdmins** entry is selected and click the **Full Control** checkbox in the **Allow** column.
1. Click **Add**, in the **Select Users, Computers, Service Accounts, or Groups** dialog box, click **Locations**.
1. In the **Locations** dialog box, click the entry representing the local computer (**AzSHOST-1**) and click **OK**.
1. In the **Enter the object names to select** text box, type **Administrators** and click **OK**.
1. Ensure that the **Administrators** entry is selected, click the **Full Control** checkbox in the **Allow** column, and then click **OK**.
1. Back in the **Advanced Sharing** dialog box, click **OK**.
1. Back in the **AzSHLogs Properties** window, click the **Security** tab and click **Edit**.
1. Click **Add**, in the **Select Users, Computers, Service Accounts, or Groups** dialog box, type **CloudAdmins** and click **OK**.
1. In the **Permissions for AzSHLogs** dialog box, in the list of entries in the **Groups or user names** pane, click **CloudAdmins**, in the **Permissions for CloudAdmins** pane, click **Full Control** in the **Allow** column and then click **OK**.
1. Back in the **AzSHLogs Properties** window, click **Close**.

    >**Note**: Next, you can confgure diagnostic log collection to a file share from either the Azure Stack Hub administration portal or via the privileged endpoint.

1. Within the Remote Desktop session to **AzSHOST-1**, in the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/), in the hub menu, click **Help + support**.
1. On the **Overview** blade, click **Log Collection**.
1. On the **Overview \| Log Collection** blade, click **Settings**.
1. On the **Settings** blade, change the **Log location** option from **Azure (Recommended)** to **Local file share** and specify the following settings:

    - SMB fileshare path: **\\\\AzSHOST-1.azurestack.local\\AzSHLogs**
    - Username: **AZURESTACK\\AzureStackAdmin**
    - Password: **Pa55w.rd1234**

1. On the **Settings** blade, click **Save**.
1. Back on the **Overview \| Log Collection** blade, click **Send logs now**.
1. On the **Send logs now** blade, specify the following settings and click **Collect + upload**.

    - Start: current date and time - 3 hours
    - End: current date and time

    >**Note**: To view the progress of the log collection and upload, on the **Overview \| Log Collection** blade, click **Refresh**.

    >**Note**: Do not wait for the upload to complete but instead proceed to the next step. The log collection should succeed but it might take about 15 minutes for it to complete. 

    >**Note**: Now you will configure the equivalent functionality by using Azure Stack Hub PowerShell. This requires connecting to the privileged endpoint.

1. Within the Remote Desktop session to **AzSHOST-1**, switch back to the **Administrator: Windows PowerShell ISE** console from which you initiated the PowerShell Remoting session to the privileged endpoint in the previous task.
1. From the console pane of the **Administrator: Windows PowerShell ISE** window, run the following to copy Azure Stack Hub storage diagnostic logs to a local file share:

    ```powershell
    Invoke-Command -Session $session { Get-AzureStackLog -OutputSharePath '\\AzSHOST-1.azurestack.local\AzSHLogs' -OutputShareCredential $using:adminCredentials -FilterByRole Storage}
    ```

1. Wait until the cmdlet completes and, in File Explorer, review the content of the **C:\\AzSHLogs** folder.

    >**Note**: The folder should contain folders corresponding to each individual copy you initiated. The folders should have the names in the format **AzureStackLogs-*YYYYMMDDHHMMSS*-AZS-ERCS01**, where ***YYYYMMDDHHMMSS*** represents the timestamp of the copy.

1. From the PowerShell Remoting session prompt in the **Administrator: Windows PowerShell** window, run the following to close the session:

    ```powershell
    Close-PrivilegedEndpoint -TranscriptsPathDestination '\\AzSHOST-1.azurestack.local\AzSHLogs' -Credential $using:adminCredentials
    ```

>**Review**: In this exercise, you have established a PowerShell Remoting session to the privileged endpoint and used PowerShell cmdlets available from that endpoint to collect diagnostics logs.