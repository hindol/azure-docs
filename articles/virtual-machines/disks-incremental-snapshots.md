---
title: Create an incremental snapshot
description: Learn about incremental snapshots for managed disks, including how to create them using the Azure portal, Azure PowerShell module, and Azure Resource Manager.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 08/10/2021
ms.author: rogarana
ms.subservice: disks 
ms.custom: devx-track-azurepowershell
---

# Create an incremental snapshot for managed disks

[!INCLUDE [virtual-machines-disks-incremental-snapshots-description](../../includes/virtual-machines-disks-incremental-snapshots-description.md)]

## Restrictions

[!INCLUDE [virtual-machines-disks-incremental-snapshots-restrictions](../../includes/virtual-machines-disks-incremental-snapshots-restrictions.md)]

# [Azure CLI](#tab/azure-cli)

You can use the Azure CLI to create an incremental snapshot. You will need the latest version of the Azure CLI. See the following articles to learn how to either [install](/cli/azure/install-azure-cli) or [update](/cli/azure/update-azure-cli) the Azure CLI.

The following script will create an incremental snapshot of a particular disk:

```azurecli
# Declare variables
diskName="yourDiskNameHere"
resourceGroupName="yourResourceGroupNameHere"
snapshotName="desiredSnapshotNameHere"

# Get the disk you need to backup
yourDiskID=$(az disk show -n $diskName -g $resourceGroupName --query "id" --output tsv)

# Create the snapshot
az snapshot create -g $resourceGroupName -n $snapshotName --source $yourDiskID --incremental true
```

You can identify incremental snapshots from the same disk with the `SourceResourceId` property of snapshots. `SourceResourceId` is the Azure Resource Manager resource ID of the parent disk.

You can use `SourceResourceId` to create a list of all snapshots associated with a particular disk. Replace `yourResourceGroupNameHere` with your value and then you can use the following example to list your existing incremental snapshots:


```azurecli
# Declare variables and create snapshot list
subscriptionId="yourSubscriptionId"
resourceGroupName="yourResourceGroupNameHere"
diskName="yourDiskNameHere"

az account set --subscription $subscriptionId

diskId=$(az disk show -n $diskName -g $resourceGroupName --query [id] -o tsv)

az snapshot list --query "[?creationData.sourceResourceId=='$diskId' && incremental]" -g $resourceGroupName --output table
```


# [Azure PowerShell](#tab/azure-powershell)

You can use the Azure PowerShell module to create an incremental snapshot. You will need the latest version of the Azure PowerShell module. The following command will either install it or update your existing installation to latest:

```PowerShell
Install-Module -Name Az -AllowClobber -Scope CurrentUser
```

Once that is installed, login to your PowerShell session with `Connect-AzAccount`.

To create an incremental snapshot with Azure PowerShell, set the configuration with [New-AzSnapShotConfig](/powershell/module/az.compute/new-azsnapshotconfig) with the `-Incremental` parameter and then pass that as a variable to [New-AzSnapshot](/powershell/module/az.compute/new-azsnapshot) through the `-Snapshot` parameter.

```PowerShell
$diskName = "yourDiskNameHere>"
$resourceGroupName = "yourResourceGroupNameHere"
$snapshotName = "yourDesiredSnapshotNameHere"

# Get the disk that you need to backup by creating an incremental snapshot
$yourDisk = Get-AzDisk -DiskName $diskName -ResourceGroupName $resourceGroupName

# Create an incremental snapshot by setting the SourceUri property with the value of the Id property of the disk
$snapshotConfig=New-AzSnapshotConfig -SourceUri $yourDisk.Id -Location $yourDisk.Location -CreateOption Copy -Incremental 
New-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName -Snapshot $snapshotConfig 
```

You can identify incremental snapshots from the same disk with the `SourceResourceId` and the `SourceUniqueId` properties of snapshots. `SourceResourceId` is the Azure Resource Manager resource ID of the parent disk. `SourceUniqueId` is the value inherited from the `UniqueId` property of the disk. If you were to delete a disk and then create a new disk with the same name, the value of the `UniqueId` property changes.

You can use `SourceResourceId` and `SourceUniqueId` to create a list of all snapshots associated with a particular disk. Replace `yourResourceGroupNameHere` with your value and then you can use the following example to list your existing incremental snapshots:

```PowerShell
$resourceGroupName = "yourResourceGroupNameHere"
$snapshots = Get-AzSnapshot -ResourceGroupName $resourceGroupName

$incrementalSnapshots = New-Object System.Collections.ArrayList
foreach ($snapshot in $snapshots)
{
    
    if($snapshot.Incremental -and $snapshot.CreationData.SourceResourceId -eq $yourDisk.Id -and $snapshot.CreationData.SourceUniqueId -eq $yourDisk.UniqueId){

        $incrementalSnapshots.Add($snapshot)
    }
}

$incrementalSnapshots
```

# [Portal](#tab/azure-portal)
[!INCLUDE [virtual-machines-disks-incremental-snapshots-portal](../../includes/virtual-machines-disks-incremental-snapshots-portal.md)]

# [Resource Manager Template](#tab/azure-resource-manager)

You can also use Azure Resource Manager templates to create an incremental snapshot. You'll need to make sure the apiVersion is set to **2019-03-01** and that the incremental property is also set to true. The following snippet is an example of how to create an incremental snapshot with Resource Manager templates:

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "diskName": {
      "type": "string",
      "defaultValue": "contosodisk1"
    },
  "diskResourceId": {
    "defaultValue": "<your_managed_disk_resource_ID>",
    "type": "String"
  }
  }, 
  "resources": [
  {
    "type": "Microsoft.Compute/snapshots",
    "name": "[concat( parameters('diskName'),'_snapshot1')]",
    "location": "[resourceGroup().location]",
    "apiVersion": "2019-03-01",
    "properties": {
      "creationData": {
        "createOption": "Copy",
        "sourceResourceId": "[parameters('diskResourceId')]"
      },
      "incremental": true
    }
  }
  ]
}
```
---

## Next steps

If you'd like to see sample code demonstrating the differential capability of incremental snapshots, using .NET, see [Copy Azure Managed Disks backups to another region with differential capability of incremental snapshots](https://github.com/Azure-Samples/managed-disks-dotnet-backup-with-incremental-snapshots).
