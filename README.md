# 🍷 Wine Classifier — Azure Deployment Guide

A machine learning wine classifier deployed on **Azure App Service** via **GitHub Actions CI/CD**.

---

## 📋 Prerequisites

- Azure CLI installed and authenticated or use Azure clous shell in azure console. 
- GitHub repository with Actions enabled
- Python 3.11 application with a `startup.sh` script

---

## 🚀 Azure Infrastructure Setup

### 1. Login to Azure
```bash
az login
```

### 2. Create Resource Group
```bash
az group create --name wine-classifier-rg-demo --location westeurope
```

### 3. Create App Service Plan
```bash
az appservice plan create \
  --name wine-classifier-plan-demo \
  --resource-group wine-classifier-rg-demo \
  --sku B1 --is-linux
```

### 4. Create Web App
```bash
az webapp create \
  --resource-group wine-classifier-rg-demo \
  --plan wine-classifier-plan-demo \
  --name wine-classifier-demo-12345 \
  --runtime "PYTHON|3.11"
```

### 5. Configure App Settings
```bash
az webapp config appsettings set \
  --resource-group wine-classifier-rg-demo \
  --name wine-classifier-demo-12345 \
  --settings WEBSITES_PORT=8000 SCM_DO_BUILD_DURING_DEPLOYMENT=true
```

### 6. Set Startup Command
```bash
az webapp config set \
  --resource-group wine-classifier-rg-demo \
  --name wine-classifier-demo-12345 \
  --startup-file "bash startup.sh"
```

---

## 🔐 GitHub Actions Secrets

Go to: **Settings → Secrets and variables → Actions → New repository secret**

| Secret Name | Value |
|---|---|
| `AZURE_RESOURCE_GROUP` | `wine-classifier-rg-demo` |
| `AZURE_WEBAPP_NAME` | `wine-classifier-demo-12345` |
| `AZURE_CREDENTIALS` | Full JSON from `az ad sp create-for-rbac` (see below) |

### Generate `AZURE_CREDENTIALS`
```bash
az ad sp create-for-rbac \
  --name "wine-classifier-sp-demo" \
  --role contributor \
  --scopes /subscriptions/<YOUR_SUBSCRIPTION_ID>/resourceGroups/wine-classifier-rg-demo \
  --json-auth
```
Copy the entire JSON output as the secret value.

---

## ⚙️ Workflow Usage
```yaml
env:
  AZURE_RESOURCE_GROUP: ${{ secrets.AZURE_RESOURCE_GROUP }}
  AZURE_WEBAPP_NAME: ${{ secrets.AZURE_WEBAPP_NAME }}

- name: Azure Login
  uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
```

---

## 🔍 Useful Commands
```bash
# List web apps
az webapp list --resource-group wine-classifier-rg-demo --query "[].name" -o tsv

# Stream live logs
az webapp log tail \
  --resource-group wine-classifier-rg-demo \
  --name wine-classifier-demo-12345
```