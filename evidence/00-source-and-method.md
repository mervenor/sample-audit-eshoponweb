# Source and method

## Subject

| Item | Value |
|---|---|
| Repository | https://github.com/dotnet-architecture/eShopOnWeb (MIT License, Copyright (c) .NET Foundation and Contributors) |
| Branch | `main` (remote `HEAD` and `refs/heads/main` both resolve to the commit below) |
| Commit | `4da8212117e87d808d4bbc7da6286fd2147ce606` |
| Commit date | 2025-01-13T12:55:18-08:00 ("Update README.md") |
| Repository status | Archived by its owner on 2025-01-13; read-only (GitHub repository page, read 2026-10-28) |
| Last change to `src/`, `tests/` or `Directory.Packages.props` | 2024-04-01 (dependency-update merges) |
| History | 825 commits from 2017-03-21; 92 distinct author e-mail addresses; commits per year 2020: 73, 2021: 106, 2022: 119, 2023: 150, 2024: 42, 2025: 1 |
| Upstream note | README.md:3 says active development moved to a newer sample; README.md:27 says the sample "is not meant to be an eCommerce reference application" |

## How the source was fetched

- `git clone --depth 1 --no-tags` into an isolated scratch folder, used for reading only.
- A second, blobless bare clone (`--filter=blob:none`) was used only to read commit metadata (counts, dates, which commits touched which paths). No file contents from history were read.
- The cloned tree is not part of this deliverable. Short excerpts, with line numbers, are in `02-code-excerpts.md`.

## What we did not do

We did not build, restore, install, run or execute any of the project's code, scripts, Dockerfiles, compose files, workflows or infrastructure templates, and we did not install a .NET SDK. Everything here comes from reading files and from public package and advisory metadata.

## Tools and sources

| Purpose | Tool or source |
|---|---|
| Fetch | git 2.43.0 |
| Parsing `*.csproj`, `Directory.Packages.props`, `global.json`, YAML, Bicep, Razor | Reading, plus small Python 3.13 standard-library scripts (no project code imported or executed) |
| Line counts | Python non-blank line counter over `git ls-files`; pygount 3.2.0 for a second, code-only count |
| Package versions, publish dates, deprecation | NuGet V3 API (flat container, registration), queried 2026-10-28 |
| Vulnerabilities | NuGet vulnerability database (`api.nuget.org/v3/vulnerabilities`); OSV.dev API, which carries the GitHub Advisory Database records (github.com advisory pages were not reachable from our environment) |
| Front-end libraries | npm registry (release dates, latest versions), cdnjs API (file layout), OSV.dev |
| End-of-life dates | .NET: 8 and 9 end support 10 November 2026; 10 is the current LTS (endoflife.date lists 14 November 2028). Bootstrap and jQuery: endoflife.date |
| Framework behaviour | ASP.NET Core `release/8.0` source on GitHub (cookie builder defaults, events-type resolution, Identity cookie registration, lockout defaults, WebAssembly debug endpoint) |
| Other references | OWASP Top 10:2025; Microsoft Learn pages for `global.json` (`sdk.version` takes no wildcards) and Azure SQL Edge (retired 30 September 2025) |
| PDF | pandoc 3.1.3 with wkhtmltopdf 0.12.6 |

## AI assistance

An AI assistant read the source, ran the metadata lookups listed above and drafted the report. Every finding cites the files and lines it rests on, so a reader can check it. In a client engagement, a Mervenor engineer re-checks every finding before delivery.
