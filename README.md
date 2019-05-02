# Moving Virtual Machines Between Azure Subscriptions
<sub>Author: Daniel Grecoe, A Microsoft employee</sub>

Moving virtrual machines between subscriptions is not something you can accomplish with the Azure Portal. It requires a bit of work to get the parts in place correctly. 

The script ***MoveVM.ps1*** in this directory can accomplish moving a VM between two Azure Subscriptions where the user has access rights to both subscriptions. 

During this process, a copy of the virtual machine is created in a destination 
subscription. The original machine is not affected.

### Use Cases ###
1. Copy a Virtual Machine from one subscription to antoher.
2. Copy a Virtual Machine within a subscription.

### What next? ###
- Finish reading this document.
- Identify a Virtual Machine you want to move and to where.
- Create the destination resource group and virtual network.
- Modify the ***MoveVMConfig.json*** file with your settings.
- Run the ***MoveVM.ps1*** script.

# Configuration Requirements
The file ***MoveVMConfig.json*** has several settings that you, the user, will have to collect and replace before calling the ***MoveVM.ps1*** script.

|Parameter|Description|
|-------------------|----------------------|
|SOURCE_SUBSCRIPTION_ID|The subscription ID where the Virtual Machine resides in now.|
|SOURCE_SUBSCRIPTION_RESOURCE_GROUP|The Azure Resource Group that contains the Virtual Machine to move.|
|SOURCE_MACHINE|The name of the Virtual Machine in the above Azure Resource Group|
|DESTINATION_SUBSCRIPTION_ID|The subscription ID where the VM will be re-constituted. If copying to the same subscription, this value will be equal to SOURCE_SUBSCRIPTION_ID|
|DESTINATION_SUBSCRIPTION_RESOURCE_GROUP|The Azure Resource Group that will hold the copied Virtual Machine <sup>1</sup>|
|DESTINATION_SUBSCRIPTION_VIRTUAL_NETWORK|The Azure Virtual Network to attach to the copied Virtual Machine <sup>2</sup>|

<sup>1</sup> If this resource group exists in the subscription, it is used. Otherwise it is created for you in the same region that the source Virtual Machine resides. If you have created it already, it should also be in the same region as the source Virtual Machine.

<sup>2</sup> If this VNET exists (resource checked by name only in the destination resource group) it is used. If it does not exist it is created with a single default subnet with the address space of 172.30.25.0/24. If created on your behalf, it should be in the same region as the source Virtual Machine and you may need to modify the address space. 

# The Process
The following are the steps that are taken by the scripts when the Virtual machine is re-created. 

<b>NOTE</b> The source virtual machine will NOT be removed or stopped during this process. A new copy of the virtural machien will be created in the destination resource group. It is up to the caller to delete the source virtual machine when they have verified that the move was succesful. 

<b>NOTE</b> Before calling the script, ensure you are logged into Azure. Mine happens automatically, you may ned to issue the Login-AzureRMAccount before proceeding.

1. Set the context to the source subscription.
2. Get the source virtual machine information.
3. Create a snapshot of the os disk of the source machine.
4. Create a managed disk from the snapshot.
    - <b>You will be prompted to confirm the move when the scritpt runs. </b>
5. Determine if the destination resource group exists, if not create it.
6. Determine if the virtual network exists, if not create it.
7. Copy the managed disk to the identified resource group in the destination subscription.
8. Set the context to the destination subscription.
9. Create the virtual machine
    - New Virtual Machine will have the same:
        - Disk Size
        - VM SKU
        - Name
    - Create the network information:
        - New public IP
        - Virtual network
        - New network security group    
        - New network interface
    - Virtual machine creation