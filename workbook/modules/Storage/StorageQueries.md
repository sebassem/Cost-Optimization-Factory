# Storage cost optimization queries

## General

### Azure Advisor cost recommendations for Storage

#### Description

This query will display all Azure Advisor cost recommendations with Storage resources

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
advisorresources
| where type == "microsoft.advisor/recommendations"
| where tostring (properties.category) has "Cost"
| where properties.impactedField has "Storage"
| project AffectedResource=tostring(properties.resourceMetadata.resourceId),Impact=properties.impact,resourceGroup,AdditionaInfo=properties.extendedProperties,subscriptionId,Recommendation=tostring(properties.shortDescription.problem),name
| where resourceGroup in ({ResourceGroup})
```

## Storage Accounts

### Storage Accounts that are not V2

#### Description

This query will list all Storage Accounts that are not of type V2

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
resources
| where type =~ 'Microsoft.Storage/StorageAccounts' and kind !='StorageV2' and kind !='FileStorage'
| where resourceGroup in ({ResourceGroup})
| extend StorageAccountName=name, SAKind=kind,AccessTier=tostring(properties.accessTier),SKUName=sku.name, SKUTier=sku.tier, Location=location
| order by id asc
| project id,StorageAccountName, SKUName, SKUTier, SAKind,AccessTier, resourceGroup, Location, subscriptionId
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

## Managed Disks

### Unattached Managed Disks

#### Description

This query will list all Managed Disks that are not attached to any resource

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
resources
| where type =~ 'microsoft.compute/disks' and managedBy == ""
| where resourceGroup in ({ResourceGroup})
| extend diskState = tostring(properties.diskState)
| where managedBy == "" and diskState != 'ActiveSAS'
or diskState == 'Unattached' and diskState != 'ActiveSAS'
| extend DiskId=id, DiskIDfull=id, DiskName=name, SKUName=sku.name, SKUTier=sku.tier, DiskSizeGB=tostring(properties.diskSizeGB), Location=location, TimeCreated=tostring(properties.timeCreated), QuickFix=id
| order by DiskId asc
| project DiskId, DiskIDfull, DiskName, DiskSizeGB, SKUName, SKUTier, resourceGroup, QuickFix, Location, TimeCreated, subscriptionId
```

### Old Azure Disks snapshots (> 30 days)

#### Description

This query will list all snaphshots that are older than 30 days

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
resources
| where type == 'microsoft.compute/snapshots'
| where resourceGroup in ({ResourceGroup})
| extend TimeCreated = tostring(properties.timeCreated)
| where TimeCreated < ago(30d)
| order by id asc
| project id, resourceGroup, location, TimeCreated ,subscriptionId
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
