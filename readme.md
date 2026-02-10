Overview:

This project documents how DataHub was deployed on Azure Kubernetes Service (AKS) using the official DataHub Helm chart.
The goal was to get DataHub running and accessible online using Kubernetes and Azure-managed networking components.

What was deployed:

Azure Kubernetes Service (AKS)
DataHub platform (Frontend, GMS, etc.)
Azure Application Gateway as ingress
Public IP for external access
Kubernetes Ingress for routing traffic

Why Helm was used:

DataHub has many services and dependencies.
Instead of writing and managing many Kubernetes YAML files manually, the official Helm chart was used.

Helm makes the deployment:

Easier to manage
Reproducible
Aligned with official DataHub recommendations

Official Repository Used:

The deployment uses the official DataHub Helm repository:
https://github.com/acryldata/datahub-helm

The chart used is:
 charts/datahub
The upstream chart was not modified or forked.

Configuration (values.yaml)
All custom configuration was done using a values.yaml file.
Helm uses this file to generate Kubernetes manifests automatically, which is why no raw manifest files are included in this repository.

Accessing DataHub
DataHub is exposed using:
Azure Application Gateway
A public IP address
Kubernetes Ingress

Once deployed, DataHub can be accessed through the public endpoint provided by Azure.

What is not included
No custom Kubernetes manifests
No governance automation
No custom DataHub code
This setup focuses only on deployment and access.

Notes
This repository contains only deployment configuration and documentation.
The actual application code is maintained in the official DataHub repository.




******************************************************************************

the exact steps followed to deploy DataHub on Azure Kubernetes Service (AKS) using Helm.
All steps are shown in the order they were executed.

Step 1: Azure Login
Logged into Azure using Azure CLI.

az login
Verified the correct subscription.

az account show

Step 2: Create Resource Group

Created a resource group to hold all Azure resources.

az group create \
  --name datahub-rg \
  --location eastus

Step 3: Create AKS Cluster

Created an AKS cluster with default node pool.

az aks create \
  --resource-group datahub-rg \
  --name datahub-aks \
  --node-count 2 \
  --enable-managed-identity \
  --generate-ssh-keys

Step 4: Connect to AKS Cluster

Fetched Kubernetes credentials to access the cluster.

az aks get-credentials \
  --resource-group datahub-rg \
  --name datahub-aks

Verified cluster access.

kubectl get nodes

Step 5: Create Namespace for DataHub

Created a dedicated namespace for DataHub.

kubectl create namespace datahub

Step 6: Install Helm
Installed Helm CLI (if not already installed).

helm version

Step 7: Add Official DataHub Helm Repository
Added the official DataHub Helm repository.

helm repo add datahub https://helm.datahubproject.io
helm repo update

Step 8: Prepare values.yaml Configuration

Created a values.yaml file to customize the deployment.
This file contains:
Ingress configuration
Resource settings
Service exposure
No changes were made to the upstream Helm chart.

Step 9: Deploy DataHub Using Helm

Installed DataHub using Helm and the custom values file.

helm upgrade --install datahub datahub/datahub \
  --namespace datahub \
  --create-namespace \
  -f values.yaml

Step 10: Verify Pods and Services
Checked that all DataHub pods are running.

kubectl get pods -n datahub
Checked services.

kubectl get svc -n datahub

Step 11: Configure Ingress and Load Balancer
Configured Kubernetes Ingress to expose DataHub.
Application Gateway Ingress Controller (AGIC) syncs Ingress rules to Azure Application Gateway.
Verified ingress.

kubectl get ingress -n datahub
Step 12: Get Public IP Address
Fetched the public IP assigned to the Application Gateway / Load Balancer.

kubectl get ingress -n datahub
or

kubectl get svc -n datahub

Step 13: Access DataHub UI

Opened the DataHub UI in a browser using the public IP address.
Confirmed:
UI loads successfully
Application is reachable
Step 14: Post-Deployment Verification
Verified:
Pods are stable
No crash loops
UI is accessible
No governance or metadata automation was implemented at this stage.
DataHub Helm Charts
 
Notes
Azure Application Gateway Ingress Controller (AGIC) is used instead of a Kubernetes LoadBalancer service
Traffic is routed at Layer 7 (HTTP/HTTPS)
All configuration is managed via values.yaml
The official DataHub Helm chart was used without modification
 


below are the commands used in order to create an application gateway ,firewall ,network and public ip ,network peering 


az network application-gateway waf-policy create -g myResourceGroup -n myWAFPolicy

az network public-ip create -n myPublicIp -g datahub-rg --allocation-method Static --sku Standard
az network vnet create -n myVnet -g datahub-rg --address-prefix 10.0.0.0/16 --subnet-name mySubnet --subnet-prefix 10.0.0.0/24
az network application-gateway create -n myApplicationGateway -l eastus -g datahub-rg --sku WAF_v2 --public-ip-address myPublicIp --vnet-name myVnet --subnet mySubnet --priority 100 --waf-policy /subscriptions/{subscription_id}/resourceGroups/datahub-rg/providers/Microsoft.Network/ApplicationGatewayWebApplicationFirewallPolicies/myWAFPolicy


appgwId=$(az network application-gateway show -n myApplicationGateway -g github-rg -o tsv --query "id")
az aks enable-addons -n myCluster -g github-rg -a ingress-appgw --appgw-id $appgwId


nodeResourceGroup=$(az aks show -n myCluster -g myResourceGroup -o tsv --query "nodeResourceGroup")
aksVnetName=$(az network vnet list -g $nodeResourceGroup -o tsv --query "[0].name")

aksVnetId=$(az network vnet show -n $aksVnetName -g $nodeResourceGroup -o tsv --query "id")
az network vnet peering create -n AppGWtoAKSVnetPeering -g myResourceGroup --vnet-name myVnet --remote-vnet $aksVnetId --allow-vnet-access

appGWVnetId=$(az network vnet show -n myVnet -g myResourceGroup -o tsv --query "id")
az network vnet peering create -n AKStoAppGWVnetPeering -g $nodeResourceGroup --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access


In order to use the ingress controller to expose frontend pod, we need to update the datahub-frontend section of the values.yaml file that was used to deploy DataHub. Here is a sample configuration:

datahub-frontend:
  enabled: true
  image:
    repository: acryldata/datahub-frontend-react
    # tag: "v0.10.0 # defaults to .global.datahub.version

  # Set up ingress to expose react front-end
 
  <img width="393" height="185" alt="image" src="https://github.com/user-attachments/assets/5e44a0f7-f742-44ce-85bb-c36e43e281b0" />




---
helm upgrade --install datahub datahub/datahub --values values.yaml

kubectl get ingress
