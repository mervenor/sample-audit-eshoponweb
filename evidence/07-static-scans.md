# Static scans (read-only text searches) at commit 4da8212117e87d808d4bbc7da6286fd2147ce606

## <img> elements without an alt attribute

src/BlazorAdmin/Pages/CatalogItemPage/Create.razor:31: <img class="col-md-6 esh-picture" src="@LoadPicture">
src/BlazorAdmin/Pages/CatalogItemPage/Delete.razor:29: <img class="col-md-6 esh-picture" src="@($"{_item.PictureUri}")">
src/BlazorAdmin/Pages/CatalogItemPage/Details.razor:30: <img class="col-md-6 esh-picture" src="@($"{_item.PictureUri}")">
src/BlazorAdmin/Pages/CatalogItemPage/Edit.razor:32: <img class="col-md-6 esh-picture" src="@LoadPicture">
src/BlazorAdmin/Pages/CatalogItemPage/List.razor:41: <img class="img-thumbnail" src="@($"{item.PictureUri}")">
src/Web/Pages/Admin/EditCatalogItem.cshtml:10: <img class="esh-catalog-thumbnail" src="@Model.CatalogModel.PictureUri" />
src/Web/Pages/Basket/Checkout.cshtml:8: <img class="esh-catalog-title" src="~/images/main_banner_text.png" />
src/Web/Pages/Basket/Checkout.cshtml:33: <img class="esh-basket-image" src="@item.PictureUrl" />
src/Web/Pages/Basket/Index.cshtml:8: <img class="esh-catalog-title" src="~/images/main_banner_text.png" />
src/Web/Pages/Basket/Index.cshtml:33: <img class="esh-basket-image" src="@item.PictureUrl" />
src/Web/Pages/Basket/Success.cshtml:9: <img class="esh-catalog-title" src="~/images/main_banner_text.png" />
src/Web/Pages/Index.cshtml:8: <img class="esh-catalog-title" src="~/images/main_banner_text.png" />
src/Web/Pages/Shared/Components/BasketComponent/Default.cshtml:8: <img src="~/images/cart.png" />
src/Web/Pages/Shared/_editCatalog.cshtml:5: <img class="esh-catalog-thumbnail" src="@Model.PictureUri" />
src/Web/Pages/Shared/_product.cshtml:4: <img class="esh-catalog-thumbnail" src="@Model.PictureUri" />
src/Web/Views/Order/Detail.cshtml:55: <img class="esh-orders-detail-image" src="@item.PictureUrl">
src/Web/Views/Shared/Components/Basket/Default.cshtml:9: <img src="~/images/cart.png" />
src/Web/Views/Shared/_LoginPartial.cshtml:8: <img class="esh-identity-image" src="~/images/arrow-down.png">
src/Web/Views/Shared/_LoginPartial.cshtml:30: <img class="esh-identity-image" src="~/images/logout.png">

(19 matches)

## All <img> elements

src/BlazorAdmin/Pages/CatalogItemPage/Create.razor:31: <img class="col-md-6 esh-picture" src="@LoadPicture">
src/BlazorAdmin/Pages/CatalogItemPage/Delete.razor:29: <img class="col-md-6 esh-picture" src="@($"{_item.PictureUri}")">
src/BlazorAdmin/Pages/CatalogItemPage/Details.razor:30: <img class="col-md-6 esh-picture" src="@($"{_item.PictureUri}")">
src/BlazorAdmin/Pages/CatalogItemPage/Edit.razor:32: <img class="col-md-6 esh-picture" src="@LoadPicture">
src/BlazorAdmin/Pages/CatalogItemPage/List.razor:41: <img class="img-thumbnail" src="@($"{item.PictureUri}")">
src/Web/Pages/Admin/EditCatalogItem.cshtml:10: <img class="esh-catalog-thumbnail" src="@Model.CatalogModel.PictureUri" />
src/Web/Pages/Basket/Checkout.cshtml:8: <img class="esh-catalog-title" src="~/images/main_banner_text.png" />
src/Web/Pages/Basket/Checkout.cshtml:33: <img class="esh-basket-image" src="@item.PictureUrl" />
src/Web/Pages/Basket/Index.cshtml:8: <img class="esh-catalog-title" src="~/images/main_banner_text.png" />
src/Web/Pages/Basket/Index.cshtml:33: <img class="esh-basket-image" src="@item.PictureUrl" />
src/Web/Pages/Basket/Success.cshtml:9: <img class="esh-catalog-title" src="~/images/main_banner_text.png" />
src/Web/Pages/Index.cshtml:8: <img class="esh-catalog-title" src="~/images/main_banner_text.png" />
src/Web/Pages/Shared/Components/BasketComponent/Default.cshtml:8: <img src="~/images/cart.png" />
src/Web/Pages/Shared/_editCatalog.cshtml:5: <img class="esh-catalog-thumbnail" src="@Model.PictureUri" />
src/Web/Pages/Shared/_product.cshtml:4: <img class="esh-catalog-thumbnail" src="@Model.PictureUri" />
src/Web/Views/Order/Detail.cshtml:55: <img class="esh-orders-detail-image" src="@item.PictureUrl">
src/Web/Views/Shared/Components/Basket/Default.cshtml:9: <img src="~/images/cart.png" />
src/Web/Views/Shared/_Layout.cshtml:33: <img src="~/images/brand.png" alt="eShop On Web" />
src/Web/Views/Shared/_LoginPartial.cshtml:8: <img class="esh-identity-image" src="~/images/arrow-down.png">
src/Web/Views/Shared/_LoginPartial.cshtml:30: <img class="esh-identity-image" src="~/images/logout.png">

