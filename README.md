# ASP.NET Core Web API app deployed to Azure App Service (Linux)

Deploy an ASP.NET Core app to an Azure App Service (Linux) using Azure Pipelines

## Infrastructure notes

```bash
az group create --name rg-dotnet-linux-australiaeast --location australiaeast
```

Create a new service connection in Azure Pipelines that can access this resource group
