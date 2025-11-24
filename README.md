# Java-Spring-Boot-Application-Using-Azure-Container-Apps-ACR

This project demonstrates how to deploy a containerized Java (Spring Boot) application to Azure Container Apps using Azure Container Registry (ACR) and Azure CLI.
Repository : https://github.com/codiebyheaart/coin_base_api_temp
Docker file - https://github.com/codiebyheaart/java_docker_file_for_java17

*Technology Stack
Java 17 / Spring Boot
Docker
Azure Container Registry (ACR)
Azure Container Apps
Azure CLI (Cloud Shell)
GitHub Source Code

Architecture Overview

The deployment flow follows a modern cloud-native pipeline:

GitHub → Azure Cloud Shell → Azure Container Registry → Azure Container Apps


This workflow avoids local Docker dependency by using ACR Cloud Build.

🛠️ Step-by-Step Deployment Guide
1. Clone Repository in Azure Cloud Shell
git clone https://github.com/your-username/app_cloud.git
cd app_cloud

2. Create Resource Group
az group create --name app-rg --location eastus

3. Create Azure Container Registry
az acr create --name codieacr --resource-group app-rg --sku Basic --location eastus


Enable admin (to pull images later):

az acr update -n codieacr --admin-enabled true

4. Build & Push Docker Image Using ACR (No local Docker required)
az acr build -r codieacr -t appcloud:v1 .


This uses the Dockerfile inside the repo.

5. Register Required Azure Providers (One-time)
az provider register -n Microsoft.App --wait
az provider register -n Microsoft.Web --wait
az provider register -n Microsoft.OperationalInsights --wait

6. Create Container App Environment
az containerapp env create \
  --name app-env \
  --resource-group app-rg \
  --location eastus

7. Deploy Container App with ACR Credentials
az containerapp create \
  --name app-cloud \
  --resource-group app-rg \
  --environment app-env \
  --image codieacr.azurecr.io/appcloud:v1 \
  --target-port 8080 \
  --ingress external \
  --registry-server codieacr.azurecr.io \
  --registry-username $(az acr credential show -n codieacr --query username -o tsv) \
  --registry-password $(az acr credential show -n codieacr --query "passwords[0].value" -o tsv)

8. Get Application Public URL
az containerapp show \
  --name app-cloud \
  --resource-group app-rg \
  --query properties.configuration.ingress.fqdn \
  -o tsv

🔄 Updating the Application (CI/CD Style)

Build new image:

az acr build -r codieacr -t appcloud:v2 .


Update deployment:

az containerapp update \
  --name app-cloud \
  --resource-group app-rg \
  --image codieacr.azurecr.io/appcloud:v2

🧹 Clean Up Resources
az group delete -n app-rg --yes --no-wait

🎯 What This Project Demonstrates
This repository showcases:
Building cloud-native Java applications
Creating a secure container registry
Pushing images without local Docker
Deploying using Azure Container Apps (serverless containers)
Working with ACR credentials
Clean production-ready deployment flow

This is ideal for Fiverr portfolio, YouTube tutorials, interview preparation, and personal cloud learning.
