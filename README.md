# Demobook on Azure Container Instances (ACI)

A small demo that packages a static web page into a Docker image (nginx), stores the image in Azure Container Registry (ACR), and runs it on Azure Container Instances (ACI) with a public URL.

## Project structure

```
demobook/
├── Dockerfile    # nginx:alpine image that serves index.html
├── index.html    # the web page
└── README.md
```

## Architecture

```
Local project -> Docker image -> Azure Container Registry -> Azure Container Instances -> Public URL
```

## Prerequisites

- Azure account (Azure for Students works)
- Azure CLI installed
- Docker Desktop installed and running
- Git

## Steps

### 1. Login to Azure

```
az login
az account show
```

### 2. Register the providers (one time)

```
az provider register --namespace Microsoft.ContainerInstance
az provider register --namespace Microsoft.ContainerRegistry
```

### 3. Create a resource group

```
az group create --name ACI-DevOps-Demo-RG --location centralindia
```

### 4. Create the container registry

The registry name must be globally unique, lowercase letters and numbers only.

```
az acr create --resource-group ACI-DevOps-Demo-RG --name demobookacrsonu134 --sku Basic
az acr update --name demobookacrsonu134 --admin-enabled true
```

### 5. Build and push the image

```
az acr login --name demobookacrsonu134
docker build -t demobookacrsonu134.azurecr.io/demobook:v1 .
docker push demobookacrsonu134.azurecr.io/demobook:v1
az acr repository list --name demobookacrsonu134 --output table
```

The last command should list `demobook`.

### 6. Get the registry password

Windows Command Prompt:

```
for /f "delims=" %i in ('az acr credential show --name demobookacrsonu134 --query "passwords[0].value" --output tsv') do set ACRPW=%i
```

### 7. Deploy to Azure Container Instances

```
az container create --resource-group ACI-DevOps-Demo-RG --name aci-demobook-app --image demobookacrsonu134.azurecr.io/demobook:v1 --registry-login-server demobookacrsonu134.azurecr.io --registry-username demobookacrsonu134 --registry-password "%ACRPW%" --dns-name-label aci-demobook-devmritunjai --ports 80 --os-type Linux --cpu 1 --memory 1
```

### 8. Get the public URL

```
az container show --resource-group ACI-DevOps-Demo-RG --name aci-demobook-app --query ipAddress.fqdn --output tsv
```

Open `http://<the-output>` in a browser (use `http`, not `https`).

### 9. Check status and logs

```
az container show --resource-group ACI-DevOps-Demo-RG --name aci-demobook-app --query instanceView.state --output tsv
az container logs --resource-group ACI-DevOps-Demo-RG --name aci-demobook-app
```

## Cleanup

Delete all resources to avoid using up credits:

```
az group delete --name ACI-DevOps-Demo-RG --yes --no-wait
```

## Common errors

| Error | Cause and fix |
|---|---|
| `InaccessibleImage` | Wrong registry name or password, or the image was not pushed. Check with `az acr repository list`. |
| `DOCKER_COMMAND_ERROR` | Docker Desktop is not running. Start it and retry. |
| `MissingSubscriptionRegistration` | Run the `az provider register` commands from step 2 and wait a minute. |
| DNS name label already in use | Change `--dns-name-label` to something unique. |

## Security note

Never commit registry passwords or other secrets to this repository. Load them into environment variables as shown above.

## Author

Sonu, MCA, PES University
