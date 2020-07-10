

# SQL Migration

The next section introduces the instructions to help you migrate to on-premise SQL Server on Azure by using Data Migration Assistant **DMA**.

For the sake of simplicity, the demo provides a Virtual Machine in Azure with a built-in SQL Server. From now on, we will assume that this is our on-premise SQL server that needs to be migrated to Azure.

Go to the following link and download the DMA tool:

https://www.microsoft.com/en-us/download/details.aspx?id=53595

Open the Data Migration Assistant.

## DMA SQL Assessment

Create a new assessment:

1. Select the *New* (+) icon, and then select the *Assessment* project type.

2. Set the source and target server type. In this case, our source server is SQL Server and Azure SQL Database as a target server.

3. Select *Create*.

![dma-assessment-new](Images/dma-assessment-new.png)

4. Click next button in the wizard and add the following assessment options
 
![dma-assessment-add-options](Images/dma-assessment-add-options.png)

Click next button in the wizard and add a SQL source:

5. Select *Add Sources* to open the connection flyout menu.

6. Enter the on-premise SQL server instance name you want to migrate, choose and set the SQL authentication credentials, set the correct connection properties, and then select Connect.

7. Select the databases to assess, and then select *Add*.

![dma-assessment-add-source](Images/dma-assessment-add-source.png)

8. Select *Start Assessment*.

![dma-assessment-start](Images/dma-assessment-start.png)

9. Once the assessment has been finished, review the compatibility issues across all compatibility levels supported by the target SQL Server version that you selected, in this case Azure SQL. Ensure that there is no compatibility issues.

![dma-assessment-results](Images/dma-assessment-results.png)

## DMA SQL Migration

Create a new migration:

1. On the left pane, select *New* (+), and then select the *Migration project* type.

2. Set the source type to *SQL Server* and the target server type to *Azure SQL Database*.

3. Select as a *Migration Scope* Schema and Data.

4. Select *Create*.

![dma-migration-new](Images/dma-migration-new.png)

Add a sql source:

5. For the source, under *Connect to source server*, in the Server name text box, enter the name of the source SQL Server instance. 

6. Enter the on-premise SQL server instance name you want to migrate, choose and set the SQL authentication credentials, set the correct connection properties, and then select *Connect*.

![dma-migration-add-source](Images/dma-migration-add-source.png)

Create your targeted Azure SQL in Azure by following this steps:

https://docs.microsoft.com/en-us/azure/azure-sql/database/single-database-create-quickstart?tabs=azure-portal

Go back to DMA wizard and add an Azure sql target previously created:

7. For the target, under *Connect to target server*, in the Server name text box, enter the name of the Azure SQL Database instance.

8. Select the *Authentication type* supported by the target Azure SQL Database instance.

9. Select *Connect*.

![dma-migration-add-source](Images/dma-migration-add-target.png)

Select schema objects:

10. Select the schema objects from the source database that you want to migrate to Azure SQL Database.

11. Select *General SQL script*.

![dma-migration-add-tables](Images/dma-migration-add-tables.png)

Deploy sql schemas:

12. Select *Deploy schema*.

![dma-migration-deploy](Images/dma-migration-deploy.png)

13. After deploying and rewiewing the results select *migrate data*.

14. Select the tables with the data you want to migrate and start migration.

15. Ensure that the schema and data have successfully been deployed to Azure sql.

![dma-migration-data](Images/dma-migration-data.png)

16. Update the web.config file under the directory 'C:\inetpub\wwwroot\ttrewardsdemo' and replace the connection string with the connection provided by Azure SQL previously created.

```xml
 <connectionStrings>
    <add name="dbContext" connectionString="Data Source=ttrewardssqldemo.eastus.cloudapp.azure.com,1433;Initial Catalog=rewards;User ID=admintt;Password=<your_sql_pwd>;" providerName="System.Data.SqlClient" />
  </connectionStrings>
```

17. Navigate to TailwindTraders Web app and check that it is still working fine. In case that the IIS server is stopped open a new command prompt and execute the following command:

```
iisreset /start
```

# ASP.NET Migration

This section is intended to guide you through the process of migrating to on-premise ASP.NET web application on Azure by using **App Service Migration Assistant**.

For the sake of simplicity, the demo provides a Virtual Machine in Azure with a built-in IIS Server with our ASP.NET application already deployed. From now on, we will assume that this is our on-premise ASP.NET application that needs to be migrated to *Azure App Service*.

Go to the following link and download the App Service Migration Assistant tool:

https://appmigration.microsoft.com/readiness

Open the App Service Migration Assistant.

1. Select the default web site where your ASP.NET application is deployed.

![asma-start](Images/asma-start.png)

2. Click next button and check for errors in *Assessment Report*.

![asma-assessment-errors](Images/asma-assessment-errors.png)

In this case we got an error indicating that our on-premise web application is assigned to an application pool in IIS that runs under an unsupported user.

Since Azure App Services only supports system managed accounts associated with the application pool you must change the application pool identity in IIS:

3. Go to IIS and select *Application Pools*.

4. Right click to *Default Web Site* application pool and select *Advanced Settings*.

5. Replace the *Identity* property *LocalSystem* with *ApplicationPoolIdentity* and save.

![asma-assessment-apppool](Images/asma-assessment-apppool.png)

