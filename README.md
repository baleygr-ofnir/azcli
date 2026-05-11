# **azcli** -
## Automated deployment of Azure App Service with SQL database and Key Vault deployment including GitHub Actions CI/CD base

## **az-webapp-publish** <resource-group-name> <linux-runtime-string> <price-tier> -
### Orchestrator for all the script functions:

- Available runtimes: az webapp list-runtimes --os-type linux

- Price Tiers: {B1, B2, B3, D1, F1, FREE, I1MV2, I1V2, I2MV2, I2V2, I3MV2, I3V2, I4MV2, I4V2, I5MV2, I5V2, I6V2, P0V3, P0V4, P1MV3, P1MV4, P1V2, P1V3, P1V4, P2MV3, P2MV4, P2V2, P2V3, P2V4, P3MV3, P3MV4, P3V2, P3V3, P3V4, P4MV3, P4MV4, P5MV3, P5MV4, S1, S2, S3, SHARED, WS1, WS2, WS3}

- Example: az-webapp-publish Azure-RG-Name DOTNETCORE:10.0 F1

## **webapp-deploy** with script functions:
### app_plan_create
### webapp_create
- Creates webapp that returns its Principal ID for later use
### webapp_set_connstring
- Configures environment variable on the web app for SQL Database Connection String with Key Vault Secret URI
### webapp_deploy
- Configures OIDC for GitHub Actions, a workflow and triggers it when pushing the workflow to the repository. Checks in the beginning if working directory is a configured git repository, if not makes sure it is and also attemps to create GitHub repository with GitHub CLI and push to it.

## **sqlsrv-deploy** with script funtions:
### sqlsrv_create

### sqldb_create

### sqlsrv_whitelist_appsvc_ips


## **keyvault-deploy** with script functions:
### keyvault_create

### keyvault_assign_admin_permissions
- Assigns Key Vault Secrets Officer to the currently azcli signed-in user

### keyvault_assign_app_svc_permissions
- Assigns Key Vault Secrets User to the web app

### keyvault_set_connstring_secret

Uses name of working directory for project name as it is currently designed to be run from the solution root directory. Project name is then used to create resource names.

SQL Database user is grabbed from GitHub CLI signed-in username

All functions are configured idempotently such that it will not attempt creating any resource that already exists, in case there as issues in some part and it has to be run again.

TODO: App Insights+Log Analytics Workspace
