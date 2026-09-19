# Declarative Azure Windows VM Provisioning via ARM Templates

![Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![ARM](https://img.shields.io/badge/IaC-ARM%20Templates-blue?style=for-the-badge&logo=azuredevops&logoColor=white)
![Verification](https://img.shields.io/badge/Deployment-Verified-success?style=for-the-badge)

## 1. Business Problem & Architecture Overview

Ad-hoc, CLI-driven virtual machine provisioning without codified boundaries frequently leads to configuration drift, unmanaged resource lifecycles, and security exposure (e.g., blanket `0.0.0.0/0` access on port 3389).

This project demonstrates the transition from procedural Azure CLI deployment to declarative Infrastructure-as-Code (IaC) using **Azure Resource Manager (ARM)** templates.

[Internet]
│
▼
[Network Security Group (Port 3389 Restricted)]
│
[Public IP] ───► [Network Interface] ───► [Windows Server 2019 VM]
│
[VNet / Subnet]


### Resource Inventory
| Resource Type | Resource Name | Purpose | Configuration Detail |
| :--- | :--- | :--- | :--- |
| `Microsoft.Compute/virtualMachines` | `myvm-vm` | Compute Workload | Windows Server 2019 Datacenter Gen2 (`Standard_B1s`) |
| `Microsoft.Network/virtualNetworks` | `myvm-vnet` | Network Isolation | CIDR `10.0.0.0/16`, Subnet `10.0.0.0/24` |
| `Microsoft.Network/networkSecurityGroups` | `myvm-nsg` | Perimeter Defense | Inbound TCP 3389 restricted to authorized IP |
| `Microsoft.Network/publicIPAddresses` | `myvm-pip` | Inbound Management | Static allocation, Standard SKU |

---

## 2. Codified Infrastructure (ARM JSON)

The template enforces clean resource lifecycle governance using `deleteOption: "Delete"` on both the OS disk and the NIC, ensuring unattached resources do not linger after VM deletion:

```json
"osDisk": {
  "createOption": "FromImage",
  "managedDisk": {
    "storageAccountType": "Premium_LRS"
  },
  "deleteOption": "Delete"
}
Deployment Command
Bash
az deployment group create \
  --resource-group rg-core-compute-prod-01 \
  --template-file templates/azuredeploy.json \
  --parameters adminUsername="demouser" \
               adminPassword="<SecurePassword>" \
               allowedRdpSourceIp="<Your-Client-IP>/32"
3. Implementation & Verification Proof
Verification 1: CLI Provisioning Execution
Initial deployment executed via Azure CLI passing secure environment variables:

Verification 2: Running State & Attached Interfaces
Azure Portal confirms running state, network bindings, and private IP assignment:

4. Key Engineering Takeaways & Real-World Framing
Preventing Orphan Resource Waste: In standard deployments, deleting a VM leaves the managed OS disk and NIC intact, incurring unnecessary cost. Setting deleteOption: "Delete" guarantees clean resource de-allocation.

Perimeter Ingress Hardening: Default Azure CLI commands generate open NSG rules (* source). Using an explicit source CIDR parameter on NSG rule priority 1000 ensures least-privilege administrative access.

Idempotency: Transitioning from procedural CLI scripts to declarative ARM JSON guarantees repeatable deployments without state collisions or configuration drift.
