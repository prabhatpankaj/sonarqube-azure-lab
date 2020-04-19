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
az acr create --resource-group SonarqubeResourceGroup --name sonarqubeacruniquename --sku Basic
```

* Gather data to set env variables

```
export YOUR_KEY_VAULT="SonarqubeResource-Vault2"
az keyvault secret set --vault-name $YOUR_KEY_VAULT --name 'sonarqube-sql-admin' --value 'admindemo'
az keyvault secret set --vault-name $YOUR_KEY_VAULT --name 'sonarqube-sql-admin-password' --value '01@Pass1234'
az keyvault secret set --vault-name $YOUR_KEY_VAULT --name 'container-registry-admin' --value 'admindemo'
az keyvault secret set --vault-name $YOUR_KEY_VAULT --name 'container-registry-admin-password' --value '01@Pass1234'

```
* Here is the set of variables used throughout this guide. Notice we retrieve Azure Keyvault secrets using the Azure CLI and not with a Service Principal. This would be more secure but you Azure user account is required to be permitted to work with Service Principal users.
```

# General
export PROJECT_PREFIX="demo"
export RESOURCE_GROUP_NAME="$PROJECT_PREFIX-sonarqube-rg"
export LOCATION="eastus"

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
export CONTAINER_REGISTRY_NAME="Sonarqubeacruniquename"
export CONTAINER_REGISTRY_FQDN="$CONTAINER_REGISTRY_NAME.azurecr.io"
export REG_ADMIN_USER=`az keyvault secret show -n container-registry-admin --vault-name $YOUR_KEY_VAULT | jq -r '.value'`
export REG_ADMIN_PASSWORD=`az keyvault secret show -n container-registry-admin-password --vault-name $YOUR_KEY_VAULT | jq -r '.value'`
export WEBAPP_NAME="$PROJECT_PREFIX-sonarqube-webapp"
export CONTAINER_IMAGE_NAME="$PROJECT_PREFIX-sonar"
export CONTAINER_IMAGE_TAG="demo"

# Concatenated variable strings for better readability
export DB_CONNECTION_STRING="jdbc:sqlserver://$SQL_SERVER_NAME.database.windows.net:1433;database=$DATABASE_NAME;user=$SQL_ADMIN_USER@$SQL_SERVER_NAME;password=$SQL_ADMIN_PASSWORD;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;"

```
* Build Sonarqube Docker image and push it to private registry

```
# Checkout the repository containing the Dockerfile and the config script
git clone https://github.com/prabhatpankaj/sonarqube-azure-lab.git && cd sonarqube-azure-lab

# Log into your Azure Container Registry  
az acr login --name $CONTAINER_REGISTRY_NAME

# Build the image and push to the registry
docker build -t $CONTAINER_IMAGE_NAME:$CONTAINER_IMAGE_TAG dockerfiles/sonarqube/.
docker tag $CONTAINER_IMAGE_NAME:$CONTAINER_IMAGE_TAG "$CONTAINER_REGISTRY_FQDN/$CONTAINER_IMAGE_NAME:$CONTAINER_IMAGE_TAG"
docker push "$CONTAINER_REGISTRY_FQDN/$CONTAINER_IMAGE_NAME:$CONTAINER_IMAGE_TAG"
```

* Create the managed SQL database

```
# Add resource group; tag appropriately :-)
az group create \
    --name $RESOURCE_GROUP_NAME \
    --location $LOCATION \
    --tag 'createdBy=admin' 'createdFor=Resource group for SonarQube components'

# Create sql server and database
az sql server create \
    --name $SQL_SERVER_NAME \
    --resource-group $RESOURCE_GROUP_NAME \
    --location $LOCATION \
    --admin-user $SQL_ADMIN_USER \
    --admin-password $SQL_ADMIN_PASSWORD
az sql db create \
    --resource-group $RESOURCE_GROUP_NAME \
    --server $SQL_SERVER_NAME \
    --name $DATABASE_NAME \
    --service-objective $DATABASE_SKU \
    --collation "SQL_Latin1_General_CP1_CS_AS"

# Set SQL server's firewall rules to accept requests from Azure services only (this is going to be our Azure Webapp)
az sql server firewall-rule create \
    --resource-group $RESOURCE_GROUP_NAME \
    --server $SQL_SERVER_NAME -n "AllowAllWindowsAzureIps" \
    --start-ip-address 0.0.0.0 \
    --end-ip-address 0.0.0.0
```

