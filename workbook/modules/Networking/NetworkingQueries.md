# Networking cost optimization queries

## Azure Application Gateways

### Idle Application Gateways

#### Description

This query will get the Application Gateways that have empty backend pools

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
resources
| where type =~ 'Microsoft.Network/applicationGateways' and resourceGroup in ({ResourceGroup})
| extend backendPoolsCount = array_length(properties.backendAddressPools),SKUName= tostring(properties.sku.name), SKUTier= tostring(properties.sku.tier),SKUCapacity=properties.sku.capacity,backendPools=properties.backendAddressPools
| project id, name, SKUName, SKUTier, SKUCapacity
| join (
    resources
    | where type =~ 'Microsoft.Network/applicationGateways' and resourceGroup in ({ResourceGroup})
    | mvexpand backendPools = properties.backendAddressPools
    | extend backendIPCount = array_length(backendPools.properties.backendIPConfigurations)
    | extend backendAddressesCount = array_length(backendPools.properties.backendAddresses)
    | extend backendPoolName  = backendPools.properties.backendAddressPools.name
    | summarize backendIPCount = sum(backendIPCount) ,backendAddressesCount=sum(backendAddressesCount) by id
) on id
| project-away id1
| where  (backendIPCount == 0 or isempty(backendIPCount)) and (backendAddressesCount==0 or isempty(backendAddressesCount))
| order by id asc
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

## Azure Load Balancers

### Idle Load Balancers

#### Description

This query will get the Load Balancers that have empty backend pools

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
resources
| where resourceGroup in ({ResourceGroup})
| where type =~ 'microsoft.network/loadbalancers' and tostring(properties.backendAddressPools) == '[]' 
| extend LBRG=resourceGroup, LoadBalancerName=name, SKU=sku, LBLocation=location
| order by id asc
| project id,LoadBalancerName, SKU.name,SKU.tier, LBLocation,LBRG, subscriptionId
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

## Public Ip addresses

### Idle Public IP addresses

#### Description

This query will get the Public IP addresses that are not attached to any resource or attached to the Network Card that is not attached to any resource

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
resources
| where resourceGroup in ({ResourceGroup})
| where type =~ 'Microsoft.Network/publicIPAddresses' and isempty(properties.ipConfiguration) and isempty(properties.natGateway) 
| extend PublicIpId=id, IPName=name, AllocationMethod=tostring(properties.publicIPAllocationMethod), SKUName=sku.name, Location=location
| project PublicIpId,IPName, SKUName, resourceGroup, Location, AllocationMethod, subscriptionId
| union (
 Resources
 | where type =~ 'microsoft.network/networkinterfaces' and isempty(properties.virtualMachine) and isnull(properties.privateEndpoint) and isnotempty(properties.ipConfigurations)
 | extend IPconfig = properties.ipConfigurations
 | mv-expand IPconfig
 | extend PublicIpId= tostring(IPconfig.properties.publicIPAddress.id)
 | project PublicIpId
 | join (
    resources
    | where type =~ 'Microsoft.Network/publicIPAddresses'
    | extend PublicIpId=id, IPName=name, AllocationMethod=tostring(properties.publicIPAllocationMethod), SKUName=sku.name, resourceGroup, Location=location
 ) on PublicIpId
 | project PublicIpId,IPName, SKUName, resourceGroup, Location, AllocationMethod, subscriptionId
)
| join kind = innerunique(
    resources
    | extend replaced_tags = replace('{}', 'null', tostring(tags))
    | extend replaced_tags = parse_json(replaced_tags)
    | mv-expand replaced_tags
    | extend tagName = tostring(bag_keys(replaced_tags)[0])
    | extend tagValue = tostring(replaced_tags['{TagName}'])
    | where tagName has '{TagName}' and tagValue has '{TagValue}'
    | extend PublicIpId=id
    | distinct PublicIpId
    )
    on PublicIpId
```

## Virtual Network Gateways

### Idle Virtual Network Gateways

#### Description

This query will get the Virtual Network Gateways that have no connections defined

#### Query type

Azure Resource Graph

#### Azure Resource Graph query

```python
resources
| where type == "microsoft.network/virtualnetworkgateways"
| project id
| join kind = leftouter(
    resources
    | where type == "microsoft.network/connections"
    | extend id = tostring(properties.virtualNetworkGateway1.id)
    | project id
    | union (
         resources
         | where type == "microsoft.network/connections"
         | extend id = tostring(properties.virtualNetworkGateway2.id)
         | project id
    )
) on id
| where isempty(id1)
| project id
```
