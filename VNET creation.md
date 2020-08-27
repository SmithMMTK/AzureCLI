# Create VNET

```bash
az group create -n azvnet -l southeastasia


# Create 3 VNet (Hub and Spoke vnet1, vnet2, vnet 3)
az network vnet create -g azvnet -n vnet1 --address-prefix 10.1.0.0/16 \
--subnet-name subnet1 --subnet-prefix 10.1.0.0/24
                            
az network vnet create -g azvnet -n vnet2 --address-prefix 10.2.0.0/16 \
--subnet-name subnet2 --subnet-prefix 10.2.0.0/24         

az network vnet create -g azvnet -n vnet3 --address-prefix 10.3.0.0/16 \
--subnet-name subnet3 --subnet-prefix 10.3.0.0/24
                            
az vm create \
    --resource-group azvnet \
    --name vm1  \
    --image UbuntuLTS \
    --admin-username azureuser \
    --vnet-name vnet1 --subnet subnet1 \
    --generate-ssh-keys 
    
az vm create \
    --resource-group azvnet \
    --name vm2  \
    --image UbuntuLTS \
    --admin-username azureuser \
    --vnet-name vnet2 --subnet subnet2 \
    --generate-ssh-keys

az vm create \
    --resource-group azvnet \
    --name vm3  \
    --image UbuntuLTS \
    --admin-username azureuser \
    --vnet-name vnet3 --subnet subnet3 \
    --generate-ssh-keys


# Create Gateway on Hub VNET (vnet3)
az network vnet subnet create --vnet-name vnet3 \
-n GatewaySubnet -g azvnet --address-prefix 10.3.1.0/27

# Create Public IP to use on VPN Gateway
az network public-ip create -n vnet3pubip -g azvnet --allocation-method Dynamic

# Create VPN Gateway
avz network vnet-gateway create -n vnet3gw -l southeastasia \
--public-ip-address vnet3pubip -g azvnet --vnet vnet3 \
--gateway-type Vpn --sku VpnGw1 --vpn-type RouteBased --no-wait

# Create Peering between VNet3, VNet2, VNet 1
az network vnet peering create -g azvnet -n vnet3to1 \
--vnet-name vnet3 --remote-vnet vnet1 --allow-vnet-access

az network vnet peering create -g azvnet -n vnet1to3 \
--vnet-name vnet1 --remote-vnet vnet3 --allow-vnet-access

az network vnet peering create -g azvnet -n vnet3to2 \
--vnet-name vnet3 --remote-vnet vnet2 --allow-vnet-access

az network vnet peering create -g azvnet -n vnet2to3 \
--vnet-name vnet2 --remote-vnet vnet3 --allow-vnet-access

# Create Allow Forward Traffic

az network vnet peering update -g azvnet -n vnet3to1 --vnet-name vnet3 --set allowForwardedTraffic=true
az network vnet peering update -g azvnet -n vnet3to2 --vnet-name vnet3 --set allowForwardedTraffic=true
az network vnet peering update -g azvnet -n vnet1to3 --vnet-name vnet1 --set allowForwardedTraffic=true
az network vnet peering update -g azvnet -n vnet2to3 --vnet-name vnet2 --set allowForwardedTraffic=true

# Create effective setting
az network vnet peering list --resource-group azvnet --vnet-name vnet3 -o table
az network vnet peering list --resource-group azvnet --vnet-name vnet2 -o table
az network vnet peering list --resource-group azvnet --vnet-name vnet1 -o table


# Create Allow Gateway Transit
az network vnet peering update -g azvnet -n vnet3to1 --vnet-name vnet3 --set allowGatewayTransit=true
az network vnet peering update -g azvnet -n vnet3to2 --vnet-name vnet3 --set allowGatewayTransit=true

# Check creating status
az network vnet-gateway list --resource-group azvnet -o table

# Create Use Remote Gateway
az network vnet peering update -g azvnet -n vnet1to3 --vnet-name vnet1 --set useRemoteGateways=true
az network vnet peering update -g azvnet -n vnet2to3 --vnet-name vnet2 --set useRemoteGateways=true

# Create Routing Table
az network route-table create -n myRoute -g azvnet

# Create Routing Rule
az network route-table route create -g azvnet -n myRoute \
--route-table-name myRoute -n vnet1rt --next-hop-type VirtualNetworkGateway --address-prefix 10.1.0.0/16

az network route-table route create -g azvnet -n myRoute \
--route-table-name myRoute -n vnet2rt --next-hop-type VirtualNetworkGateway --address-prefix 10.2.0.0/16

# Add Routing Table to Subnet
az network vnet subnet update -g azvnet -n subnet1 --vnet-name vnet1 --route-table myRoute
az network vnet subnet update -g azvnet -n subnet2 --vnet-name vnet2 --route-table myRoute

# Delete resource
az group delete -g azvnet --no-wait -y

```
