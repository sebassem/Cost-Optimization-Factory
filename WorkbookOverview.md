# Workbook overview
In this section, you will find an overview of all of the queries already covered by the Workbook. 

- [Compute](#compute)
- [Azure Hybrid Benefit](#azure-hybrid-benefit)
- [Storage](#storage)
- [Networking](#networking)

## compute

### **Advisor Recommendations**
Review the advisor recommendations for Compute. Some of the recommendations available in this tile could be "Optimize virtual machine spend by resizing or shutting down underutilized instances" or "Buy reserved virtual machine instances to save money over pay-as-you-go costs."

#### **Web Apps**

Review the App Service list. Advise the customer to review the Stopped App Services, as they will be charged.
Advise customer to upgrade V2 Sku to V3 SKU. V3 SKU is cheaper than similar V2 SKU and allows [Reserved Instances and Savings plan for compute](https://azure.microsoft.com/en-us/pricing/details/app-service/windows/ "Reserved Instances and Savings plan for compute").


#### **Azure Kubernetes Clusters (AKS)**

Review the AKS list. Some of the discussion opportunities are:

* Enable cluster autoscaler to automatically adjust the number of agent nodes in response to resource constraints
* Consider using Azure Spot VMs for workloads that can handle interruptions, early terminations, or evictions. For example, workloads such as batch processing jobs, development and testing environments, and large compute workloads may be good candidates for scheduling on a spot node pool.
* Utilize the Horizontal pod autoscaler to adjust the number of pods in a deployment depending on CPU utilization or other select metrics.
* Use the Start/Stop feature in Azure Kubernetes Services (AKS).
* Use appropriate VM SKU per node pool and reserved instances where long-term capacity is expected.


## Azure Hybrid Benefit

### **Windows VMs and VMSS not using Hybrid Benefit**

Azure Hybrid Benefit represents an excellent opportunity to save on Virtual Machines OS costs.
Ask your customer if they have Software Assurance. If so, your customer can enable [Hybrid Benefit](https://docs.microsoft.com/en-us/windows-server/get-started/azure-hybrid-benefit)
Discuss potential savings using [Azure Hybrid Benefit Calculator](https://azure.microsoft.com/en-gb/pricing/hybrid-benefit/#calculator "Azure Hybrid Benefit Calculator")

> NOTE
> If the customer has selected Dev/Test subscription(s) within this assessment then they should already have discounts on Windows licenses so recommendations here don't apply to this subscription(s)

### **Linux VM not using Hybrid Benefit**

Azure Hybrid Benefit represents an excellent opportunity to save on Virtual Machines OS costs. [Azure Hybrid Benefit for Linux](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/azure-hybrid-benefit-linux) is a licensing benefit that helps you to significantly reduce the costs of running your Red Hat Enterprise Linux (RHEL) and SUSE Linux Enterprise Server (SLES) virtual machines (VMs) in the cloud.

### **SQL HUB Licenses**

Azure Hybrid Benefit represents an excellent opportunity to save costs on SQL instances.
Ask your customer if they have Software Assurance. If so, your customer can enable [SQL Hybrid Benefit](https://docs.microsoft.com/en-us/azure/azure-sql/azure-hybrid-benefit?tabs=azure-powershell)
Discuss potential savings using [Azure Hybrid Benefit Calculator ](https://azure.microsoft.com/en-gb/pricing/hybrid-benefit/#calculator "Azure Hybrid Benefit Calculator ")

## Storage

### **Advisor Recommendations**

Review the advisor recommendations for Storage. Some of the recommendations available in this tile could be "Blob storage reserved capacity" or "Use lifecycle management."

### **Unattached Managed Disks**

<br>Review the list of managed unattached disks with the customer, explain to the customer that unattached disks represent a cost in the subscription. Ensure that the customer understands that they need to ensure the disks are not needed before deleting the data.

> NOTE
>This query has a **Quick Fix** colunmn that help the customer to remove the disk if not needed.

### **Disk snapshots older than 30 days**

This query will show snapshots that are older than 30 days. Explain to the customer the costs of keeping a snapshot and delete them if not needed anymore.


### **Idle Disk Snapshots**

This query will show snapshots which the source disk doesn't exist anymore. Explain to the customer the costs storing an idle snapshot and delete them if not needed anymore.


### Networking

### **Advisor Recommendations**

Review the advisor recommendations for Networking. Some of the recommendations available in this tile could be "Reduce costs by deleting or reconfiguring idle virtual network gateways" or "Reduce costs by eliminating unprovisioned ExpressRoute circuits."

### **Application Gateway with empty backendpool**

<br> Review the Application Gateways with empty backend pools, discuss with the customer if they can be deleted.
App gateways are considered idle if there isn't any backend pool with targets.

Explain to the customer that this resource represents a cost.

### **Load Balancer with empty backendpool**

<br> Review the Load Balancers with empty backend pool, discuss with the customer if they can be deleted.

Explain to the customer that this resource represents a cost.

### **Unattached Public IPs**

<br> Review the idle Public Ip Addresses (this query will also show Public IP addresses attached to idle NICs), discuss with the customer if they can be deleted.

Explain to the customer that public IP addresses represent a cost.

### **Idle Virtual Network Gateways**
eview the Idle Virtual Network Gateways, discuss with the customer if they can be deleted.

Explain to the customer that VPN gateways represent a cost.