(20 matches)

## <input type="image"> without alt

src/Web/Pages/Index.cshtml:20: <input class="esh-catalog-send" type="image" src="~/images/arrow-right.svg" />

(1 matches)

## <html> elements (lang attribute check)

src/Web/Pages/Admin/Index.cshtml:13: <html>
src/Web/Views/Shared/_Layout.cshtml:2: <html>

(2 matches)

## Viewport settings that disable zoom

src/Web/Pages/Admin/Index.cshtml:16: <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />

(1 matches)

## Third-party CDN references

src/Web/Areas/Identity/Pages/_ValidationScriptsPartial.cshtml:6: <script src="https://ajax.aspnetcdn.com/ajax/jquery.validate/1.17.0/jquery.validate.min.js"
src/Web/Areas/Identity/Pages/_ValidationScriptsPartial.cshtml:12: <script src="https://ajax.aspnetcdn.com/ajax/jquery.validation.unobtrusive/3.2.9/jquery.validate.unobtrusive.min.js"
src/Web/Pages/Admin/Index.cshtml:23: <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css"
src/Web/Views/Shared/_Layout.cshtml:20: <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.0.0-alpha.5/css/bootstrap.min.css"
src/Web/Views/Shared/_Layout.cshtml:58: <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-2.2.0.min.js"
src/Web/Views/Shared/_Layout.cshtml:62: <script src="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.6/bootstrap.min.js"
src/Web/Views/Shared/_ValidationScriptsPartial.cshtml:6: <script src="https://ajax.aspnetcdn.com/ajax/jquery.validate/1.14.0/jquery.validate.min.js"
src/Web/Views/Shared/_ValidationScriptsPartial.cshtml:12: <script src="https://ajax.aspnetcdn.com/ajax/jquery.validation.unobtrusive/3.2.6/jquery.validate.unobtrusive.min.js"

(8 matches)

## Subresource Integrity attributes

src/Web/Areas/Identity/Pages/_ValidationScriptsPartial.cshtml:10: integrity="sha384-rZfj/ogBloos6wzLGpPkkOr/gpkBNLZ6b6yLy4o+ok+t/SAKlL5mvXLr0OXNi1Hp">
src/Web/Areas/Identity/Pages/_ValidationScriptsPartial.cshtml:16: integrity="sha384-ifv0TYDWxBHzvAk2Z0n8R434FL1Rlv/Av18DXE43N/1rvHyOG4izKst0f2iSLdds">
src/Web/Pages/Admin/Index.cshtml:27: integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" />
src/Web/Views/Shared/_ValidationScriptsPartial.cshtml:10: integrity="sha384-Fnqn3nxp3506LP/7Y3j/25BlWeA3PXTyT1l78LjECcPaKCV12TsZP7yyMxOe/G/k">
src/Web/Views/Shared/_ValidationScriptsPartial.cshtml:16: integrity="sha384-JrXK+k53HACyavUKOsL+NkmSesD2P+73eDMrbTtTk0h4RmOF8hF8apPlkp26JlyH">

(5 matches)

## TODO / HACK / FIXME comments

src/ApplicationCore/Constants/AuthorizationConstants.cs:7: // TODO: Don't use this in production
src/ApplicationCore/Constants/AuthorizationConstants.cs:10: // TODO: Change this to an environment variable
src/BlazorAdmin/CustomAuthStateProvider.cs:15: // TODO: Get Default Cache Duration from Config
src/BlazorAdmin/Pages/CatalogItemPage/Delete.razor:110: // TODO: Add some kind of "are you sure" check before this
src/Infrastructure/Services/EmailSender.cs:12: // TODO: Wire this up to actual email sending logic via SendGrid, local SMTP, etc.
src/Web/Areas/Identity/Pages/Account/Logout.cshtml.cs:13: //TODO : replace IMemoryCache by distributed cache if you are in multi-host scenario
src/Web/Configuration/ConfigureCookieSettings.cs:18: //TODO need to check that.
src/Web/Configuration/RevokeAuthenticationEvents.cs:11: //TODO : replace IMemoryCache with a distributed cache if you are in multi-host scenario

(8 matches)

