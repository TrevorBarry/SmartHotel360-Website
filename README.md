# SmartHotel360 Website

> [!WARNING]
> This project is retired, archived, and no longer supported. You are welcome to continue using and forking the repository. The codebase targets older .NET, Visual Studio, and JavaScript dependencies, so some setup steps may require historical tooling.

This repository contains the website experience for the SmartHotel360 reference application. The website is part of a broader SmartHotel360 sample ecosystem and demonstrates a public hotel booking experience built with ASP.NET Core, React, and Redux, plus an optional Azure Function used for the pet-checker workflow.

If you missed the original demos, you can still watch [Scott Guthrie’s Connect(); 2017 keynote](https://channel9.msdn.com/Events/Connect/2017/K100) and Scott Hanselman's Ignite 2018 session, [An end-to-end tour of the Microsoft developer platform](https://myignite.techcommunity.microsoft.com/sessions/66696#ignite-html-anchor).

![SmartHotel360 home page](Documents/Images/screen1.png)
![SmartHotel360 search experience](Documents/Images/screen2.png)

## Related SmartHotel360 repositories

- [SmartHotel360](https://github.com/Microsoft/SmartHotel360)
- [SmartHotel360-IoT](https://github.com/Microsoft/SmartHotel360-IoT)
- [SmartHotel360-MixedReality](https://github.com/Microsoft/SmartHotel360-MixedReality)
- [SmartHotel360-Backend](https://github.com/Microsoft/SmartHotel360-Backend)
- [SmartHotel360-Website](https://github.com/Microsoft/SmartHotel360-Website)
- [SmartHotel360-Mobile](https://github.com/Microsoft/SmartHotel360-Mobile)
- [SmartHotel360-SentimentAnalysis](https://github.com/Microsoft/SmartHotel360-SentimentAnalysis)
- [SmartHotel360-Registration](https://github.com/Microsoft/SmartHotel360-Registration)

## Repository contents

- `Source/` - Visual Studio solution, website project, and Azure Function project
- `Deploy/` - Azure Resource Manager template for provisioning required resources
- `Documents/` - screenshots and demo scripts

## Solution overview

Open `Source/SmartHotel360.Website.sln` in Visual Studio 2017 15.5 or later. The solution contains two projects:

- `SmartHotel360.Website` - an [ASP.NET Core](https://dotnet.microsoft.com/) website using React, Redux, and server-side rendering
- `SmartHotel360.WebsiteFunction` - an [Azure Function](https://azure.microsoft.com/services/functions/) that analyzes pet photos using Cognitive Services Vision API, Azure Cosmos DB, and SignalR

## Prerequisites

To work with this repository locally, install:

- Visual Studio 2017 15.5 or later
- Node.js
- Azure CLI
- An Azure subscription and related Azure resources if you want to use the optional pet-checker workflow
- Azure Functions tooling if you want to run or publish the function locally

> [!NOTE]
> This is a historical sample that targets older runtimes such as .NET Core 2.1 and older npm packages. Modern environments may require additional compatibility work.

## Quick start: run the website locally

The simplest local path is to run only the website and use the public backend endpoints.

1. Open `Source/SmartHotel360.Website.sln` in Visual Studio.
2. Set `SmartHotel360.Website` as the startup project.
3. Start debugging with `F5`.
4. The website is configured to use public backend endpoints by default, so you do not need to run the backend locally.

## Website configuration

`SmartHotel360.Website/appsettings.Development.json` contains the local website settings.

- `SettingsUrl` - URL of the configuration endpoint. By default, it points to the public backend service. Change it only if you have deployed your own instance of [SmartHotel360-Backend](https://github.com/Microsoft/SmartHotel360-Backend).
- `FakeAuth` - local settings used to simulate sign-in. It includes `Name`, `UserId`, and `PicUrl`.

If `FakeAuth` is not set, the website uses [Azure Active Directory B2C](#azure-ad-b2c-optional) for sign-in.

### Optional website settings for the pet checker

To enable the pet-checker feature, update the `PetsConfig` section in `appsettings.Development.json` or `appsettings.Production.json`:

- `blobName` - storage account name
- `blobKey` - storage account key
- `cosmosUri` - Cosmos DB endpoint, for example `https://petpictureuploadmetadata.documents.azure.com:443/`
- `cosmosKey` - Cosmos DB key
- `api` - Azure Function URL, for example `http://petchecker.azurewebsites.net`

## Optional: run the Azure Function for pet checking

The Azure Function is optional. You only need it if you want to run or deploy the pet-checker workflow yourself.

Update `SmartHotel360.WebsiteFunction/local.settings.json` with these values:

- `AzureWebJobsStorage` - storage account connection string
- `AzureWebJobsDashboard` - storage account connection string
- `cosmos_uri` - Cosmos DB SQL API endpoint, for example `https://petpictureuploadmetadata.documents.azure.com:443`
- `cosmos_key` - Cosmos DB SQL API key
- `constr` - Cosmos DB SQL API connection string
- `MicrosoftVisionApiKey` - Cognitive Services Vision API key
- `MicrosoftVisionApiEndpoint` - Cognitive Services Vision API URL, for example `https://southcentralus.api.cognitive.microsoft.com/vision/v1.0`
- `MicrosoftVisionNumTags` - number of tags to fetch from Vision API, for example `10`
- `AzureSignalRConnectionString` - SignalR Service connection string

### Run the website and function together

If you want to debug both projects locally:

1. Open solution properties.
2. Select **Multiple startup projects**.
3. Set both projects to **Start**.

![](Documents/Images/multiple-startup.png)

If the function is not triggered, verify that:

1. Your blob storage contains a public container named `pets`.
2. `local.settings.json` contains the correct `cosmos_uri`, `cosmos_key`, and `constr` values.
3. The website is saving pet images to the `pets` container and creating documents in the `checks` collection in the `pets` database.

![Pets document in Cosmos DB](Documents/Images/pets-document.png)

## Azure resources and deployment

The optional pet-checker scenario requires the following Azure resources:

- Azure Function
- Blob Storage
- Cosmos DB
- Cognitive Services Vision API
- SignalR Service
- App Service
- Application Insights

An Azure Resource Manager template is included to help provision these resources:

[![Deploy to Azure](Documents/Images/deploy-to-azure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FSmartHotel360-Website%2Fmaster%2FDeploy%2Fdeployment.json)

> [!NOTE]
> Some Azure resources are required even if you plan to run the function locally.

After provisioning resources, complete these manual steps:

1. Create a blob storage container named `pets` and make it publicly accessible.
2. Create a Cosmos DB database named `pets` and add a collection named `checks`.

![Public Blob Storage](Documents/Images/blob.png)
![Cosmos DB collection](Documents/Images/collection1.jpg)

### Deploy the website

You can use Visual Studio's [Publish](https://learn.microsoft.com/visualstudio/deployment/quickstart-deploy-to-a-web-site) feature to deploy the website to Azure App Service.

If you deploy the website to App Service and disable `FakeAuth` to use Azure AD B2C, switch back to `FakeAuth` if your deployed redirect URL is not registered in B2C.

### Deploy the function

You can publish the Azure Function from Visual Studio, but `local.settings.json` values are not published automatically. Add those keys manually during the initial publish flow.

You can also publish the function with Azure Functions Core Tools:

```bash
func azure functionapp publish <YourFunctionAppName> --publish-local-settings -i
```

Make sure you are signed in with the [Azure CLI](https://learn.microsoft.com/cli/azure/authenticate-azure-cli) before running the publish command.

## Azure AD B2C (optional)

The website can use [Azure AD B2C](https://azure.microsoft.com/services/active-directory-b2c/) for sign-in with Live ID as the external provider.

To enable it:

1. Remove the `FakeAuth` section from `appsettings.Development.json`.
2. Run the website at `https://localhost:57458`.

> [!IMPORTANT]
> Do not change the local port. The redirect URL configured in B2C expects `https://localhost:57458`.

> [!NOTE]
> B2C requires HTTPS. The historical public backend for this sample was not configured with SSL, so this flow may require additional backend updates in a modern environment.

## Demo scripts

`Documents/DemoScripts/` contains historical demo materials for the original SmartHotel360 sessions:

- [Azure Functions Local Debugging](Documents/DemoScripts/AzureFunctionsNETCoreDebugging.pdf)
- [App Service Production Debugging & Application Insights](Documents/DemoScripts/ProductionNETCoreDebugging.pdf)

## Contributing

This project welcomes contributions and suggestions, but because the repository is archived they may not be actively reviewed.

Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant Microsoft the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide a CLA and decorate the PR appropriately. You only need to complete this process once across all repositories using the Microsoft CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information, see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com).
