# azcli Documentation

Automated deployment suite for Azure App Service, SQL Database, and Key Vault, featuring GitHub Actions CI/CD integration for .NET 10 applications.

## Core Orchestrator

### `az-webapp-publish`

The primary entry point that orchestrates the entire deployment sequence. It coordinates resources across web apps, SQL servers, monitoring, and storage.

**Usage:**

```bash
./az-webapp-publish -g <resource_group> -r <runtime> -t <price_tier>

```

* **Parameters**:
* `-g`: Target Azure Resource Group.
* `-r`: Linux runtime string (e.g., `DOTNETCORE:10.0`).
* `-t`: App Service Plan price tier (e.g., `P1V3`, `F1`).


* **Features**:
* Determines project names based on the current working directory.
* Retrieves the local public IP for automated whitelisting.
* Ensures idempotent execution by checking for existing resources before creation.

---

## Script Modules and Functions

### 1. Web App Deployment (`webapp-deploy`)

Handles the lifecycle of the Azure App Service and its integration with GitHub.

* `app_plan_create`: Creates a Linux-based App Service Plan using the specified price tier.
* `webapp_create`: Provisions the Web App with a System Assigned Managed Identity and forces HTTPS.
* `webapp_auth_gh`: Manages OpenID Connect (OIDC) between Azure and GitHub, including application registration and role assignments.
* `webapp_deploy`: Generates a `.github/workflows/az-workflow.yml` file for .NET 10, sets GitHub secrets, and pushes the code to trigger the initial CI/CD run.
* `webapp_set_connstring`: Maps the SQL connection string from Key Vault to the Web App's environment variables using a Key Vault reference.
* `webapp_set_ip_restrictions`: Configures inbound network security to only allow traffic from the developer's public IP address.
* `webapp_configure_monitoring`: Links Application Insights to the Web App using Key Vault-backed connection strings.
* `webapp_configure_backup`: Sets up daily backups to Azure Blob Storage (skipped for Free/F1 tiers).
* `webapp_configure_storage_logs`: Redirects web server logs to a designated storage container.
* `webapp_configure_storage_mount`: Mounts an Azure File Share to the Web App at `/site/wwwroot/static`.
* `webapp_configure_autoscale`: Defines CPU-based scaling rules (Scale-out at >70%, Scale-in at <30%).

### 2. SQL Server Deployment (`sqlsrv-deploy`)

Manages the database infrastructure.

* `sqlsrv_create`: Provisions an Azure SQL Server and prompts for an administrator password if a new server is required.
* `sqldb_create`: Creates a SQL Database using the "Basic" service objective.
* `sqlsrv_whitelist_appsvc_ips`: Automatically identifies and whitelists all possible outbound IP addresses of the Web App in the SQL firewall.

### 3. Key Vault Management (`keyvault-deploy`)

Secures application secrets and manages access policies via RBAC.

* `keyvault_create`: Creates a Key Vault with Azure RBAC authorization enabled.
* `keyvault_assign_admin_permissions`: Assigns the "Key Vault Secrets Officer" role to the currently signed-in CLI user.
* `keyvault_assign_appsvc_permissions`: Assigns the "Key Vault Secrets User" role to the Web App's Managed Identity.
* `keyvault_set_connstring_secret`: Formats the SQL connection string and stores it as a secure secret in the vault.

### 4. Monitoring (`monitoring-deploy`)

Sets up observability for the stack.

* `monitoring_create_law`: Provisions a Log Analytics Workspace for centralized logging.
* `monitoring_create_appinsights`: Creates an Application Insights component linked to the workspace.
* `monitoring_store_secret`: Stores the Application Insights connection string in Key Vault for secure retrieval.

### 5. Storage (`storage-deploy`)

Handles persistent storage and backups.

* `storage_create`: Provisions a General Purpose v2 Storage Account.
* `storage_create_container`: Creates a blob container named `webapp-backups`.
* `storage_mount_file_share`: Provisions an Azure File Share for static content.

### 6. GitHub Repository Setup (`mkghrepo`)

Automates local and remote repository synchronization.

* Initializes a local Git repository if one does not exist.
* Uses the GitHub CLI to create a public repository and push the local main branch.
* Supports multiple package managers (pacman, dpkg, rpm, apk, brew) to verify GitHub CLI installation.
