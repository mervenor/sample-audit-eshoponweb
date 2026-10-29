# Front-end libraries

The storefront layout loads different library versions depending on the ASP.NET Core environment. Development (and the `Docker` environment) uses files restored by LibMan (`src/Web/libman.json`). Staging and Production load other versions from public CDNs (`src/Web/Views/Shared/_Layout.cshtml`, `src/Web/Views/Shared/_ValidationScriptsPartial.cshtml`).

| Library | Development (libman.json) | Staging / Production (CDN) | Advisories at the production version (OSV.dev, 2026-10-28) | Advisories at the development version | Line status |
|---|---|---|---|---|---|
| jQuery | 3.6.3 (2022-12-20) | 2.2.0 (2016-01-08), `_Layout.cshtml:58`, no SRI | 4: CVE-2015-9251, CVE-2019-11358, CVE-2020-11022, CVE-2020-11023 (all Moderate) | 0 | 2.x end of life; current 4.0.0 |
| Bootstrap JS | 3.4.1 (2019-02-13) | 3.3.6 (2015-11-24), `_Layout.cshtml:62`, no SRI | 7 XSS advisories: CVE-2016-10735, CVE-2018-14040, CVE-2018-14042, CVE-2018-20676, CVE-2018-20677, CVE-2019-8331, CVE-2024-6485 (all Moderate) | 2: CVE-2024-6485, CVE-2025-1647 | 3.x end of life 2019-07-24; current 5.3.8 |
| Bootstrap CSS | 3.4.1 | 4.0.0-alpha.5 (a 2016 pre-release), `_Layout.cshtml:20`, no SRI | CSS only; the production CSS (4.x pre-release) and JS (3.x) belong to different major versions | – | 4.x end of life 2023-01-01 |
| jquery-validation | 1.19.5 (2022-07-01) | 1.14.0 (2015-06-30), with SRI, `_ValidationScriptsPartial.cshtml:6` | 4: CVE-2021-21252 (High), CVE-2022-31147 (High), CVE-2021-43306 (Low), CVE-2025-3573 (Moderate) | 1: CVE-2025-3573 | current 1.22.1 |
| jquery-validation-unobtrusive | 4.0.0 | 3.2.6, with SRI | 0 | 0 | – |
| Identity area validation | – | jquery-validation 1.17.0, with SRI (`Areas/Identity/Pages/_ValidationScriptsPartial.cshtml:6`) | the same 4 as 1.14.0 | – | – |
| Bootstrap (Blazor admin host page) | 4.3.1 local copy | 4.3.1 from a CDN, with SRI (`Pages/Admin/Index.cshtml:23`) | 0 (CVE-2024-6531 was withdrawn) | 0 | 4.x end of life 2023-01-01 |
| toastr | 2.1.4 | – | – | – | Restored but not referenced by any view or script |
| @aspnet/signalr | 1.0.27 | – | – | – | Deprecated on npm ("no longer supported"); restored but not referenced |

## Fallback paths do not match where LibMan puts the files (static reading; confirm in a browser)

- `libman.json` restores jQuery to `wwwroot/lib/jquery/` with files at the top level (the cdnjs layout for jquery 3.6.3 is `jquery.js`, `jquery.min.js`, ...). The production fallback is `~/lib/jquery/dist/jquery.min.js` (`_Layout.cshtml:59`). The development reference `~/lib/jquery/jquery.js` (`_Layout.cshtml:53`) matches.
- `libman.json` restores jquery-validate to `wwwroot/lib/jquery-validate/`. Both the development reference (`~/lib/jquery-validation/dist/jquery.validate.js`, `_ValidationScriptsPartial.cshtml:2`) and the production fallback (`_ValidationScriptsPartial.cshtml:7`) point at `lib/jquery-validation/dist/`.

From reading alone, we expect the jQuery fallback, and the main-site client-side validation script in development, to return 404. A network check on a running site would confirm it.

## Other observations

- `_Layout.cshtml:2` and `Pages/Admin/Index.cshtml:13` declare `<html>` without a `lang` attribute.
- `wwwroot/lib/` is git-ignored (`.gitignore:28`), so local library files exist only after a LibMan restore.
