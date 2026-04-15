# ASP.NET Core Web API on Azure App Service (Linux)

This repository shows how to build and deploy a .NET 8 minimal API to Azure App Service (Linux) using Azure Pipelines and Bicep deployment stacks.

## What this repo does

- Builds and publishes the API in [src/Gardiner.Api](src/Gardiner.Api)
- Provisions Azure infrastructure from [main.bicep](main.bicep)
- Deploys the app package to App Service on Linux
- Runs a post-deployment smoke test against `/weatherforecast`
- Supports scheduled cleanup through [destroy-pipeline.yml](destroy-pipeline.yml)

## Project structure

- [src/Gardiner.Api](src/Gardiner.Api): ASP.NET Core 8 minimal API
- [main.bicep](main.bicep): App Service Plan + Linux Web App
- [azure-pipelines.yml](azure-pipelines.yml): build, deploy infra, deploy app, smoke test
- [destroy-pipeline.yml](destroy-pipeline.yml): scheduled deletion of the deployment stack

## Prerequisites

- .NET SDK 8.x (uses [global.json](global.json))
- Azure subscription
- Azure DevOps project + Azure Pipelines
- An Azure service connection in Azure DevOps named `sp-dotnet-linux-australiaeast`

## Local development

From repository root:

```bash
cd src/Gardiner.Api
dotnet restore
dotnet run
```

Test endpoint:

```bash
curl http://localhost:5000/weatherforecast
```

If HTTPS is enabled locally, use the HTTPS URL printed by `dotnet run`.

## Azure setup

Create the resource group used by pipelines:

```bash
az group create --name rg-dotnet-linux-australiaeast --location australiaeast
```

Create an Azure DevOps service connection that can manage this resource group, then set the service connection name to:

- `sp-dotnet-linux-australiaeast`

If you use a different name, update both pipeline files:

- [azure-pipelines.yml](azure-pipelines.yml)
- [destroy-pipeline.yml](destroy-pipeline.yml)

## CI/CD pipeline behavior

[azure-pipelines.yml](azure-pipelines.yml) runs on pushes to `main` and performs:

1. Install .NET SDK from `global.json`
2. `dotnet publish` for all `.csproj` files under `src`
3. Create or update a deployment stack from `main.bicep`
4. Deploy the published zip to Linux App Service
5. Invoke `GET /weatherforecast` as a smoke test

### Infrastructure created

Current defaults in [main.bicep](main.bicep):

- App Service Plan: `plan-dotnet-linux-australiaeast` (B1)
- Web App: `app-dotnet-linux-australiaeast`
- Runtime: `DOTNETCORE|8.0`

## Scheduled cleanup pipeline

[destroy-pipeline.yml](destroy-pipeline.yml) is scheduled daily at midnight (UTC) and:

- checks whether deployment stack `dotnet` exists in `rg-dotnet-linux-australiaeast`
- deletes the stack with `--action-on-unmanage deleteResources`

This removes resources managed by the stack to help control costs in demo or sandbox subscriptions.

## Notes

- The Web App name in pipeline and Bicep must stay aligned.
- This sample uses fixed names and location for simplicity; parameterize them if you want reusable multi-environment deployments.
