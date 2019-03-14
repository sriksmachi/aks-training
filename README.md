## Connect to Azure Subscription

az login --use-device-code

## Create AKS Cluster 

az group create --name az300-rg --location eastus

az ad sp create-for-rbac --skip-assignment

Sample Result

```
{
  "appId": "9d3093f2-6597-47cd-9337-9305e5c0c1aa",
  "displayName": "azure-cli-2019-03-13-09-03-25",
  "name": "http://azure-cli-2019-03-13-09-03-25",
  "password": "7f432cc5-6621-4977-b8a1-fbb88f047280",
  "tenant": "c43c712f-b296-44c0-a286-00afd8bf9dfb"
}

```

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

```
AcrLoginServer
-------------------
az300acr.azurecr.io

```
```
docker tag redis az300acr.azurecr.io/redis
docker tag microsoft/azure-vote-front:v1 az300acr.azurecr.io/azure-vote-front:v1

docker push az300acr.azurecr.io/redis
docker push az300acr.azurecr.io/azure-vote-front:v1
```


## Granting AKS to pull images from AKS Cluster

az acr show --resource-group myResourceGroup --name <acrName> --query "id" --output tsv
az role assignment create --assignee 9d3093f2-6597-47cd-9337-9305e5c0c1aa --scope /subscriptions/be64c606-f37c-4678-ae92-7833953c6cc0/resourceGroups/az300-rg/providers/Microsoft.ContainerRegistry/registries/az300acr --role acrpull

Sample Response

```
{
  "canDelegate": null,
  "id": "/subscriptions/be64c606-f37c-4678-ae92-7833953c6cc0/resourceGroups/az300-rg/providers/Microsoft.ContainerRegistry/registries/az300acr/providers/Microsoft.Authorization/roleAssignments/ebe34f0d-dffa-4c19-8577-3ca163fb6798",
  "name": "ebe34f0d-dffa-4c19-8577-3ca163fb6798",
  "principalId": "490804c4-4e7c-47b5-aee9-0da4770b41cf",
  "resourceGroup": "az300-rg",
  "roleDefinitionId": "/subscriptions/be64c606-f37c-4678-ae92-7833953c6cc0/providers/Microsoft.Authorization/roleDefinitions/7f951dda-4ed3-4680-a7ca-43fe172d538d",
  "scope": "/subscriptions/be64c606-f37c-4678-ae92-7833953c6cc0/resourceGroups/az300-rg/providers/Microsoft.ContainerRegistry/registries/az300acr",
  "type": "Microsoft.Authorization/roleAssignments"
}
```

## Infra Ops
az aks get-upgrades --resource-group az300-rg --name az300-cluster --output table
az aks upgrade --resource-group az300-rg --name az300 --kubernetes-version 1.10.9

## Basic Deployment on Ops for K8s Application
az aks get-credentials --resource-group az300-rg --name az300-cluster
kubectl apply -f azure-vote.yaml
kubectl get service azure-vote-front --watch
kubectl scale --replicas=5 deployment/azure-vote-front
kubectl set image deployment azure-vote-front azure-vote-front=<acrLoginServer>/azure-vote-front:v2
