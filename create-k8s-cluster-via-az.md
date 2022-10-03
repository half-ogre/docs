# Create an Azure Kubernetes Cluster Via the Azure CLI

This doc shows how to create an Azure Kubernetes Service cluster, step by step, using the Azure CLI (`az`). This should properly be done with some sort of IaC normally, but doing this imperatively in the shell is helpful when experimenting or learning.

_Adapted from:_

 - _https://docs.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-cli_
 - _https://docs.microsoft.com/en-us/azure/application-gateway/quick-create-cli_

## Set up variables
To make this easier to copy and paste, we'll create some local shell variables. We'll need to have the `jq` command in the `$PATH`.

```
VNET_BASE=10.33
K8S_CLUSTER_NAME=sample
K8S_CLUSTER_LOCATION=eastus
```

## Create a resource group
First we'll need a resource group all the various moving parts will use.

```
 az group create --name $K8S_CLUSTER_NAME-rg --location $K8S_CLUSTER_LOCATION
```

## Create a managed identity for the cluster
When using our own subnet for the cluster, the guidance is to use a separate managed identity rather than letting Azure Kubernetes Service create and assign its own. Therefore, we'll need to create a managed identity and stash its resource ID.

```
az identity create -g $K8S_CLUSTER_NAME-rg -n $K8S_CLUSTER_NAME-identity -l K8S_CLUSTER_LOCATION

K8S_IDENTITY_RESOURCE_ID=$(az identity list -g $K8S_CLUSTER_NAME-rg | jq -r --arg name "$K8S_CLUSTER_NAME-identity" '.[] | select(.name==$name) | .id')
```

## Create a virtual network

```
az network vnet create \
  --name $K8S_CLUSTER_NAME-vnet \
  --resource-group $K8S_CLUSTER_NAME-rg \
  --location $K8S_CLUSTER_LOCATION \
  --address-prefix $VNET_BASE.0.0/16 \
  --subnet-name $K8S_CLUSTER_NAME-main-subnet \
  --subnet-prefix $VNET_BASE.0.0/24

az network vnet subnet create \
  --name $K8S_CLUSTER_NAME-backend-subnet \
  --resource-group $K8S_CLUSTER_NAME-rg \
  --vnet-name $K8S_CLUSTER_NAME-vnet \
  --address-prefix $VNET_BASE.1.0/24

VNET_BACKEND_SUBNET_ID=$(az network vnet list | jq -r --arg name "$K8S_CLUSTER_NAME-backend-subnet" '.[].subnets[] | select(.name==$name) | .id')
``` 

## Create the Kubernetes cluster
Next we'll create the Kubernetes cluster itself. We'll add the monitoring add-on (`--enable-addons monitoring`, see https://docs.microsoft.com/en-us/azure/aks/monitor-aks), and let Azure manage the cluster's identity (`--enable-managed-identity`, see https://docs.microsoft.com/en-us/azure/aks/use-managed-identity).

```
az aks create -g $K8S_CLUSTER_NAME-rg -n $K8S_CLUSTER_NAME-k8s --assign-identity "$K8S_IDENTITY_RESOURCE_ID" --node-count 1 --enable-addons monitoring --vnet-subnet-id "$VNET_BACKEND_SUBNET_ID"
```

## Create an application gateway
_Adapted from https://docs.microsoft.com/en-us/azure/application-gateway/quick-create-cli._

```
az network vnet create \
  --name half-ogre-com-vnet \
  --resource-group half-ogre-com-rg \
  --location eastus \
  --address-prefix 10.33.0.0/16 \
  --subnet-name half-ogre-com-subnet \
  --subnet-prefix 10.33.0.0/24

az network vnet subnet create \
  --name half-ogre-com-backend-subnet \
  --resource-group half-ogre-com-rg \
  --vnet-name half-ogre-com-vnet \
  --address-prefix 10.33.1.0/24

az network public-ip create \
  --resource-group half-ogre-com-rg \
  --name half-ogre-com-appgw-ip \
  --allocation-method Static \
  --sku Standard
```


---

## Create a service principal for the Kubernetes cluster

```
AZ_SUB_ID=$(az account show | jq -r .id)

az ad sp create-for-rbac --name $K8S_CLUSTER_NAME-sp --role Contributor --scopes /subscriptions/$AZ_SUB_ID

 SP_ID=$(az ad sp list --filter "displayname eq '$K8S_CLUSTER_NAME-sp'" | jq -r --arg name "$K8S_CLUSTER_NAME-sp" '.[] | select(.displayName==$name) | .id')
```