6. Run the assessment report again and make sure there is no error.

7. Continue with the migration wizard skipping the *Azure Migrate Hub* section.

8. Once you reach the *Login To Azure* section, follow the instructions to login using your Azure account and subscription you want to work with.

9. After finishing the login process, continue with the Azure options section and introduce the following settings for the Azure App Service creation:
    - Select an Azure subscription
    - Select or create a Resource group
    - Select a Region
    - Select or create an App service plan
    - Introduce an App Service website name
    - Check *Skip database setup*

![asma-migration-azureoptions](Images/asma-migration-azureoptions.png)

10. Select *Migrate*.

11. Ensure the migration process has successfully finished and select *Go to your website* i order to verify that your web site is running in your Azure App service.

![asma-migration-success](Images/asma-migration-success.png)


# GitHub Actions

GitHub provides a feature named **Actions** which allows setting up in your repository to build, test, package, release, or deploy any project. This section is aimed to create a new GitHub Action to automatize this workflow and avoid having to build and deploy to Azure App Service manually.

1. Go to 'https://github.com/microsoft/TailwindTraders-Rewards' and fork it to your GitHub repository.

![gh-actions-fork](Images/gh-actions-fork.png)

Before start creating a new action we need to define our Web site secrets such as your database connection string since secrets should never be pushed to your repository. GitHub provides a secret's store so that GitHub Actions are able to retrieve these secrets during execution. You can reference these secrets in your action scripts with **'${{ secrets.YOUR_SECRET_NAME }}'**.

3. Go to your GitHub repo you just forked, select *Settings* tab and navigate to *Secrets* on the left side menu.

![gh-actions-secrets](Images/gh-actions-secrets.png)

4. Create a new secret by clicking the *New secret button*.

5. Add a secret with name 'CONNECTION_STRINGS' and with the following values. Replace the connection string accordingly to your SQL Database.

```json
[
  {
    "name": "dbContext",
    "value": "Data Source=ttrewardssqldemo.database.windows.net,1433;Initial Catalog=rewards;User ID=<my_userId>;Password=<my_password>;",
    "slotSetting": false
  }
]
```

When deploying a resource to Azure, GitHub Actions need credentials to be able to connect to Azure. 

6. Create a new secret with name 'AZURE_CREDENTIALS' and with the following value.

```json
{
    "clientId": "<Azure Service principal Id>",
    "clientSecret": "<Azure Service principal Secret>",
    "subscriptionId": "<Azure Subscription Id>",
    "tenantId": "<Azure Tenant Id>"
}
```
Options:
- ClientId: Azure Service principal Id
- ClientSecret: Azure Service principal Secret
- SubscriptionId: Subscription where the your App Service is created
- TenantID: Tenant you are working with

In case you don't have service principal credentials you can obtain new credentials by executing the following command:

```
# Replace {subscription-id}, {resource-group} with the subscription, resource group details of the WebApp

az ad sp create-for-rbac --name "myApp" --role contributor \
                            --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                            --sdk-auth
```

Once you have configured your secrets you can start creating a new Action:

2. Select Actions tab and click *New workflow*.

3. When asking for a new template just select 'Skip this and set up a workflow yourself'.

![gh-actions-new-workflow](Images/gh-actions-new-workflow.png)

2. Name the new workflow with 'tt-rewards-demo.cicd.yml', copy the following content into the *edit file* section and replace the 'AZURE_WEBAPP_NAME' environment variable with the name of your Azure App Service site.

```yml
name: Build and deploy TailwindTraders Rewards ASP.NET MVC App to Azure Web App

on:
  push:
    branches:
      - master
env:
  AZURE_WEBAPP_NAME: ttrewardswebdemo    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: './Source'      # set this to the path to your web app project, defaults to the repository root
  NUGET_VERSION: '5.3.1'           # set this to the dot net version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:
    - uses: actions/checkout@master  
    - name: Install Nuget
      uses: nuget/setup-nuget@v1
      with:
        nuget-version: ${{ env.NUGET_VERSION}}
    - name: NuGet to restore dependencies as well as project-specific tools that are specified in the project file
      run: nuget restore '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/TailwindTraders.Rewards.Website.sln' 
    - name: Add msbuild to PATH
      uses: microsoft/setup-msbuild@v1.0.0 
    - name: Run MSBuild
      run: msbuild '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/TailwindTraders.Rewards.Website.sln'  
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - name: 'Run Azure webapp deploy action'
      uses: azure/webapps-deploy@v2
      with: 
        app-name: ${{ env.AZURE_WEBAPP_NAME }} # Replace with your app name
        package: '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/Website/'
    - uses: azure/appservice-settings@v1
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        connection-strings-json: '${{ secrets.CONNECTION_STRINGS }}'
        general-settings-json: '{"alwaysOn": "false", "webSocketsEnabled": "true", "netFrameworkVersion": "v4.0"}' 
      id: settings
```
![gh-actions-edit](Images/gh-actions-edit.png)


7. Select *Start Commit* and commit the changes.

![gh-actions-commit](Images/gh-actions-commit.png)

After having committed the change, a new Action workflow should be triggered.

7. Go back to Actions tab, select your Action workflow and verify that the job has successfully been completed.

![gh-actions-completed](Images/gh-actions-completed.png)