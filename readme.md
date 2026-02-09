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
