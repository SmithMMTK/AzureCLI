# Create VM with VNET by CLI

### Generate ssh key pair
```bash
ssh-keygen -m PEM -t rsa -b 4096

## Public / Private default directory is ~/.ssh
cd ~/.ssh

## You should see list of Public / Private key pair
ls
```

---
### Create Infrastucture
```bash
export rg="RGname"
export vnet="RGvnet"
export addressprefixes="10.0.0.0/16"
export subnetname="workload"
export subnetprefixes="10.0.0.0/24"

export subnetname2="backend"
export subnetprefixes2="10.0.1.0/24"

## Create Azure Resource Group
az group create -n $rg -l southeastasia

## Create Virtual Network
az network vnet create \
    --address-prefixes $addressprefixes \
    --name $vnet \
    --resource-group $rg \
    --subnet-name $subnetname \
    --subnet-prefixes $subnetprefixes

## Create Additional Subnet
az network vnet subnet create \
    -g $rg \
    --vnet-name $vnet \
    -n $subnetname2 \
    --address-prefixes $subnetprefixes2

## Create Network Security Group
az network nsg create \
    -g $rg \
    -n $subnetname
az network nsg create \
    -g $rg \
    -n $subnetname2

## Create NSG Rule to allow SSH
az network nsg rule create \
    -g $rg \
    --nsg-name $subnetname \
    -n "Allow SSH" \
    --priority 400 \
    --source-address-prefixes Internet \
    --destination-address-prefixes VirtualNetwork \
    --destination-port-ranges 22 \
    --direction Inbound \
    --access Allow \
    --protocol Tcp \
    --description "Allow SSH from Internet."

## Assign NSG to Subnet
az network vnet subnet update \
-g $rg \
-n $subnetname \
--vnet-name $vnet \
--network-security-group $subnetname
```
---

### Genereate VM with SSH Key
```bash

az vm create \
  --resource-group $rg \
  --name myVM \
  --image RedHat:RHEL:7.6:latest \
  --admin-username azureuser \
  --ssh-key-values @~/.ssh/rhelkey.pub \
  --size Standard_A2_v2 \
  --vnet-name $vnet --subnet $subnetname \
  --no-wait

ssh -i rhelvm azureuser@public_ip_address
az group delete -n $rg --no-wait
az group list -o table
```

### Alternative : Generate VM by Generate SSH Key
```bash
az vm create \
  --resource-group RGname \
  --name myVM \
  --image RedHat:RHEL:7.6:latest \
  --admin-username azureuser \
  --generate-ssh-keys
```

---
