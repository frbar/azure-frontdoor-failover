# Purpose

This repository contains a Bicep template to setup:
- 2 App Service Linux,
- Azure Front Door in front of both with priority 1 and 2

There is also a very basic API backend.

# Deploy the infrastructure

```powershell
$subscription = "Training Subscription"

az login
az account set --subscription $subscription

$rgName = "frbar-fd-fo"
$envName = "fbz12"
$location = "West Europe"

az group create --name $rgName --location $location
az deployment group create --resource-group $rgName --template-file infra.bicep --mode complete --parameters envName=$envName
```

# Build and Deploy the API backend

```powershell
dotnet publish .\api\ -r linux-x64 --self-contained -o publish
Compress-Archive publish\* publish.zip -Force
az webapp deployment source config-zip --src .\publish.zip -n "$($envName)-api-0" -g $rgName
az webapp deployment source config-zip --src .\publish.zip -n "$($envName)-api-1" -g $rgName

curl "https://$($envName)-api-0.azurewebsites.net/health" -UseBasicParsing
curl "https://$($envName)-api-1.azurewebsites.net/health" -UseBasicParsing

Remove-Item publish -Recurse
Remove-Item publish.zip
```

# Get the Azure Front Door endpoint

And note it for later reference.

```powershell
az afd endpoint list -g $rgName --profile-name "$($envName)-afd" --query [0].hostName
```

# Tear down

```powershell
az group delete --name $rgName
```

