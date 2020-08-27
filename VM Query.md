# List VM with Query
```bash
az vm list -d --output table --query '[].{name:name,powerState:powerState,location:location,resourceGroup:resourceGroup,publicIP:publicIps, hardwareProfile:vmSize}'

az network vnet list -d -o table


az network vnet show --ids $(az network vnet list --query "[].id" -o tsv)


az network vnet list --query "[].id"-o table


az vm list-vm-resize-options --ids $(az vm list -g MyResourceGroup --query "[].id" -o tsv)

az vm list-vm-resize-options --ids $(az vm list -d --query "[].id" -o tsv)

az vm nic list -g MyResourceGroup --vm-name MyVm


az account set --subscription azSmithM-MSFT

az account set --subscription IVL-SAP-WUS-Production

az account set --subscription IVL-SAP-WUS-Hub

az account set --subscription IVL-SAP-WEU-Hub

az account set --subscription IVL-SAP-SEA-Hub

az account set --subscription IVL-SAP-EUS-DR

az vm list -d --query "[].{name:name, resourceGroup:resourceGroup, location:location, VMSize:hardwareProfile.vmSize, publisher:storageProfile.imageReference.publisher,imageReference:storageProfile.imageReference.offer, winsku:storageProfile.osDisk.sku, licenseType:licenseType}" -o table 

az vm list -d --query "[].{name:name, resourceGroup:resourceGroup, tags:tags.project}" -o table
az vm list -d --query "[].{name:name, resourceGroup:resourceGroup, tags:tags.Environment}" -o table
az tag list -o table
```

