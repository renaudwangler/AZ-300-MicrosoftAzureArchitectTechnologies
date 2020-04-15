# Evaluating and Performing Server Migration to Azure

# Lab: Implementing Azure to Azure migration

### Scenario

Adatum Corporation wants to migrate their existing Azure VMs to another region

### Objectives

After completing this lab, you will be able to:

- Implement Azure Site Recovery Vault

- Migrate Azure VMs between Azure regions

### Lab Setup

Estimated Time: 45 minutes

User Name: **Student**

Password: **Pa55w.rd**

## Exercise 1: Implement prerequisites for migration of Azure VMs by using Azure Site Recovery

The main tasks for this exercise are as follows:

1. Deploy an Azure VM to be migrated

1. Create an Azure Recovery Services vault

#### Task 1: Deploy an Azure VM to be migrated

1. From the lab virtual machine, start Microsoft Edge and browse to the Azure portal at [**http://portal.azure.com**](http://portal.azure.com) and sign in by using the Microsoft account that has the Owner role in the target Azure subscription.

1. In the Azure portal, in the Microsoft Edge window, start a **PowerShell** session within the **Cloud Shell**.

1. If you are presented with the **You have no storage mounted** message, configure storage using the following settings:

   - Subsciption: the name of the target Azure subscription

   - Cloud Shell region: the name of the Azure region that is available in your subscription and which is closest to the lab location

   - Resource group: use **StagiaireXXX-RG1**

   - Storage account: a name of a new storage account

   - File share: a name of a new file share

1. From the Cloud Shell pane, upload the Azure Resource Manager template **\\allfiles\\AZ-300T02\\Module_01\\azuredeploy06.json** into the home directory.

1. From the Cloud Shell pane, upload the parameter file **\\allfiles\\AZ-300T02\\Module_01\\azuredeploy06.parameters.json** into the home directory.

1. From the Cloud Shell pane, switch to your home directory :
   ```pwsh
   cd $home
   ```

1. From the Cloud Shell pane, deploy an Azure VM hosting Windows Server 2016 Datacenter by running:

   ```pwsh
   New-AzResourceGroupDeployment -ResourceGroupName StagiaireXXX-RG1 -TemplateFile azuredeploy06.json -TemplateParameterFile azuredeploy06.parameters.json
   ```

   > **Note**: Do not wait for the deployment to complete but instead proceed to the next task.


#### Task 2: Implement an Azure Recovery Service vault

1. From Azure Portal, create an instance of **Recovery Services vault** with the following settings:

   - Name: **vaultStagiaireXXX**

   - Subscription: the name of the target Azure subscription

   - Resource group: **StagiaireXXX-RG2**

1. Once the vault is provisioned (should take about one minute), navigate to the vault's **Overview** blade and select **Properties** in the left-hand pane.

1. On the **vaultStagiaireXXX | Properties** blade, click the **Update** link under **Security Settings**

1. In **Security Settings**, change the value of the **Soft Delete (For Azure Virtual Machines)** parameter to **Disabled**, then click **Save**.

> **Result**: After you completed this exercise, you have created an Azure VM to be migrated and an Azure Site Recovery vault that will host the migrated disk files of the Azure VM.

## Exercise 2: Migrate an Azure VM between Azure regions by using Azure Site Recovery

The main tasks for this exercise are as follows:

1. Configure Azure VM replication

1. Review Azure VM replication settings

1. Disable replication of an Azure VM and delete the Azure Recovery Services vault

#### Task 1: Configure Azure VM replication

1. In the the Azure portal, navigate to the blade of the newly provisioned Azure Recovery Services vault.

1. On the Recovery Services vault blade, click the **+ Replicate** button.

1. On the **Source** blade, select the following values, then click **OK**:
   
   - Source: **Azure**
   
   - Source location: the same Azure region into which you deployed the VM in the previous exercise
   
   - Azure virtual machine deployment model: **Resource Manager**
   
   - Source subscription: Leave the default value
   
   - Source resource group: **StagiaireXXX-RG1**

1. On the **Select virtual machines** blade, check the box next to **az3000601-vm** and click **OK**

1. On the **Configure settings** blade:

   - Change the **Target location** to the location associated with resource group **StagiaireXXX-RG2** (i.e. East US)

   - Click **Customize** next to **Resource group, Network, Storage and Availability**
   
   - On the **Customize target settings** blade, change the **Target resource group** to **StagiaireXXX-R2**, leave the other values as they are and click **OK**
   
   - Back on the **Confgure settings** blade, click **Customize** next to **Replication Policy**
   
   - On the **Configure replication settings** blade, set the follwing values for a new replication policy:
      - Name: **12-hour-retention policy**
      - Recovery point retention: **24 hours**
      - App consistent snapshot frequency: **6 hours**
      - Multi-VM consistency: **No**
   
   
   
   
   > **Note**: Wait for the operation of enabling the replication to complete. Then proceed to the next task.

#### Task 2: Review Azure VM replication settings

1. In the Azure portal, from the Azure Site Recovery vault blade, navigate to the replicated item blade representing the Azure VM **az3000601-vm**.

2. On the replicated item blade, review the **Health and status**, **Latest available recovery points**, and **Failover readiness** sections. Note the **Failover** and **Test Failover** entries in the toolbar. Scroll down to the **Infrastructure view**.

3. If time permits, wait until the status of the Azure VM changes to **Protected**. This might take additional 15-20 minutes. At that point, examine the values **Crash-consistent** and **App-consistent** recovery points. In order to view **RPO**, you should perform a test failover.

#### Task 3: Disable replication of an Azure VM and delete the Azure Recovery Services vault

1. In the Azure portal, disable replication of the Azure VM **az3000601-vm**.

2. Wait until the replication is disabled.

3. From the Azure portal, delete the Recovery Services vault.

   > **Note**: You must ensure that the replicated item is removed first before you can delete the vault.

> **Result**: After you completed this exercise, you have implemented automatic replication of an Azure VM.

## Exercise 3: Remove lab resources

#### Task 1: Open Cloud Shell

1. At the top of the portal, click the **Cloud Shell** icon to open the Cloud Shell pane.

1. If needed, switch to the Bash shell session by using the drop down list in the upper left corner of the Cloud Shell pane.

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to list all resource groups you created in this lab:

   ```
   az group list --query "[?starts_with(name,'az30006')].name" --output tsv
   ```

1. Verify that the output contains only the resource groups you created in this lab. These groups will be deleted in the next task.

#### Task 2: Delete resource groups

1. At the **Cloud Shell** command prompt, type in the following command and press **Enter** to delete the resource groups you created in this lab

   ```sh
   az group list --query "[?starts_with(name,'az30006')].name" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. Close the **Cloud Shell** prompt at the bottom of the portal.

> **Result**: In this exercise, you removed the resources used in this lab.
