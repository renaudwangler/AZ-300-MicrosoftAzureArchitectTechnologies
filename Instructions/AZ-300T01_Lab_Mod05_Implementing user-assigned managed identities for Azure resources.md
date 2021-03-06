﻿# Managing Identities

# Lab: Implementing user-assigned managed identities for Azure resources

### Scenario

Adatum Corporation wants to use managed identities to authenticate applications running in Azure VMs

### Objectives

After completing this lab, you will be able to:

- Create and configure user-assigned managed identities

- Validate functionality of user-assigned managed identities

### Lab Setup

Estimated Time: 30 minutes

User Name: **Student**

Password: **Pa55w.rd**

## Exercise 1: Creating and configuring a user-assigned managed identity.

The main tasks for this exercise are as follows:

1. Deploy an Azure VM running Windows Server 2016 Datacenter

1. Create a user-assigned managed identity.

1. Assign the user-assigned managed identity to the Azure VM.

1. Grant RBAC-based permissions to the user-assigned managed identity.

#### Task 1: Deploy an Azure VM running Windows Server 2016 Datacenter

1. From the lab virtual machine, start Microsoft Edge and browse to the Azure portal at [**http://portal.azure.com**](http://portal.azure.com) and sign in by using the Microsoft account that has the Owner role in the target Azure subscription.

1. In the Azure portal, in the Microsoft Edge window, start a **Bash** session within the **Cloud Shell**.

1. If you are presented with the **You have no storage mounted** message, refer to Lab00

1. From the Cloud Shell pane, use the following command to copy the Azure Resource Manager template **azuredeploy05.json** into the home directory:

```sh
cp ./az-300/AZ-300T01/Module_05/azuredeploy05.json .
```

1. From the Cloud Shell pane, use the following command to copy parameter file **azuredeploy05.parameters.json** into the home directory.

```sh
cp ./az-300/AZ-300T01/Module_05/azuredeploy05.parameters.json .
```

1. From the Cloud Shell pane, deploy an Azure VM hosting Windows Server 2016 Datacenter into the first virtual network by running:

   ```
   az group deployment create --resource-group StagiaireXXX-RG1 --template-file azuredeploy05.json --parameters @azuredeploy05.parameters.json
   ```

   > **Note**: Wait for the deployment to complete. This might take about 5 minutes.

#### Task 2: Create a user-assigned managed identity and assign it to the Azure VM.

1. From the Cloud Shell pane, run the following to create a user-assigned managed identity:

   ```
   az identity create --resource-group StagiaireXXX-RG1 --name az3000501-mi
   ```

1. From the Cloud Shell pane, run the following to assign the user-assigned managed identity to the Azure VM:

   ```
   az vm identity assign --resource-group StagiaireXXX-RG1 --name az3000501-vm --identities az3000501-mi
   ```

#### Task 3: Configure RBAC referencing the user-assigned managed identity.

1. In the Azure portal, navigate to the **StagiaireXXX-RG2 - Access control (IAM)** blade.

1. From the **StagiaireXXX-RG2    - Access control (IAM)** blade, assign the Owner role to the newly created user-assigned managed identity.

> **Result**: After you completed this exercise, you have created and configured a user-assigned managed identity.

## Exercise 2: Validating functionality of user-assigned managed identities

The main tasks for this exercise are as follows:

1. Configure an Azure VM for authenticating via user-assigned managed identity.

1. Validate functionality of user-assigned managed identity from the Azure VM.

#### Task 1: Configure an Azure VM for authenticating via user-assigned managed identity.

1. In the Azure portal, navigate to the **az3000501-vm** blade.

1. Connect to the Azure VM by using Remote Desktop and authenticate by providing the following credentials:

   - Username: **Student**

   - Password: **Pa55w.rd1234**

1. Once you establish a Remote Desktop session, you will be presented with an **Administrator: C:\\Windows\\system32\\cmd.exe** window. To start a PowerShell session, at the command prompt, type `PowerShell` and press Enter.

1. From the PowerShell prompt, run the following to install the latest version of the PowerShellGet module (press Enter if prompted for confirmation):

   ```pwsh
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force
   ```

1. From the PowerShell prompt, run the following to install the latest version of the Az module (type **Y** and press Enter when prompted for confirmation):

   ```pwsh
   Install-Module -Name Az -AllowClobber
   ```

1. Exit the current PowerShell session by typing `exit` and pressing Enter and then start it again by typing at the command prompt `PowerShell` and pressing Enter.

1. From the PowerShell prompt, run the following to install the AzureRM.ManagedServiceIdentity module:

   ```pwsh
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name Az.ManagedServiceIdentity
   ```

#### Task 2: Validate functionality of user-assigned managed identity from the Azure VM.

1. From the PowerShell prompt, run the following to sign-in as the user-assigned managed identity:

   ```pwsh
   Add-AzAccount -Identity
   ```

1. From the PowerShell prompt, run the following to attempt to retrieve the currently used managed identity:

   ```pwsh
   (Get-AzVM -ResourceGroupName az3000501-LabRG -Name az3000501-vm).Identity
   ```

1. Note the error message. As the message states, the current security context does not grant sufficent authorization to the target resource. To resolve this issue, switch to the Azure portal, navigate to the **StagiaireXXX-RG1- Access control (IAM)** blade.

1. From the **StagiaireXXX-RG1 - Access control (IAM)** blade, assign the Contributor role to the user-assigned managed identity **az3000501-mi**.

1. Switch back to the Remote Desktop session, and, from the PowerShell prompt, run the following to attempt to retrieve the currently used managed identity:

   ```pwsh
   (Get-AzVM -ResourceGroupName StagiaireXXX-RG1  -Name az3000501-vm).Identity
   ```

   > **Note**: If you receive an error message indicating insufficient privileges, from the PowerShell prompt, run
   
   ```pwsh
   Remove-AzAccount
   ```
   
   followed by:
   
   ```pwsh
   Add-AzAccount -Identity
   ```
      
1. From the PowerShell prompt, run the following to store location in a variable:

   ```pwsh
   $location = (Get-AzResourceGroup -Name az3000502-LabRG).Location
   ```

1. From the PowerShell prompt, run the following to create a public IP address resource:

   ```pwsh
   New-AzPublicIpAddress -Name az3000502-pip -ResourceGroupName az3000502-LabRG -AllocationMethod Dynamic -Location $location
   ```

1. Verify that the command completed successfully.

> **Result**: After you completed this exercise, you have validated the functionality of the user-defined managed identity.
