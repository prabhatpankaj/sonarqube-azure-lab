# sonarqube-azure-lab

* Install Azure CLI on macOS

```
brew update && brew install azure-cli
```

* To sign in to Azure using the CLI you can type:

```
az login

```

* Create a resource group

```
az group create --name "SonarqubeResourceGroup" --location eastus

```

* Create an Azure Key Vault instance

```
az keyvault create --name "SonarqubeResource-Vault2" --resource-group "SonarqubeResourceGroup" --location eastus

```

* Create an Azure Container Registry

```
az acr create --resource-group SonarqubeResourceGroup --name Sonarqubeacruniquename --sku Basic
```

* Gather data to set env variables

```
export YOUR_KEY_VAULT="SonarqubeResource-Vault2"
az keyvault secret set --vault-name $YOUR_KEY_VAULT --name 'sonarqube-sql-admin' --value 'admin'
az keyvault secret set --vault-name $YOUR_KEY_VAULT --name 'sonarqube-sql-admin-password' --value 'pass1234'
az keyvault secret set --vault-name $YOUR_KEY_VAULT --name 'container-registry-admin' --value 'admin'
az keyvault secret set --vault-name $YOUR_KEY_VAULT --name 'container-registry-admin-password' --value 'pass1234'

```
* Here is the set of variables used throughout this guide. Notice we retrieve Azure Keyvault secrets using the Azure CLI and not with a Service Principal. This would be more secure but you Azure user account is required to be permitted to work with Service Principal users.
```

# General
export PROJECT_PREFIX="<your-project-prefix>"
export RESOURCE_GROUP_NAME="$PROJECT_PREFIX-sonarqube-rg"
export LOCATION="westeurope"

# SQL database related
export SQL_ADMIN_USER=`az keyvault secret show -n sonarqube-sql-admin --vault-name $YOUR_KEY_VAULT | jq -r '.value'`
export SQL_ADMIN_PASSWORD=`az keyvault secret show -n sonarqube-sql-admin-password --vault-name $YOUR_KEY_VAULT | jq -r '.value'`
export SQL_SERVER_NAME="$PROJECT_PREFIX-sql-server"
export DATABASE_NAME="$PROJECT_PREFIX-sonar-sql-db"
export DATABASE_SKU="S0"

# Webapp related 
export APP_SERVICE_NAME="$PROJECT_PREFIX-sonarqube-app-service"
export APP_SERVICE_SKU="S1"

# Container image related
export CONTAINER_REGISTRY_NAME="<your-acr-name>"
export CONTAINER_REGISTRY_FQDN="$CONTAINER_REGISTRY_NAME.azurecr.io"
export REG_ADMIN_USER=`az keyvault secret show -n container-registry-admin --vault-name $YOUR_KEY_VAULT | jq -r '.value'`
export REG_ADMIN_PASSWORD=`az keyvault secret show -n container-registry-admin-password --vault-name $YOUR_KEY_VAULT | jq -r '.value'`
export WEBAPP_NAME="$PROJECT_PREFIX-sonarqube-webapp"
export CONTAINER_IMAGE_NAME="$PROJECT_PREFIX-sonar"
export CONTAINER_IMAGE_TAG="<Tag>"

# Concatenated variable strings for better readability
export DB_CONNECTION_STRING="jdbc:sqlserver://$SQL_SERVER_NAME.database.windows.net:1433;database=$DATABASE_NAME;user=$SQL_ADMIN_USER@$SQL_SERVER_NAME;password=$SQL_ADMIN_PASSWORD;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"

```
