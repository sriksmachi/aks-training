## Connect to Azure Subscription

az login --use-device-code

## Create AKS Cluster 

az group create --name az300-rg --location eastus

az ad sp create-for-rbac --skip-assignment

Sample Result

``
{
  "appId": "9d3093f2-6597-47cd-9337-9305e5c0c1aa",
  "displayName": "azure-cli-2019-03-13-09-03-25",
  "name": "http://azure-cli-2019-03-13-09-03-25",
  "password": "7f432cc5-6621-4977-b8a1-fbb88f047280",
  "tenant": "c43c712f-b296-44c0-a286-00afd8bf9dfb"
}

``

az aks create --resource-group az300-rg --name az300-cluster --node-count 1 --enable-addons monitoring --generate-ssh-keys --enable-rbac --enable-addons http_application_routing --service-principal 9d3093f2-6597-47cd-9337-9305e5c0c1aa --client-secret 7f432cc5-6621-4977-b8a1-fbb88f047280

## Installing Kubectl

az aks install-cli

## Connect with Kubectl

az aks get-credentials --resource-group az300-rg --name az300-cluster

## Enabling Kubernetes Dashboard

kubectl create clusterrolebinding kubernetes-dashboard -n kube-system --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard

## Browse Kubernetes Dashboard

az aks browse --resource-group az300-rg --name az300-cluster

## Create ACR

az acr create --resource-group az300-rg --name az300-acr --sku Basic
az acr list --resource-group az300-rg --query "[].{acrLoginServer:loginServer}" --output table

Sample Response

``
AcrLoginServer
-------------------
az300acr.azurecr.io

``

## Granting AKS to pull images from AKS Cluster

az ad sp create-for-rbac --skip-assignment
az group create --name az300-rg-prod --location eastus

az aks create --resource-group az300-rg --name az300-cluster --node-count 1 --enable-addons monitoring --generate-ssh-keys --enable-rbac --enable-addons http_application_routing

az acr show --resource-group myResourceGroup --name <acrName> --query "id" --output tsv