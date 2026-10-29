# Runtime, CI, containers and infrastructure

## Where the .NET version is set

| File | Setting | Value |
|---|---|---|
| global.json:3-4 | `sdk.version`, `rollForward` | `"8.0.x"`, `latestFeature`. The `global.json` documentation says `sdk.version` takes a full version number and supports no wildcards |
| Directory.Packages.props:4 | `TargetFramework` | `net8.0` (applies to all projects) |
| Directory.Packages.props:5,7 | ASP.NET Core / EF Core package line | 8.0.2 (published 2024-02-13; latest 8.0.x is 8.0.33, published 2026-10-25) |
| src/Web/Dockerfile:10,20 | build / runtime images | `mcr.microsoft.com/dotnet/sdk:8.0`, `aspnet:8.0` |
| src/PublicApi/Dockerfile:3,8 | runtime / build images | `aspnet:8.0`, `sdk:8.0` |
| .github/workflows/dotnetcore.yml:15-16 | CI SDK | `8.0.x`, `include-prerelease: true` |
| .devcontainer/devcontainer.json:5 | dev container image | `mcr.microsoft.com/devcontainers/dotnet:8.0` |
| infra/main.bicep:56-57 | App Service runtime | `dotnetcore` `8.0` |
| src/Web/.config/dotnet-tools.json | dotnet-ef tool | 8.0.0 |

Support dates: .NET 8 (LTS) and .NET 9 (STS) both end support on 10 November 2026. .NET 10 is the current LTS (endoflife.date: supported until 14 November 2028). Twelve days separate the date of this report (29 October 2026) from the end of .NET 8 support.

## CI (GitHub Actions)

| Workflow | Trigger | Steps | Notes |
|---|---|---|---|
| `dotnetcore.yml` "eShopOnWeb Build and Test" | push, pull_request, workflow_dispatch | `actions/checkout@v2`, `actions/setup-dotnet@v1`, `dotnet build ./eShopOnWeb.sln -c Release`, `dotnet test ./eShopOnWeb.sln -c Release` | Both actions declare `using: node12` in their action.yml. Current majors are checkout v7 and setup-dotnet v6. No lint, format, vulnerability, secret or licence scan; no coverage collection or gate; no container build; no deployment |
| `richnav.yml` "Code Index" | manual only | build `Everything.sln` on Windows, then code-navigation indexer | Not a quality gate |
| `dependabot.yml` | daily | NuGet ecosystem, `/` | No entries for GitHub Actions, Docker or LibMan/npm assets. The repository is archived, so update pull requests can no longer land |

We could not see CI run history from our environment, so we did not establish whether the last runs were green.

## Containers

| File | Observation |
|---|---|
| src/Web/Dockerfile:13-18 | `COPY *.sln .` then `COPY . .` before `dotnet restore`, so every source change invalidates the restore layer, and the whole repository (including `src/Web/key-*.xml`) enters the build context |
| src/PublicApi/Dockerfile:5-6 | `EXPOSE 80` / `EXPOSE 443`. .NET 8 `aspnet` images listen on 8080 by default, which compose sets explicitly (`docker-compose.override.yml:6,15`) |
| both Dockerfiles | No `USER` instruction, so the process runs as root; .NET 8 images ship a non-root `app` user. No `HEALTHCHECK` |
| .dockerignore:7-8 | `*/bin` and `*/obj` match one directory level only (for example `src/bin`), not `src/Web/bin` |
| docker-compose.yml:1 | `version: '3.4'` (obsolete key in current Compose) |
| docker-compose.yml:19-24 | `mcr.microsoft.com/azure-sql-edge`, retired 30 September 2025 (Microsoft Learn); SA password in clear text; port 1433 published on the host |
| docker-compose.override.yml:5,14 | `ASPNETCORE_ENVIRONMENT=Docker`. `src/Web/Program.cs:25,169` treats `Docker` like `Development` (developer exception page, service listing, migrations endpoint, WebAssembly debugging) |

## Infrastructure as code (Bicep, Azure Developer CLI)

| File | Observation |
|---|---|
| azure.yaml:4-8 | One service, `web` (`./src/Web`). PublicApi is not deployed |
| infra/main.bicep:48-65 | One App Service (`web`); app settings point at Key Vault for the two connection strings |
| infra/main.bicep:121-132 | App Service plan SKU `B1` |
| infra/main.bicep:77-106 | Two separate SQL logical servers, one database each (catalogue, identity) |
| infra/core/database/sqlserver/sqlserver.bicep:28-31 | Database resource without `sku`, so the tier is whatever the platform default is at deployment time (not stated in the template reference) |
| sqlserver.bicep:21-23, 33-42 | `publicNetworkAccess: 'Enabled'`; firewall rule `0.0.0.1`–`255.255.255.254` ("Allow all clients", with a comment that it is for developer debugging) |
| sqlserver.bicep:45-95 | Deployment script (Azure CLI 2.37.0) downloads `sqlcmd` v0.8.1 from the internet and runs it with the SQL admin password; it adds the app user to `db_owner` |
| infra/core/host/appservice.bicep:6, 73 | Application Insights is optional and not passed by main.bicep, so no telemetry is wired |
| appservice.bicep:77-88 | Application logs to the file system at `Verbose`; detailed error messages and failed-request tracing on; HTTP logs kept 1 day / 35 MB |
| appservice.bicep:49, 61 | `minTlsVersion: '1.2'`, `httpsOnly: true` (good) |
| infra/core/security/keyvault.bicep | Access-policy model (not RBAC); secrets get/list for the app identity and the deploying principal |
