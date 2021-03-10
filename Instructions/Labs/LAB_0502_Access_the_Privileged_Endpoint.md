---
lab:
    title: 'Lab: Access the Privileged Endpoint in Azure Stack Hub'
    module: 'Module 5: Manage Infrastructure'
---

# Lab - Access the Privileged Endpoint in Azure Stack Hub
# Student lab manual

## Lab dependencies

- None

## Estimated Time

30 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to identify the method of accessing the privileged endpoint.

## Objectives

After completing this lab, you will be able to:

- Access the Azure Stack Hub privileged endpoint 

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


### Exercise 1: Manage Azure Stack Hub via the privileged endpoint

In this exercise, you will establish a PowerShell Remoting session to the privileged endpoint to run Windows PowerShell cmdlets accessible via the Remoting session. The exercise consists of the following tasks:

1. Connect to the privileged endpoint via Windows PowerShell
1. Review the functionality available via the privileged endpoint
1. Close the connection to the privilege endpoints and collect the session transcript

#### Task 1: Connect to the privileged endpoint via Windows PowerShell

In this task, you will:

- Connect to the privileged endpoint via Windows PowerShell

1. If needed, sign in to **AzS-HOST1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzS-HOST1**, start PowerShell ISE as administrator.
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
    Enter-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $adminCredentials
    ```

1. Verify that the PowerShell Remoting session has been successfully established. The console pane in the Windows PowerShell ISE window should be displaying the prompt starting with the IP address of the infrastructure VM running the privileged endpoint enclosed in square brackets.


#### Task 2: Review the functionality available via the privileged endpoint

In this task, you will:

- Review the functionality available via the privileged endpoint.

1. Within the Remote Desktop session to **AzS-HOST1**, from the PowerShell Remoting session in the **Administrator: Windows PowerShell ISE** window, in the console pane, run the following to identify all available PowerShell cmdlets:

    ```powershell
    Get-Command
    ```

1. From the PowerShell Remoting session in the **Administrator: Windows PowerShell ISE** window, run the following to identify current Cloud Admin user accounts:

    ```powershell
    Get-CloudAdminUserList
    ```

    >**Note**: The list should include only two accounts - CloudAdmin and AzureStackAdmin.

1. From the PowerShell Remoting session in the **Administrator: Windows PowerShell ISE** window, run the following to validate update readiness of Azure Stack Hub and review the results:

    ```powershell
    Test-AzureStack -Group UpdateReadiness
    ```

    >**Note**: Keep in mind that ASDK does not support updates, so this is strictly for demonstration purposes. 

    >**Note**: For an introduction to the **Test-AzureStack** functionality, refer to [Validate Azure Stack Hub system state](https://docs.microsoft.com/en-us/azure-stack/operator/azure-stack-diagnostic-test?view=azs-2008).

    >**Note**: During support scenarios, a Microsoft support engineer might need to elevate the privileged endpoint PowerShell session to access the internals of the Azure Stack Hub infrastructure. This process is referred to as unlocking the privileged endpoint. The session elevation process is a two step, two people, two organization authentication process. The unlock procedure is initiated by the Azure Stack Hub operator, who retains control of their environment at all times. You will step through the emulated scenario illustrating this process next.

1. From the PowerShell Remoting session in the **Administrator: Windows PowerShell ISE** window, run the following to validate that the privileged endpoint is currently locked:

    ```powershell
    Get-SupportSessionInfo
    ```

1. From the PowerShell Remoting session in the **Administrator: Windows PowerShell ISE** window, run the following to generate the support session token:

    ```powershell
    Get-SupportSessionToken
    ```

    >**Note**: In a support scenario, you would passs the request token to a Microsoft support engineer via a medium of their choice, such as chat or email. The Microsoft support engineer then would use the request token to generate a support session authorization token and relay its value to you. Within the same PowerShell Remoting session, you would next run the **Unlock-SupportSession** cmdlet and, when prompted, provide the value of the support session authorization token. At that point, the PowerShell Remoting session would become elevated, with full admin capabilities and full reachability into the infrastructure.


#### Task 3: Close the session to the privilege endpoints and collect the session transcript

In this task, you will:

- Close the session to the privilege endpoints and collect the session transcript.

>**Note**: Privileged endpoint logs every action and its output. To collect the logs, close the session by using the **Close-PrivilegedEndpoint** cmdlet. This closes the endpoint and transfers the log files to an external file share for retention.

>**Note**: You will start by creating a file share to store the privileged endpoint logs.

1. Within the Remote Desktop session to **AzS-HOST1**, start another PowerShell ISE as administrator.
1. From the **Administrator: Windows PowerShell ISE** console, run the following to create and configure share that will store the privileged endpoint session logs:

    ```powershell
    $pepGroup = 'AZURESTACK\CloudAdmins'
    New-Item -Path 'C:\PEPLogs' -ItemType Directory -Force
    $pepShare = New-SmbShare -Name 'PEPLogs' -Description 'PEPLogs' -Path 'C:\PEPLogs'
    Grant-SmbShareAccess -Name $pepShare.Name -AccountName $pepGroup -AccessRight Full -Force
    Revoke-SmbShareAccess -Name $pepShare.Name -AccountName 'Everyone' -Force
    ```

1. Switch back to the PowerShell Remoting session in the **Administrator: Windows PowerShell ISE** window, run the following to close the privileged endpoint session and transfers the session log files to an external file share for retention:

    ```powershell
    Close-PrivilegedEndpoint -TranscriptsPathDestination '\\AzS-HOST1.azurestack.local\PEPLogs' -Credential $using:adminCredentials
    ```

1. Wait until the cmdlet completes and, in File Explorer, review the content of the **C:\\PEPLogs** folder.

>**Review**: In this exercise, you have established a PowerShell Remoting session to the privileged endpoint, reviewed its functionality, and closed the session.