## Raw SQL or raw HTML output (injection review)


(0 matches)

## [AllowAnonymous]

src/Web/Areas/Identity/Pages/Account/ConfirmEmail.cshtml.cs:13: [AllowAnonymous]
src/Web/Areas/Identity/Pages/Account/Login.cshtml.cs:13: [AllowAnonymous]
src/Web/Areas/Identity/Pages/Account/Register.cshtml.cs:17: [AllowAnonymous]
src/Web/Controllers/UserController.cs:37: [AllowAnonymous]
src/Web/Controllers/UserController.cs:44: [AllowAnonymous]

(5 matches)

## new HttpClient()

src/BlazorAdmin/Program.cs:23: builder.Services.AddScoped(sp => new HttpClient() { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });
src/Web/HealthChecks/ApiHealthCheck.cs:24: var client = new HttpClient();
src/Web/HealthChecks/HomePageHealthCheck.cs:25: var client = new HttpClient();
src/Web/Program.cs:102: builder.Services.AddScoped<HttpClient>(s => new HttpClient

(4 matches)

## async void

src/BlazorAdmin/Pages/CatalogItemPage/List.razor.cs:43: private async void DetailsClick(int id)

(1 matches)

## Task.Delay

src/PublicApi/CatalogItemEndpoints/CatalogItemListPagedEndpoint.cs:42: await Task.Delay(1000);

(1 matches)

## In-process memory cache use

src/Web/Areas/Identity/Pages/Account/Logout.cshtml.cs:13: //TODO : replace IMemoryCache by distributed cache if you are in multi-host scenario
src/Web/Areas/Identity/Pages/Account/Logout.cshtml.cs:18: private readonly IMemoryCache _cache;
src/Web/Areas/Identity/Pages/Account/Logout.cshtml.cs:20: public LogoutModel(SignInManager<ApplicationUser> signInManager, ILogger<LogoutModel> logger, IMemoryCache cache)
src/Web/Configuration/RevokeAuthenticationEvents.cs:11: //TODO : replace IMemoryCache with a distributed cache if you are in multi-host scenario
src/Web/Configuration/RevokeAuthenticationEvents.cs:14: private readonly IMemoryCache _cache;
src/Web/Configuration/RevokeAuthenticationEvents.cs:17: public RevokeAuthenticationEvents(IMemoryCache cache, ILogger<RevokeAuthenticationEvents> logger)
src/Web/Controllers/UserController.cs:22: private readonly IMemoryCache _cache;
src/Web/Controllers/UserController.cs:27: IMemoryCache cache)
src/Web/Services/CachedCatalogViewModelService.cs:12: private readonly IMemoryCache _cache;
src/Web/Services/CachedCatalogViewModelService.cs:15: public CachedCatalogViewModelService(IMemoryCache cache,

(10 matches)

## Security headers, rate limiting, data protection, telemetry (expect none)


(0 matches)

## Identity options configured in code (expect none)


(0 matches)

## Concurrency tokens (expect none)


(0 matches)

## Projects with <Nullable>enable

src/ApplicationCore/ApplicationCore.csproj:5: <Nullable>enable</Nullable>
src/Infrastructure/Infrastructure.csproj:5: <Nullable>enable</Nullable>
src/PublicApi/PublicApi.csproj:8: <Nullable>enable</Nullable>
src/Web/Web.csproj:4: <Nullable>enable</Nullable>
tests/FunctionalTests/FunctionalTests.csproj:6: <Nullable>enable</Nullable>
tests/PublicApiIntegrationTests/PublicApiIntegrationTests.csproj:4: <Nullable>enable</Nullable>
tests/UnitTests/UnitTests.csproj:4: <Nullable>enable</Nullable>

(7 matches)

## Projects without <Nullable>enable

src/BlazorAdmin/BlazorAdmin.csproj
src/BlazorShared/BlazorShared.csproj
tests/IntegrationTests/IntegrationTests.csproj

## Declared central package versions not referenced by any project

Microsoft.AspNetCore.Mvc

## Files whose names contain a space

src/BlazorAdmin/Services/CachedCatalogLookupDataServiceDecorator .cs

## Lock files (packages.lock.json)

none found

## Repository documents

README.md: .devcontainer/devcontainerreadme.md, README.md, src/BlazorAdmin/wwwroot/css/open-iconic/README.md, src/PublicApi/README.md
SECURITY.md: none
CONTRIBUTING.md: none
CODEOWNERS: none
docs/: none
adr: src/ApplicationCore/Interfaces/IReadRepository.cs
runbook: none

## Image weights

product images: 13 files, 1956024 bytes in total, largest 231538 bytes
src/Web/wwwroot/images/main_banner.png: 729906 bytes
src/Web/wwwroot/images/main_banner_text.png: 8828 bytes
src/Web/wwwroot/images/brand.png: 5228 bytes
src/Web/wwwroot/images/cart.png: 1532 bytes
