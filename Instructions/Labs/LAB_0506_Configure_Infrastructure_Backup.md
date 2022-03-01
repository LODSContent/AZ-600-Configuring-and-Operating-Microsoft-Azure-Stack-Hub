---
lab:
    title: 'Lab: Configure Azure Stack Hub Infrastructure Backup'
    module: 'Module 5: Manage Infrastructure'
---

# Lab - Configure Azure Stack Hub Infrastructure Backup
# Student lab manual

## Lab depedndencies

- None

## Estimated Time

30 minutes

## Lab scenario

You are an operator of an Azure Stack Hub environment. You need to prepare it for infrastructure backup. 

## Objectives

After completing this lab, you will be able to:

- Configure Azure Stack Hub infrastructure backup

## Lab Environment 

This lab uses an ADSK instance integrated with Active Directory Federation Services (AD FS) (backed up Active Directory as the identity provider). 

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

You will install software necessary to manage Azure Stack Hub via PowerShell in the course of this lab. You will also create additional user accounts.

### Exercise 1: Configure Azure Stack Hub infrastructure backup

In this exercise, you will prepare configure Azure Stack Hub infrastructure backup:

1. Create a backup user
1. Create a backup share
1. Generate an encryption key
1. Configure backup controller

#### Task 1: Create a backup user

In this task, you will:

- Create a backup user

1. If needed, sign in to **AzSHOST-1** by using the following credentials:

    - Username: **AzureStackAdmin@azurestack.local**
    - Password: **Pa55w.rd1234**

1. Within the Remote Desktop session to **AzSHOST-1**, click **Start**, in the Start menu, click **Windows Administrative Tools**, and, in the list of administrative tools, double-click **Active Directory Administrative Center**.
1. In the **Active Directory Administrative Center** console, click **azurestack (local)**.
1. In the details pane, double-click the **Users** container.
1. In the **Tasks** pane, in the **Users** section, click **New -> User**.
1. In the **Create User** window, specify the following settings and click **OK**: 

    - Full name: **AzS-BackupOperator**
    - User UPN logon: **AzS-BackupOperator@azurestack.local**
    - User SamAccountName: **azurestack\AzS-BackupOperator**
    - Password: **Pa55w.rd**
    - Password options: **Other password options -> Password never expires**

#### Task 2: Create a backup share

In this task, you will:

- Create a backup share. 

    >**Note**: In non-lab scenarios, this share would be external to the Azure Stack Hub deployment. You will create it directly on the Azure Stack Hub host for the simplicity sake.

1. Within the Remote Desktop session to **AzSHOST-1**, start File Explorer. 
1. In File Explorer, create a new folder **C:\\Backup**.
1. In File Explorer, right-click the **Backup** folder and, in the right-click menu, click **Properties**.
1. In the **Backup Properties** window, click the **Sharing** tab and then click **Advanced Sharing**.
1. In the **Advanced Sharing** dialog box, click **Share this folder** and then click **Permissions**.
1. In the **Permissions for Backup** window, ensure that the **Everyone** entry is selected and then click **Remove**.
1. Click **Add**, in the **Select Users, Computers, Service Accounts, or Groups** dialog box, type **AzS-BackupOperator** and click **OK**.
1. Ensure that the **AzS-BackupOperator** entry is selected and click the **Full Control** checkbox in the **Allow** column.
1. Click **Add**, in the **Select Users, Computers, Service Accounts, or Groups** dialog box, click **Locations**.
1. In the **Locations** dialog box, click the entry representing the local computer (**AzSHOST-1**) and click **OK**.
1. In the **Enter the object names to select** text box, type **Administrators** and click **OK**.
1. Ensure that the **Administrators** entry is selected, click the **Full Control** checkbox in the **Allow** column, and then click **OK**.
1. Back in the **Advanced Sharing** dialog box, click **OK**.
1. Back in the **Backup Properties** window, click the **Security** tab and click **Edit**.
1. Click **Add**, in the **Select Users, Computers, Service Accounts, or Groups** dialog box, type **AzS-BackupOperator** and click **OK**.
1. In the **Permissions for Backup** dialog box, in the list of entries in the **Groups or user names** pane, click **AzS-BackupOperator**, in the **Permissions for AzS-BackupOperator** pane, click **Full Control** in the **Allow** column and then click **OK**. 
1. Back in the **Backup Properties** window, click **Close**.

#### Task 3: Generate an encryption key

In this task, you will:

- Generate an encryption key. 

    >**Note**: All infrastructure backups must be encrypted, so to configure an infrastructure backup you must provide a certificate corresponding to the encryption key pair. You will use Windows PowerShell to generate a key. 

1. Within the Remote Desktop session to **AzSHOST-1**, start Windows PowerShell as Administrator.
1. From the **Administrator: Windows PowerShell** prompt, run the following to generate the encryption key pair and the corresponding certificate:
    
    ```powershell
    $cert = New-SelfSignedCertificate `
               -DnsName "azsh.contoso.com" `
               -CertStoreLocation 'cert:\LocalMachine\My'

    New-Item -Path 'C:\' -Name 'CertStore' -ItemType 'Directory'

    Export-Certificate `
        -Cert $cert `
        -FilePath C:\CertStore\AzSHIBPK.cer 
    ```

    >**Note**: Azure Stack Hub supports self-signed certificate for the purpose of infrastructure backup. Keep in mind that you need to provide the private key during recovery, so in production environments, make sure to store it in a secure location.


#### Task 4: Configure backup controller

In this task, you will:

- Configure backup controller

1. Within the Remote Desktop session to **AzSHOST-1**, open the web browser window displaying the [Azure Stack Hub administrator portal](https://adminportal.local.azurestack.external/) and sign in as CloudAdmin@azurestack.local.
1. In the Azure Stack Hub administrator portal, click **All services**.
1. On the **All services** blade, select **Administration** and then select **Infrastructure backup**. 
1. On the **Infrastructure backup** blade, click **Configure**.
1. On the **Backup controller settings** blade, in the **Encryption Settings** section, next to the **Certificate .cer file** text box, select the folder icon, in the **Open** dialog box, navigate to the **C:\\CertStore** folder, select the **AzSHIBPK.cer** file, and select **Open**.
1. Back on the **Backup controller settings** blade, in the **Backup Storage Settings** section, select the **External file share** option.
1. On the **Backup controller settings** blade, specify the following settings and click **OK**:

    - Backup storage location: **\\AzSHOST-1.azurestack.local\Backup**
    - Username: **AzS-BackupOperator@azurestack.local**
    - Password: **Pa55w.rd**
    - Confirm password: **Pa55w.rd**
    - Backup frequency in hours: **12**
    - Retention period in days: **7**

1. To verify that the infrastructure backup has been enabled, refresh the web browser page displaying the Azure Stack Hub administrator portal and navigate back to the **Infrastructure backup** blade.
1. On the **Infrastructure backup** blade, review the backup settings.
1. Optionally, click **Backup now** to initiate on-demand backup.

    >**Note**: The backup process takes more than 15 minutes to complete and cannot be paused or stopped.

>**Review**: In this exercise, you have created a share that will host Azure Stack Hub infrastructure backups and an Active Directory user account that will be used by the Azure Stack Hub infrastructure backup to connect to that share, generated encryption keys, and configured backup controller. 