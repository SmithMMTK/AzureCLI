# Create Linux VM Basic

```bash
export linuxvm=az-vm-fluentd

az group create --name $linuxvm --location southeastasia


az vm create \
    --resource-group $linuxvm \
    --name $linuxvm  \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    ## --ssh-key-values @~/.ssh/mykey.pub \
```