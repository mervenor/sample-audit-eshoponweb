# Sample Perlustra audit: eShopOnWeb

This is a **sample** of Perlustra, Mervenor's fixed-scope codebase health audit. It applies our audit template to a public open-source application, so you can see what a report looks like before you commission one for your own code.

- **Subject:** [`dotnet-architecture/eShopOnWeb`](https://github.com/dotnet-architecture/eShopOnWeb), Microsoft's ASP.NET Core reference and learning application (MIT License).
- **Pinned commit:** `4da8212117e87d808d4bbc7da6286fd2147ce606` (2025-01-13, the head of `main`; the repository has been archived since that date).
- **Framing:** eShopOnWeb was written to teach, and its README says it is not meant to be a real shop. We review it as if it were a production application you had just inherited, which is the situation our clients are in. The findings are factual and cite their evidence; they are not a criticism of a teaching sample.

## A desk review

We read the source and configuration and checked public package and advisory metadata. We did **not** build, restore, install, run or execute any of the project's code, scripts, containers or infrastructure templates. Whether the build compiles, whether the tests pass, how the application behaves at run time and how fast it is all need a running build. Section 8 of the report lists exactly what that leaves unverified.

## What is here

| Path | Contents |
|---|---|
| `report.md` | The audit report: executive summary and overall rating, system overview, scorecard, 24 findings, risk register, quick wins, a sprint roadmap sized in engineering days, and appendices |
| `report.pdf` | The same report rendered to PDF |
| `evidence/` | Trimmed evidence: method and sources, inventory and counts, code excerpts with original line numbers, package and advisory lists, front-end library versions, runtime, CI and container notes, static scans |
| `LICENSE.md` | Licence terms for this sample and for the code it quotes |

The cloned eShopOnWeb source is not included. Code excerpts in `evidence/02-code-excerpts.md` quote only the lines each finding cites, with secrets redacted.

## How it was produced

1. Shallow, read-only clone of the pinned commit; a blobless clone for commit metadata only.
2. Parsed `global.json`, the `*.csproj` files, `Directory.Packages.props`, Dockerfiles, compose files, GitHub Actions workflows and Bicep templates as text.
3. Compared the 50 declared NuGet packages with the NuGet V3 API and the NuGet vulnerability database. Checked advisories and front-end libraries against OSV.dev (GitHub Advisory Database records) and the npm registry. Approximated the transitive dependency graph from package metadata, without a restore.
4. Counted code and tests with a line counter; reviewed the code by reading it; checked framework behaviour against the ASP.NET Core 8.0 source.
5. Wrote the report in Mervenor's Perlustra template. Effort is sized in engineering days only; this sample carries no prices.

An AI assistant was used to read the source, run the lookups and draft the text. Every finding cites the files and lines it rests on, so you can check it yourself.

## Want this for your app?

Mervenor — hello@mervenor.com — https://mervenor.com

*Microsoft, .NET, ASP.NET, Azure and Blazor are trademarks of the Microsoft group of companies. This sample is independent and is not affiliated with, endorsed by or sponsored by Microsoft or the .NET Foundation.*
