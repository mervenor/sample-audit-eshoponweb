# Transitive NuGet packages with advisories (indicative)

Method: starting from the 49 referenced direct packages (the unreferenced `Microsoft.AspNetCore.Mvc` excluded), we walked each package's net8.0 (or nearest compatible) dependency group from NuGet registration metadata, taking the lowest version that satisfies the declared ranges, then queried OSV.dev for each resolved package. This approximates, but is not, a NuGet restore: real resolution can pick higher versions, and the shared framework can supersede some packages at run time. Treat this list as a prompt for `dotnet list package --vulnerable --include-transitive`, not as a result of it.

Transitive packages resolved: 277. Packages with at least one advisory: 9.

| Package | Approx. version | Brought in by | Advisory | Severity | Summary |
|---|---|---|---|---|---|
| AutoMapper | 12.0.1 | AutoMapper.Extensions.Microsoft.DependencyInjection 12.0.1 | GHSA-rvv3-g6hj-g44x (CVE-2026-32933) | HIGH | AutoMapper Vulnerable to Denial of Service (DoS) via Uncontrolled Recursion |
| Microsoft.Extensions.Caching.Memory | 8.0.0 | Microsoft.EntityFrameworkCore 8.0.2 | GHSA-qj66-m88j-hmgj (CVE-2024-43483) | HIGH | Microsoft Security Advisory CVE-2024-43483 | .NET Denial of Service Vulnerability |
| Microsoft.Identity.Client | 4.56.0 | Azure.Identity 1.10.4 | GHSA-m5vv-6r4h-3vj9 (CVE-2024-35255) | MODERATE | Azure Identity Libraries and Microsoft Authentication Library Elevation of Privilege Vulnerability |
| Microsoft.Identity.Client | 4.56.0 | Azure.Identity 1.10.4 | GHSA-x674-v45j-fwxw (CVE-2024-27086) | LOW | MSAL.NET applications targeting Xamarin Android and .NET Android (MAUI) susceptible to local denial of service |
| NuGet.Common | 6.3.1 | NuGet.Configuration 6.3.1 | GHSA-6qmf-mmc7-6c2p (CVE-2023-29337) | HIGH | NuGet Client Remote Code Execution Vulnerability |
| NuGet.Packaging | 6.3.1 | NuGet.Protocol 6.3.1 | GHSA-68w7-72jg-6qpp (CVE-2024-0057) | CRITICAL | NuGet Client Security Feature Bypass Vulnerability |
| NuGet.Protocol | 6.3.1 | NuGet.DependencyResolver.Core 6.3.1 | GHSA-6qmf-mmc7-6c2p (CVE-2023-29337) | HIGH | NuGet Client Remote Code Execution Vulnerability |
| System.Formats.Asn1 | 5.0.0 | System.Security.Cryptography.Pkcs 5.0.0 | GHSA-447r-wph3-92pm (CVE-2024-38095) | HIGH | Microsoft Security Advisory CVE-2024-38095 | .NET Denial of Service Vulnerability |
| System.Net.Http | 4.3.0 | NETStandard.Library 1.6.1 | GHSA-7jgj-8wvc-jh57 (CVE-2018-8292) | HIGH | .NET Core Information Disclosure |
| System.Text.RegularExpressions | 4.3.0 | NETStandard.Library 1.6.1 | GHSA-cmhx-cq75-c4mj (CVE-2019-0820) | HIGH | Regular Expression Denial of Service in System.Text.RegularExpressions |

Reading the list:
- Run-time relevant: `AutoMapper` 12.0.1 (via the PublicApi's AutoMapper DI package), `Microsoft.Extensions.Caching.Memory` 8.0.0 (via EF Core 8.0.2), `Microsoft.Identity.Client` 4.56.0 (via Azure.Identity 1.10.4).
- Design-time tooling: the `NuGet.*` 6.3.1 and `System.Security.Cryptography.Pkcs` / `System.Formats.Asn1` entries arrive through `Microsoft.VisualStudio.Web.CodeGeneration.Design` (scaffolding), and `System.Net.Http` 4.3.0 / `System.Text.RegularExpressions` 4.3.0 through `BuildBundlerMinifier`'s NETStandard.Library 1.6.1 (Release builds of Web only). They matter less at run time. `Microsoft.VisualStudio.Web.CodeGeneration.Design` is referenced by Web.csproj:25 and PublicApi.csproj:29 without `PrivateAssets`, so its graph is part of the projects' dependency set and will be flagged by any scanner.

Full resolved list: transitive-resolved.tsv. Raw advisory data: transitive-vulnerable.json.
