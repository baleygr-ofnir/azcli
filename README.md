# azcli - Automated Azure Infrastructure & CI/CD Pipeline

This repository provides a fully automated, idempotent, and secure deployment suite for a .NET 10 Web API on Azure . It utilizes a modular "Infrastructure as Code" (IaC) architecture to provision and configure all necessary cloud resources—App Service, SQL Database, Key Vault, and Storage—while enforcing production-safe security standards.

## Orchestrator: `az-webapp-publish`

The `az-webapp-publish` script serves as the main execution engine. It handles parameter parsing, global variable assignment, and coordinates the execution of all sourced functions in the correct logical sequence to handle resource dependencies.

### Usage

```bash
./az-webapp-publish -g <resource_group> -r <runtime> -t <price_tier>

```

* **-g**: The Azure Resource Group to target.
* **-r**: The Linux Runtime stack (e.g., `DOTNETCORE:10.0`).
* **-t**: The App Service Pricing Tier (e.g., `F1`, `B1`, `S1`, `P1V2`).

---

## Modular Script Breakdown

### 1. `webapp-deploy` (Core Application Service)

Manages the application environment, platform configurations, and CI/CD setup.

* **`app_plan_create`**: Checks for and provisions the Linux App Service Plan.

* **`webapp_create`**: Provisions the Web App with **HTTPS Only** enabled, **System-Assigned Managed Identity**, and sets the environment to "Development" for Scalar access.

* **`webapp_auth_gh`**: Configures **OpenID Connect (OIDC)** federated credentials between Azure and GitHub, enabling passwordless deployments without long-lived secrets.

* **`webapp_deploy`**: Initializes the local Git repository, creates the GitHub repository via `mkghrepo`, sets repository secrets, and pushes a customized `.github/workflows/az-workflow.yml` for automated .NET 10 builds.

* **`webapp_set_connstring`**: Configures the app setting for the SQL Connection String using a secure **Key Vault Reference**.

* **`webapp_set_ip_restrictions`**: Implements inbound firewall rules to restrict access solely to the developer's public IP.

* **`webapp_configure_monitoring`**: Enables Application Insights by injecting the connection string via Key Vault reference and setting necessary agent extensions.

* **`webapp_configure_backup`**: Schedules **daily snapshots** to a storage container, with logic to skip unsupported tiers like F1/Free.

* **`webapp_configure_storage_mount`**: Mounts an Azure File Share to `/site/wwwroot/static` to fulfill the requirement for handling static assets.

### 2. `sqlsrv-deploy` (Data Layer)

Handles the persistent data infrastructure and network security.

* **`sqlsrv_create`**: Provisions a logical Azure SQL Server.

* **`sqlsrv_whitelist_appsvc_ips`**: Dynamically whitelists the Web App's outbound IP addresses in the SQL Server firewall.

* **`sqldb_create`**: Creates a Basic tier SQL database suitable for development.

### 3. `keyvault-deploy` (Identity & Secret Management)

Implements zero-trust security via Managed Identity and RBAC.

* **`keyvault_create`**: Provisions a Key Vault with RBAC authorization enabled.

* **`keyvault_assign_admin_permissions`**: Assigns the "Key Vault Secrets Officer" role to the current signed-in user.

* **`keyvault_assign_appsvc_permissions`**: Grants the Web App's Managed Identity the "Key Vault Secrets User" role.

* **`keyvault_set_connstring_secret`**: Securely stores the database connection string as a secret in the vault.

### 4. `storage-deploy` (Asset & Log Storage)

Manages the resources for backups, logs, and static files.

* **`storage_create`**: Provisions a StorageV2 account with TLS 1.2 and disabled public blob access.

* **`storage_create_container`**: Creates the `webapp-backups` container for system backups.

* **`storage_mount_file_share`**: Provisions the Azure File Share for application static resource hosting.

### 5. `monitoring-deploy` (Observability)

Configures performance tracking and logging infrastructure.

* **`monitoring_create_law`**: Provisions the Log Analytics Workspace.

* **`monitoring_create_appinsights`**: Provisions the Application Insights component.

* **`monitoring_store_secret`**: Stores the AI connection string in Key Vault for secure consumption via Managed Identity.

### 6. `mkghrepo` (Repository Automation)

Automates the local and remote repository setup.

* **`github_cli_installed`**: Verifies system dependencies.

* **`git init & create`**: Initializes the local repo, creates a public repository on GitHub via `gh repo create`, and establishes the remote.

---

## Technical Design Principles

* **Idempotency**: Every function utilizes `show` or `list` commands to verify resource existence before creation, allowing for safe script re-execution.
* **Modular Architecture**: Scripts are organized by service domain and sourced into a single orchestrator context.
* **Zero-Trust Security**: Secrets are never handled in plain text. OIDC eliminates long-lived secrets in GitHub, and Key Vault References eliminate plain-text environment variables.
* **Automated Context**: The suite dynamically identifies your public IP for firewalls and your GitHub credentials for repository management.
