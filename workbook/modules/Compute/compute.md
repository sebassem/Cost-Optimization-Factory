# Compute cost optimization queries

## General

### Azure Advisor cost recommendations for Compute

#### Description

This query will display all Azure Advisor cost recommendations with Compute resources

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
advisorresources
| where type == "microsoft.advisor/recommendations"
| where tostring (properties.category) has "Cost"
| where properties.impactedField has "Compute" or properties.impactedField has "Container" or properties.impactedField has "Web"
| project AffectedResource=tostring(properties.resourceMetadata.resourceId),Impact=properties.impact,resourceGroup,AdditionaInfo=properties.extendedProperties,subscriptionId,Recommendation=tostring(properties.shortDescription.problem)
| where resourceGroup in ({ResourceGroup})
```

## Virtual Machines and Virtual Machine Scale Sets

### Stopped Virtual Machines

#### Description

This query will display all Azure Virtual Machines in a stopped state

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
resources
| where type =~ 'microsoft.compute/virtualmachines' and tostring(properties.extended.instanceView.powerState.displayStatus) != 'VM deallocated' and tostring(properties.extended.instanceView.powerState.displayStatus) != 'VM running'
| where resourceGroup in ({ResourceGroup})
| extend  VMname=name, PowerState=tostring(properties.extended.instanceView.powerState.displayStatus), VMLocation=location
| order by id asc
| project id,VMname, PowerState, VMLocation, resourceGroup, subscriptionId
| join kind = innerunique(
    resources
    | extend replaced_tags = replace('{}', 'null', tostring(tags))
    | extend replaced_tags = parse_json(replaced_tags)
    | mv-expand replaced_tags
    | extend tagName = tostring(bag_keys(replaced_tags)[0])
    | extend tagValue = tostring(replaced_tags['{TagName}'])
    | where tagName has '{TagName}' and tagValue has '{TagValue}'
    | distinct id
    )
    on id
```

### Virtual Machines and Virtual Machine Scale Sets with Azure Hybrid Benefit not enabled

#### Description

This query will list all Virtual Machines and Virtual Machine Scale Sets with Azure Hybrid Benefit not enabled

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
ResourceContainers | where type =~ 'Microsoft.Resources/subscriptions' | where tostring (properties.subscriptionPolicies.quotaId) !has "MSDNDevTest_2014-09-01"  | extend SubscriptionName=name
| join (
resources
| where resourceGroup in ({ResourceGroup})
| where type =~ 'microsoft.compute/virtualmachines' or type =~ 'microsoft.compute/virtualMachineScaleSets'
| where tostring(properties.storageProfile.osDisk.osType) == 'Windows' or tostring(properties.virtualMachineProfile.storageProfile.osDisk.osType) == 'Windows'
| where tostring(properties.['licenseType']) !has 'Windows' and tostring(properties.virtualMachineProfile.['licenseType']) !has 'Windows'
| extend WindowsId=id, VMName=name, VMLocation=location, VMRG=resourceGroup, OSType=tostring(properties.storageProfile.imageReference.offer), OsVersion = tostring(properties.storageProfile.imageReference.sku), VMSize=tostring (properties.hardwareProfile.vmSize), LicenseType = tostring(properties.['licenseType']), VMSSize=tostring(sku.name)
    ) on subscriptionId
| order by type asc
| project WindowsId,VMName,VMRG,VMSize, VMSSize, VMLocation,OSType, OsVersion,LicenseType, subscriptionId
```

### Virtual Machines and Virtual Machine Scale Sets with Azure Hybrid Benefit enabled

#### Description

This query will list all Virtual Machines and Virtual Machine Scale Sets with Azure Hybrid Benefit enabled

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
ResourceContainers | where type =~ 'Microsoft.Resources/subscriptions' | where tostring (properties.subscriptionPolicies.quotaId) !has "MSDNDevTest_2014-09-01"  | extend SubscriptionName=name
| join (
resources
| where resourceGroup in ({ResourceGroup})
| where type =~ 'microsoft.compute/virtualmachines'
| where tostring(properties.storageProfile.osDisk.osType) == 'Windows'
| where tostring(properties.['licenseType']) has "Windows"
| extend WindowsId=id, VMName=name, VMLocation=location, VMRG=resourceGroup, OSType=tostring(properties.storageProfile.imageReference.offer), OsVersion = tostring(properties.storageProfile.imageReference.sku), VMSize=tostring (properties.hardwareProfile.vmSize), LicenseType = tostring(properties.['licenseType']), VMSSize=tostring(sku.name)
) on subscriptionId
| order by type asc
| project WindowsId,VMName,VMRG,VMSize, VMSSize, VMLocation,OSType, OsVersion,LicenseType, subscriptionId
```

### Virtual Machine Scale Sets with Spot VMs

#### Description

This query will list all Virtual Machine Scale Sets and show if Spot VMs are being used and if Spot priority mix is enabled or not.

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
resources
| where type =~ 'microsoft.compute/virtualmachinescalesets'
| where resourceGroup in ({ResourceGroup})
| extend  SpotVMs=tostring(properties.virtualMachineProfile.priority), SpotPriorityMix=tostring(properties.priorityMixPolicy), SKU=tostring(sku.name), resourceGroup=strcat('/subscriptions/',subscriptionId,'/resourceGroups/',resourceGroup)
| project id, SKU, SpotVMs,SpotPriorityMix,subscriptionId,resourceGroup, location
| join kind = innerunique(
    resources
    | extend replaced_tags = replace('{}', 'null', tostring(tags))
    | extend replaced_tags = parse_json(replaced_tags)
    | mv-expand replaced_tags
    | extend tagName = tostring(bag_keys(replaced_tags)[0])
    | extend tagValue = tostring(replaced_tags['{TagName}'])
    | where tagName has '{TagName}' and tagValue has '{TagValue}'
    | distinct id
    )
    on id
```