# Code and configuration excerpts

Source: `dotnet-architecture/eShopOnWeb` at commit `4da8212117e87d808d4bbc7da6286fd2147ce606`. Line numbers are those of the file at that commit.
Quoted code remains under the MIT License, Copyright (c) .NET Foundation and Contributors (see ../LICENSE.md).
Secrets, demo account names and a placeholder postal address are redacted; they are visible in the public repository, but we do not repeat them.

## `src/Web/Controllers/OrderController.cs` (F-01)

The user name is passed to the query...

Lines 31–43:

```csharp
  31      [HttpGet("{orderId}")]
  32      public async Task<IActionResult> Detail(int orderId)
  33      {
  34          Guard.Against.Null(User?.Identity?.Name, nameof(User.Identity.Name));
  35          var viewModel = await _mediator.Send(new GetOrderDetails(User.Identity.Name, orderId));
  36  
  37          if (viewModel == null)
  38          {
  39              return BadRequest("No such order found for this user.");
  40          }
  41  
  42          return View(viewModel);
  43      }
```

## `src/Web/Features/OrderDetails/GetOrderDetailsHandler.cs` (F-01)

...but the handler only uses the order id.

Lines 18–27:

```csharp
  18      public async Task<OrderDetailViewModel?> Handle(GetOrderDetails request,
  19          CancellationToken cancellationToken)
  20      {
  21          var spec = new OrderWithItemsByIdSpec(request.OrderId);
  22          var order = await _orderRepository.FirstOrDefaultAsync(spec, cancellationToken);
  23  
  24          if (order == null)
  25          {
  26              return null;
  27          }
```

## `src/ApplicationCore/Specifications/OrderWithItemsByIdSpec.cs` (F-01)

The specification filters by id alone.

Lines 6–15:

```csharp
   6  public class OrderWithItemsByIdSpec : Specification<Order>
   7  {
   8      public OrderWithItemsByIdSpec(int orderId)
   9      {
  10          Query
  11              .Where(order => order.Id == orderId)
  12              .Include(o => o.OrderItems)
  13              .ThenInclude(i => i.ItemOrdered);
  14      }
  15  }
```

## `src/ApplicationCore/Specifications/CustomerOrdersSpecification.cs` (F-01)

Contrast: the order list is filtered by buyer.

Lines 6–13:

```csharp
   6  public class CustomerOrdersSpecification : Specification<Order>
   7  {
   8      public CustomerOrdersSpecification(string buyerId)
   9      {
  10          Query.Where(o => o.BuyerId == buyerId)
  11              .Include(o => o.OrderItems);
  12      }
  13  }
```

## `src/Infrastructure/Data/Migrations/CatalogContextModelSnapshot.cs` (F-01)

Order ids are sequential identity integers.

Lines 152–158:

```csharp
 152              modelBuilder.Entity("Microsoft.eShopWeb.ApplicationCore.Entities.OrderAggregate.Order", b =>
 153                  {
 154                      b.Property<int>("Id")
 155                          .ValueGeneratedOnAdd()
 156                          .HasColumnType("int");
 157  
 158                      SqlServerPropertyBuilderExtensions.UseIdentityColumn(b.Property<int>("Id"), 1L, 1);
```

## `tests/UnitTests/MediatorHandlers/OrdersTests/GetOrderDetails.cs` (F-01, F-16)

The unit test returns an order owned by 'buyerId' to 'SomeUserName' and asserts success.

Lines 18–38:

```csharp
  18      public GetOrderDetails()
  19      {
  20          var item = new OrderItem(new CatalogItemOrdered(1, "ProductName", "URI"), 10.00m, 10);
  21          var address = new Address(/* fixed placeholder address, redacted */);
  22          Order order = new Order("buyerId", address, new List<OrderItem> { item });
  23                  
  24          _mockOrderRepository.FirstOrDefaultAsync(Arg.Any<OrderWithItemsByIdSpec>(), default)
  25              .Returns(order);
  26      }
  27  
  28      [Fact]
  29      public async Task NotBeNullIfOrderExists()
  30      {
  31          var request = new eShopWeb.Web.Features.OrderDetails.GetOrderDetails("SomeUserName", 0);
  32  
  33          var handler = new GetOrderDetailsHandler(_mockOrderRepository);
  34  
  35          var result = await handler.Handle(request, CancellationToken.None);
  36  
  37          Assert.NotNull(result);
  38      }
```

## `src/ApplicationCore/Constants/AuthorizationConstants.cs` (F-02, F-03)

Values redacted in this excerpt; they are in the public repository.

Lines 1–12:

```csharp
   1  namespace Microsoft.eShopWeb.ApplicationCore.Constants;
   2  
   3  public class AuthorizationConstants
   4  {
   5      public const string AUTH_KEY = "[redacted: 44-character literal]";
   6  
   7      // TODO: Don't use this in production
   8      public const string DEFAULT_PASSWORD = "[redacted default password]";
   9  
  10      // TODO: Change this to an environment variable
  11      public const string JWT_SECRET_KEY = "[redacted: 46-character literal]";
  12  }
```

## `src/PublicApi/Program.cs` (F-02, F-16)

Signing key from a constant; issuer and audience not validated; test configuration loaded by the production host.

Lines 28–34:

```csharp
  28  builder.Services.AddEndpoints();
  29  
  30  // Use to force loading of appsettings.json of test project
  31  builder.Configuration.AddConfigurationFile("appsettings.test.json");
  32  builder.Logging.AddConsole();
  33  
  34  Microsoft.eShopWeb.Infrastructure.Dependencies.ConfigureServices(builder.Configuration, builder.Services);
```

Lines 54–70:

```csharp
  54  var key = Encoding.ASCII.GetBytes(AuthorizationConstants.JWT_SECRET_KEY);
  55  builder.Services.AddAuthentication(config =>
  56  {
  57      config.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
  58  })
  59  .AddJwtBearer(config =>
  60  {
  61      config.RequireHttpsMetadata = false;
  62      config.SaveToken = true;
  63      config.TokenValidationParameters = new TokenValidationParameters
  64      {
  65          ValidateIssuerSigningKey = true,
  66          IssuerSigningKey = new SymmetricSecurityKey(key),
  67          ValidateIssuer = false,
  68          ValidateAudience = false
  69      };
  70  });
```

## `src/Infrastructure/Identity/IdentityTokenClaimService.cs` (F-02, F-13)

Tokens signed with the constant key, valid for 7 days.

Lines 23–44:

```csharp
  23      public async Task<string> GetTokenAsync(string userName)
  24      {
  25          var tokenHandler = new JwtSecurityTokenHandler();
  26          var key = Encoding.ASCII.GetBytes(AuthorizationConstants.JWT_SECRET_KEY);
  27          var user = await _userManager.FindByNameAsync(userName);
  28          if (user == null) throw new UserNotFoundException(userName);
  29          var roles = await _userManager.GetRolesAsync(user);
  30          var claims = new List<Claim> { new Claim(ClaimTypes.Name, userName) };
  31  
  32          foreach (var role in roles)
  33          {
  34              claims.Add(new Claim(ClaimTypes.Role, role));
  35          }
  36  
  37          var tokenDescriptor = new SecurityTokenDescriptor
  38          {
  39              Subject = new ClaimsIdentity(claims.ToArray()),
  40              Expires = DateTime.UtcNow.AddDays(7),
  41              SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256Signature)
  42          };
  43          var token = tokenHandler.CreateToken(tokenDescriptor);
  44          return tokenHandler.WriteToken(token);
```

## `tests/PublicApiIntegrationTests/ApiTokenHelper.cs` (F-02)

The test suite mints an administrator token from the public constant, without a password.

Lines 13–48:

```csharp
  13          public static string GetAdminUserToken()
  14          {
  15              string userName = "[admin account]";
  16              string[] roles = { "Administrators" };
  17  
  18              return CreateToken(userName, roles);
  19          }
  20  
  21          public static string GetNormalUserToken()
  22          {
  23              string userName = "[demo-user account]";
  24              string[] roles = { };
  25  
  26              return CreateToken(userName, roles);
  27          }
  28  
  29          private static string CreateToken(string userName, string[] roles)
  30          {
  31              var claims = new List<Claim> { new Claim(ClaimTypes.Name, userName) };
  32  
  33              foreach (var role in roles)
  34              {
  35                  claims.Add(new Claim(ClaimTypes.Role, role));
  36              }
  37  
  38              var key = Encoding.ASCII.GetBytes(AuthorizationConstants.JWT_SECRET_KEY);
  39              var tokenDescriptor = new SecurityTokenDescriptor
  40              {
  41                  Subject = new ClaimsIdentity(claims.ToArray()),
  42                  Expires = DateTime.UtcNow.AddHours(1),
  43                  SigningCredentials = new SigningCredentials(new SymmetricSecurityKey(key), SecurityAlgorithms.HmacSha256Signature)
  44              };
  45              var tokenHandler = new JwtSecurityTokenHandler();
  46              var token = tokenHandler.CreateToken(tokenDescriptor);
  47              return tokenHandler.WriteToken(token);
  48          }
```

## `src/PublicApi/CatalogItemEndpoints/DeleteCatalogItemEndpoint.cs` (F-02, F-07)

Administrator-only delete; no check for baskets that reference the item.

Lines 18–41:

```csharp
  18      public void AddRoute(IEndpointRouteBuilder app)
  19      {
  20          app.MapDelete("api/catalog-items/{catalogItemId}",
  21              [Authorize(Roles = BlazorShared.Authorization.Constants.Roles.ADMINISTRATORS, AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)] async
  22              (int catalogItemId, IRepository<CatalogItem> itemRepository) =>
  23              {
  24                  return await HandleAsync(new DeleteCatalogItemRequest(catalogItemId), itemRepository);
  25              })
  26              .Produces<DeleteCatalogItemResponse>()
  27              .WithTags("CatalogItemEndpoints");
  28      }
  29  
  30      public async Task<IResult> HandleAsync(DeleteCatalogItemRequest request, IRepository<CatalogItem> itemRepository)
  31      {
  32          var response = new DeleteCatalogItemResponse(request.CorrelationId());
  33  
  34          var itemToDelete = await itemRepository.GetByIdAsync(request.CatalogItemId);
  35          if (itemToDelete is null)
  36              return Results.NotFound();
  37  
  38          await itemRepository.DeleteAsync(itemToDelete);
  39  
  40          return Results.Ok(response);
  41      }
```

## `src/Web/key-768c1632-cf7b-41a9-bb7a-bff228ae8fba.xml` (F-02)

Committed data-protection key (expired 2022-03-01); key material redacted here.

Lines 1–16:

```xml
   1  <?xml version="1.0" encoding="utf-8"?>
   2  <key id="768c1632-cf7b-41a9-bb7a-bff228ae8fba" version="1">
   3    <creationDate>2021-12-01T14:37:52.0438755Z</creationDate>
   4    <activationDate>2021-12-01T14:37:52.0246578Z</activationDate>
   5    <expirationDate>2022-03-01T14:37:52.0246578Z</expirationDate>
   6    <descriptor deserializerType="Microsoft.AspNetCore.DataProtection.AuthenticatedEncryption.ConfigurationModel.AuthenticatedEncryptorDescriptorDeserializer, Microsoft.AspNetCore.DataProtection, Version=6.0.0.0, Culture=neutral, PublicKeyToken=adb9793829ddae60">
   7      <descriptor>
   8        <encryption algorithm="AES_256_CBC" />
   9        <validation algorithm="HMACSHA256" />
  10        <masterKey p4:requiresEncryption="true" xmlns:p4="http://schemas.asp.net/2015/03/dataProtection">
  11          <!-- Warning: the key below is in an unencrypted form. -->
  12          <value>[redacted: 64-byte key, base64]</value>
  13        </masterKey>
  14      </descriptor>
  15    </descriptor>
  16  </key>
```

## `docker-compose.yml` (F-02, F-20)

Local database container: retired image, fixed SA password (redacted), port published.

Lines 18–24:

```yaml
  18    sqlserver:
  19      image: mcr.microsoft.com/azure-sql-edge
  20      ports:
  21        - "1433:1433"
  22      environment:
  23        - SA_PASSWORD=[redacted]
  24        - ACCEPT_EULA=Y
```

## `src/Web/appsettings.Docker.json` (F-02)

SA credentials in configuration (redacted).

Lines 1–5:

```json
   1  {
   2    "ConnectionStrings": {
   3      "CatalogConnection": "Server=sqlserver,1433;Integrated Security=true;Initial Catalog=Microsoft.eShopOnWeb.CatalogDb;User Id=sa;Password=[redacted];Trusted_Connection=false;TrustServerCertificate=true;",
   4      "IdentityConnection": "Server=sqlserver,1433;Integrated Security=true;Initial Catalog=Microsoft.eShopOnWeb.Identity;User Id=sa;Password=[redacted];Trusted_Connection=false;TrustServerCertificate=true;"
   5    },
```

## `src/Infrastructure/Identity/AppIdentityDbContextSeed.cs` (F-03, F-08)

Seeded accounts, including an administrator, with the shared default password.

Lines 10–31:

```csharp
  10      public static async Task SeedAsync(AppIdentityDbContext identityDbContext, UserManager<ApplicationUser> userManager, RoleManager<IdentityRole> roleManager)
  11      {
  12  
  13          if (identityDbContext.Database.IsSqlServer())
  14          {
  15              identityDbContext.Database.Migrate();
  16          }
  17  
  18          await roleManager.CreateAsync(new IdentityRole(BlazorShared.Authorization.Constants.Roles.ADMINISTRATORS));
  19  
  20          var defaultUser = new ApplicationUser { UserName = "[demo-user account]", Email = "[demo-user account]" };
  21          await userManager.CreateAsync(defaultUser, AuthorizationConstants.DEFAULT_PASSWORD);
  22  
  23          string adminUserName = "[admin account]";
  24          var adminUser = new ApplicationUser { UserName = adminUserName, Email = adminUserName };
  25          await userManager.CreateAsync(adminUser, AuthorizationConstants.DEFAULT_PASSWORD);
  26          adminUser = await userManager.FindByNameAsync(adminUserName);
  27          if (adminUser != null)
  28          {
  29              await userManager.AddToRoleAsync(adminUser, BlazorShared.Authorization.Constants.Roles.ADMINISTRATORS);
  30          }
  31      }
```

## `src/Web/Program.cs` (F-03, F-06, F-08, F-10, F-11)

Seeding runs in every environment; development middleware also runs in the 'Docker' environment.

Lines 22–43:

```csharp
  22  var builder = WebApplication.CreateBuilder(args);
  23  builder.Logging.AddConsole();
  24  
  25  if (builder.Environment.IsDevelopment() || builder.Environment.EnvironmentName == "Docker"){
  26      // Configure SQL Server (local)
  27      Microsoft.eShopWeb.Infrastructure.Dependencies.ConfigureServices(builder.Configuration, builder.Services);
  28  }
  29  else{
  30      // Configure SQL Server (prod)
  31      var credential = new ChainedTokenCredential(new AzureDeveloperCliCredential(), new DefaultAzureCredential());
  32      builder.Configuration.AddAzureKeyVault(new Uri(builder.Configuration["AZURE_KEY_VAULT_ENDPOINT"] ?? ""), credential);
  33      builder.Services.AddDbContext<CatalogContext>(c =>
  34      {
  35          var connectionString = builder.Configuration[builder.Configuration["AZURE_SQL_CATALOG_CONNECTION_STRING_KEY"] ?? ""];
  36          c.UseSqlServer(connectionString, sqlOptions => sqlOptions.EnableRetryOnFailure());
  37      });
  38      builder.Services.AddDbContext<AppIdentityDbContext>(options =>
  39      {
  40          var connectionString = builder.Configuration[builder.Configuration["AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY"] ?? ""];
  41          options.UseSqlServer(connectionString, sqlOptions => sqlOptions.EnableRetryOnFailure());
  42      });
  43  }
```

Lines 45–58:

```csharp
  45  builder.Services.AddCookieSettings();
  46  
  47  builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
  48      .AddCookie(options =>
  49      {
  50          options.Cookie.HttpOnly = true;
  51          options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
  52          options.Cookie.SameSite = SameSiteMode.Lax;
  53      });
  54  
  55  builder.Services.AddIdentity<ApplicationUser, IdentityRole>()
  56             .AddDefaultUI()
  57             .AddEntityFrameworkStores<AppIdentityDbContext>()
  58                             .AddDefaultTokenProviders();
```

Lines 116–139:

```csharp
 116  var app = builder.Build();
 117  
 118  app.Logger.LogInformation("App created...");
 119  
 120  app.Logger.LogInformation("Seeding Database...");
 121  
 122  using (var scope = app.Services.CreateScope())
 123  {
 124      var scopedProvider = scope.ServiceProvider;
 125      try
 126      {
 127          var catalogContext = scopedProvider.GetRequiredService<CatalogContext>();
 128          await CatalogContextSeed.SeedAsync(catalogContext, app.Logger);
 129  
 130          var userManager = scopedProvider.GetRequiredService<UserManager<ApplicationUser>>();
 131          var roleManager = scopedProvider.GetRequiredService<RoleManager<IdentityRole>>();
 132          var identityContext = scopedProvider.GetRequiredService<AppIdentityDbContext>();
 133          await AppIdentityDbContextSeed.SeedAsync(identityContext, userManager, roleManager);
 134      }
 135      catch (Exception ex)
 136      {
 137          app.Logger.LogError(ex, "An error occurred seeding the DB.");
 138      }
 139  }
```

Lines 141–199:

```csharp
 141  var catalogBaseUrl = builder.Configuration.GetValue(typeof(string), "CatalogBaseUrl") as string;
 142  if (!string.IsNullOrEmpty(catalogBaseUrl))
 143  {
 144      app.Use((context, next) =>
 145      {
 146          context.Request.PathBase = new PathString(catalogBaseUrl);
 147          return next();
 148      });
 149  }
 150  
 151  app.UseHealthChecks("/health",
 152      new HealthCheckOptions
 153      {
 154          ResponseWriter = async (context, report) =>
 155          {
 156              var result = new
 157              {
 158                  status = report.Status.ToString(),
 159                  errors = report.Entries.Select(e => new
 160                  {
 161                      key = e.Key,
 162                      value = Enum.GetName(typeof(HealthStatus), e.Value.Status)
 163                  })
 164              }.ToJson();
 165              context.Response.ContentType = MediaTypeNames.Application.Json;
 166              await context.Response.WriteAsync(result);
 167          }
 168      });
 169  if (app.Environment.IsDevelopment() || app.Environment.EnvironmentName == "Docker")
 170  {
 171      app.Logger.LogInformation("Adding Development middleware...");
 172      app.UseDeveloperExceptionPage();
 173      app.UseShowAllServicesMiddleware();
 174      app.UseMigrationsEndPoint();
 175      app.UseWebAssemblyDebugging();
 176  }
 177  else
 178  {
 179      app.Logger.LogInformation("Adding non-Development middleware...");
 180      app.UseExceptionHandler("/Error");
 181      app.UseHsts();
 182  }
 183  
 184  app.UseHttpsRedirection();
 185  app.UseBlazorFrameworkFiles();
 186  app.UseStaticFiles();
 187  app.UseRouting();
 188  
 189  app.UseCookiePolicy();
 190  app.UseAuthentication();
 191  app.UseAuthorization();
 192  
 193  
 194  app.MapControllerRoute("default", "{controller:slugify=Home}/{action:slugify=Index}/{id?}");
 195  app.MapRazorPages();
 196  app.MapHealthChecks("home_page_health_check", new HealthCheckOptions { Predicate = check => check.Tags.Contains("homePageHealthCheck") });
 197  app.MapHealthChecks("api_health_check", new HealthCheckOptions { Predicate = check => check.Tags.Contains("apiHealthCheck") });
 198  //endpoints.MapBlazorHub("/admin");
 199  app.MapFallbackToFile("index.html");
```

## `src/Web/Areas/Identity/Pages/Account/Login.cshtml` (F-03)

The login page prints the demo credentials (redacted here).

Lines 46–54:

```cshtml
  46                      <p>
  47                          Note that for demo purposes you don't need to register and can login with these credentials:
  48                      </p>
  49                      <p>
  50                          User:     <b>[demo-user account]</b> OR <b>[admin account]</b>
  51                      </p>
  52                      <p>
  53                          Password: <b>[redacted default password]</b>
  54                      </p>
```

## `src/Web/Areas/Identity/Pages/Account/Login.cshtml.cs` (F-13)

The comment says failures do not count towards lockout; the call passes lockoutOnFailure = true.

Lines 72–79:

```csharp
  72          if (ModelState.IsValid)
  73          {
  74              // This doesn't count login failures towards account lockout
  75              // To enable password failures to trigger account lockout, set lockoutOnFailure: true
  76              //var result = await _signInManager.PasswordSignInAsync(Input.Email, Input.Password, Input.RememberMe, lockoutOnFailure: true);
  77              var result = await _signInManager.PasswordSignInAsync(Input!.Email!, Input!.Password!, 
  78                  false, true);
  79  
```

## `Directory.Packages.props` (F-04, F-05, F-14, F-21)

Central package versions.

Lines 1–72:

```xml
   1  <Project>
   2    <PropertyGroup>
   3      <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
   4      <TargetFramework>net8.0</TargetFramework>
   5      <AspNetVersion>8.0.2</AspNetVersion>
   6      <SystemExtensionVersion>8.0.0</SystemExtensionVersion>
   7      <EntityFramworkCoreVersion>8.0.2</EntityFramworkCoreVersion>
   8      <VSCodeGeneratorVersion>8.0.0</VSCodeGeneratorVersion>
   9    </PropertyGroup>
  10    <ItemGroup>
  11      <PackageVersion Include="Ardalis.ApiEndpoints" Version="4.1.0" />
  12      <PackageVersion Include="Ardalis.GuardClauses" Version="4.0.1" />
  13      <PackageVersion Include="Ardalis.Specification.EntityFrameworkCore" Version="7.0.0" />
  14      <PackageVersion Include="Ardalis.Result" Version="7.0.0" />
  15      <PackageVersion Include="Ardalis.Specification" Version="7.0.0" />
  16      <PackageVersion Include="Ardalis.ListStartupServices" Version="1.1.4" />
  17      <PackageVersion Include="Azure.Extensions.AspNetCore.Configuration.Secrets" Version="1.3.1" />
  18      <PackageVersion Include="Azure.Identity" Version="1.10.4" />
  19      <PackageVersion Include="AutoMapper.Extensions.Microsoft.DependencyInjection" Version="12.0.1" />
  20      <PackageVersion Include="BlazorInputFile" Version="0.2.0" />
  21      <PackageVersion Include="Blazored.LocalStorage" Version="4.5.0" />
  22      <PackageVersion Include="BuildBundlerMinifier" Version="3.2.449" PrivateAssets="All" />
  23      <PackageVersion Include="FluentValidation" Version="11.9.0" />
  24      <PackageVersion Include="MediatR" Version="12.0.1" />
  25      <PackageVersion Include="Microsoft.AspNetCore.Components.Authorization" Version="$(AspNetVersion)" />
  26      <PackageVersion Include="Microsoft.AspNetCore.Components.WebAssembly" Version="$(AspNetVersion)" />
  27      <PackageVersion Include="Microsoft.AspNetCore.Components.WebAssembly.Authentication" Version="$(AspNetVersion)" />
  28      <PackageVersion Include="Microsoft.AspNetCore.Components.WebAssembly.DevServer" Version="$(AspNetVersion)" PrivateAssets="all" />
  29      <PackageVersion Include="Microsoft.AspNetCore.Components.WebAssembly.Server" Version="$(AspNetVersion)" />
  30      <PackageVersion Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="$(AspNetVersion)" />
  31      <PackageVersion Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="$(AspNetVersion)" />
  32      <PackageVersion Include="Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore" Version="$(AspNetVersion)" />
  33      <PackageVersion Include="Microsoft.AspNetCore.Identity.UI" Version="$(AspNetVersion)" />
  34      <PackageVersion Include="Microsoft.AspNetCore.Mvc" Version="2.2.0" />
  35      <PackageVersion Include="Microsoft.Extensions.Identity.Core" Version="$(AspNetVersion)" />
  36      <PackageVersion Include="Microsoft.Extensions.Logging.Configuration" Version="$(SystemExtensionVersion)" />
  37      <PackageVersion Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="$(VSCodeGeneratorVersion)" />
  38      <PackageVersion Include="Microsoft.Web.LibraryManager.Build" Version="2.1.175" />
  39      <PackageVersion Include="Microsoft.EntityFrameworkCore.InMemory" Version="$(EntityFramworkCoreVersion)" />
  40      <PackageVersion Include="Microsoft.EntityFrameworkCore.SqlServer" Version="$(EntityFramworkCoreVersion)" />
  41      <PackageVersion Include="Microsoft.EntityFrameworkCore.Tools" Version="$(EntityFramworkCoreVersion)">
  42        <PrivateAssets>all</PrivateAssets>
  43        <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
  44      </PackageVersion>
  45      <PackageVersion Include="Microsoft.VisualStudio.Azure.Containers.Tools.Targets" Version="1.19.6" />
  46      <PackageVersion Include="MinimalApi.Endpoint" Version="1.3.0" />
  47      <PackageVersion Include="NSubstitute" Version="5.1.0" />
  48      <PackageVersion Include="NSubstitute.Analyzers.CSharp" Version="1.0.17" />
  49      <PackageVersion Include="System.Net.Http.Json" Version="$(SystemExtensionVersion)" />
  50      <PackageVersion Include="System.Security.Claims" Version="4.3.0" />
  51      <PackageVersion Include="System.Text.Json" Version="8.0.3" />
  52      <PackageVersion Include="System.IdentityModel.Tokens.Jwt" Version="7.3.1" />
  53      <PackageVersion Include="Swashbuckle.AspNetCore" Version="6.5.0" />
  54      <PackageVersion Include="Swashbuckle.AspNetCore.SwaggerUI" Version="6.5.0" />
  55      <PackageVersion Include="Swashbuckle.AspNetCore.Annotations" Version="6.5.0" />
  56      <!-- Test -->
  57      <PackageVersion Include="Microsoft.AspNetCore.Mvc.Testing" Version="$(AspNetVersion)" />
  58      <PackageVersion Include="Microsoft.NET.Test.Sdk" Version="17.9.0" />
  59      <PackageVersion Include="xunit" Version="2.7.0" />
  60      <PackageVersion Include="xunit.runner.visualstudio" Version="2.5.6">
  61        <PrivateAssets>all</PrivateAssets>
  62        <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
  63      </PackageVersion>
  64      <PackageVersion Include="xunit.runner.console" Version="2.7.0">
  65        <PrivateAssets>all</PrivateAssets>
  66        <IncludeAssets>runtime; build; native; contentfiles; analyzers</IncludeAssets>
  67      </PackageVersion>
  68      <PackageVersion Include="MSTest.TestAdapter" Version="3.2.2" />
  69      <PackageVersion Include="MSTest.TestFramework" Version="3.2.2" />
  70      <PackageVersion Include="coverlet.collector" Version="6.0.2" />
  71    </ItemGroup>
  72  </Project>
```

## `global.json` (F-04, F-20)

'8.0.x' is not a valid sdk version value per the global.json documentation.

Lines 1–6:

```json
   1  {
   2    "sdk": {
   3      "version": "8.0.x",
   4      "rollForward": "latestFeature"
   5    }
   6  }
```

## `.github/workflows/dotnetcore.yml` (F-04, F-16)

The only automated pipeline: build and test.

Lines 1–22:

```yaml
   1  name: eShopOnWeb Build and Test
   2  
   3  on: [push, pull_request, workflow_dispatch]
   4  
   5  jobs:
   6    build:
   7  
   8      runs-on: ubuntu-latest
   9  
  10      steps:
  11      - uses: actions/checkout@v2
  12      - name: Setup .NET
  13        uses: actions/setup-dotnet@v1
  14        with:
  15          dotnet-version: '8.0.x'
  16          include-prerelease: true
  17  
  18      - name: Build with dotnet
  19        run: dotnet build ./eShopOnWeb.sln --configuration Release
  20      
  21      - name: Test with dotnet
  22        run: dotnet test ./eShopOnWeb.sln --configuration Release
```

## `.github/dependabot.yml` (F-16)

Dependabot covers NuGet only.

Lines 1–6:

```yaml
   1  version: 2
   2  updates:
   3    - package-ecosystem: "nuget"
   4      directory: "/"
   5      schedule:
   6        interval: "daily"
```

## `src/Web/Dockerfile` (F-04, F-20)

Lines 10–27:

```
  10  FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
  11  WORKDIR /app
  12  
  13  COPY *.sln .
  14  COPY . .
  15  WORKDIR /app/src/Web
  16  RUN dotnet restore
  17  
  18  RUN dotnet publish -c Release -o out
  19  
  20  FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
  21  WORKDIR /app
  22  COPY --from=build /app/src/Web/out ./
  23  
  24  # Optional: Set this here if not setting it from docker-compose.yml
  25  # ENV ASPNETCORE_ENVIRONMENT Development
  26  
  27  ENTRYPOINT ["dotnet", "Web.dll"]
```

## `src/PublicApi/Dockerfile` (F-04, F-20)

Lines 1–25:

```
   1  #See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.
   2  
   3  FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
   4  WORKDIR /app
   5  EXPOSE 80
   6  EXPOSE 443
   7  
   8  FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
   9  WORKDIR /app
  10  COPY . .
  11  #COPY ["src/PublicApi/PublicApi.csproj", "./PublicApi/"]
  12  #RUN dotnet restore "./PublicApi/PublicApi.csproj"
  13  #COPY . .
  14  WORKDIR "/app/src/PublicApi"
  15  RUN dotnet restore
  16  
  17  RUN dotnet build "./PublicApi.csproj" -c Release -o /app/build
  18  
  19  FROM build AS publish
  20  RUN dotnet publish "./PublicApi.csproj" -c Release -o /app/publish
  21  
  22  FROM base AS final
  23  WORKDIR /app
  24  COPY --from=publish /app/publish .
  25  ENTRYPOINT ["dotnet", "PublicApi.dll"]
```

## `.dockerignore` (F-20)

Lines 1–9:

```
   1  .dockerignore
   2  .env
   3  .git
   4  .gitignore
   5  .vs
   6  .vscode
   7  */bin
   8  */obj
   9  **/.toolstarget
```

## `src/Web/Views/Shared/_Layout.cshtml` (F-12, F-17)

Production pages load different library versions from CDNs; no lang attribute.

Lines 1–25:

```cshtml
   1  <!DOCTYPE html>
   2  <html>
   3  <head>
   4      <meta charset="utf-8" />
   5      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
   6      <title>@ViewData["Title"] - Microsoft.eShopOnWeb</title>
   7      <environment names="Development,Docker">
   8          <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.css" />
   9          <link rel="stylesheet" href="~/css/app.css" />
  10          <link rel="stylesheet" href="~/css/app.component.css" />
  11          <link rel="stylesheet" href="~/css/shared/components/header/header.css" />
  12          <link rel="stylesheet" href="~/css/shared/components/identity/identity.css" />
  13          <link rel="stylesheet" href="~/css/shared/components/pager/pager.css" />
  14          <link rel="stylesheet" href="~/css/basket/basket.component.css" />
  15          <link rel="stylesheet" href="~/css/basket/basket-status/basket-status.component.css" />
  16          <link rel="stylesheet" href="~/css/catalog/catalog.component.css" />
  17          <link rel="stylesheet" href="~/css/orders/orders.component.css" />
  18      </environment>
  19      <environment names="Staging,Production">
  20          <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/4.0.0-alpha.5/css/bootstrap.min.css"
  21                asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
  22                asp-fallback-test-class="sr-only" asp-fallback-test-property="position" asp-fallback-test-value="absolute" />
  23          <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
  24      </environment>
  25  </head>
```

Lines 52–68:

```cshtml
  52      <environment names="Development,Docker">
  53          <script src="~/lib/jquery/jquery.js"></script>
  54          <script src="~/lib/bootstrap/dist/js/bootstrap.js"></script>
  55          <script src="~/js/site.js" asp-append-version="true"></script>
  56      </environment>
  57      <environment names="Staging,Production">
  58          <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-2.2.0.min.js"
  59                  asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
  60                  asp-fallback-test="window.jQuery">
  61          </script>
  62          <script src="https://ajax.aspnetcdn.com/ajax/bootstrap/3.3.6/bootstrap.min.js"
  63                  asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.min.js"
  64                  asp-fallback-test="window.jQuery && window.jQuery.fn && window.jQuery.fn.modal">
  65          </script>
  66          <script src="~/js/site.min.js" asp-append-version="true"></script>
  67      </environment>
  68      @RenderSection("scripts", required: false)
```

## `src/Web/Views/Shared/_ValidationScriptsPartial.cshtml` (F-12)

Lines 1–18:

```cshtml
   1  <environment names="Development,Docker">
   2      <script src="~/lib/jquery-validation/dist/jquery.validate.js"></script>
   3      <script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.js"></script>
   4  </environment>
   5  <environment names="Staging,Production">
   6      <script src="https://ajax.aspnetcdn.com/ajax/jquery.validate/1.14.0/jquery.validate.min.js"
   7              asp-fallback-src="~/lib/jquery-validation/dist/jquery.validate.min.js"
   8              asp-fallback-test="window.jQuery && window.jQuery.validator"
   9              crossorigin="anonymous"
  10              integrity="sha384-Fnqn3nxp3506LP/7Y3j/25BlWeA3PXTyT1l78LjECcPaKCV12TsZP7yyMxOe/G/k">
  11      </script>
  12      <script src="https://ajax.aspnetcdn.com/ajax/jquery.validation.unobtrusive/3.2.6/jquery.validate.unobtrusive.min.js"
  13              asp-fallback-src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"
  14              asp-fallback-test="window.jQuery && window.jQuery.validator && window.jQuery.validator.unobtrusive"
  15              crossorigin="anonymous"
  16              integrity="sha384-JrXK+k53HACyavUKOsL+NkmSesD2P+73eDMrbTtTk0h4RmOF8hF8apPlkp26JlyH">
  17      </script>
  18  </environment>
```

## `src/Web/libman.json` (F-12)

What developers get locally.

Lines 1–46:

```json
   1  {
   2    "version": "1.0",
   3    "defaultProvider": "cdnjs",
   4    "libraries": [
   5      {
   6        "library": "jquery@3.6.3",
   7        "destination": "wwwroot/lib/jquery/"
   8      },
   9      {
  10        "library": "twitter-bootstrap@3.4.1",
  11        "files": [
  12          "css/bootstrap.css",
  13          "css/bootstrap.css.map",
  14          "css/bootstrap.min.css",
  15          "css/bootstrap.min.css.map",
  16          "js/bootstrap.js",
  17          "js/bootstrap.min.js"
  18        ],
  19        "destination": "wwwroot/lib/bootstrap/dist/"
  20      },
  21      {
  22        "library": "jquery-validation-unobtrusive@4.0.0",
  23        "destination": "wwwroot/lib/jquery-validation-unobtrusive/"
  24      },
  25      {
  26        "library": "jquery-validate@1.19.5",
  27        "destination": "wwwroot/lib/jquery-validate/",
  28        "files": [
  29          "jquery.validate.min.js",
  30          "jquery.validate.js"
  31        ]
  32      },
  33      {
  34        "library": "toastr.js@2.1.4",
  35        "destination": "wwwroot/lib/toastr/"
  36      },
  37      {
  38        "library": "aspnet-signalr@1.0.27",
  39        "files": [
  40          "signalr.js",
  41          "signalr.min.js"
  42        ],
  43        "destination": "wwwroot/lib/@aspnet/signalr/dist/browser/"
  44      }
  45    ]
  46  }
```

## `src/Web/Configuration/ConfigureCookieSettings.cs` (F-06)

Line 29 replaces the cookie builder, discarding HttpOnly set on line 25.

Lines 13–39:

```csharp
  13      public static IServiceCollection AddCookieSettings(this IServiceCollection services)
  14      {
  15          services.Configure<CookiePolicyOptions>(options =>
  16          {
  17                  // This lambda determines whether user consent for non-essential cookies is needed for a given request.
  18                  //TODO need to check that.
  19                  //options.CheckConsentNeeded = context => true;
  20                  options.MinimumSameSitePolicy = SameSiteMode.Strict;
  21          });
  22          services.ConfigureApplicationCookie(options =>
  23          {
  24              options.EventsType = typeof(RevokeAuthenticationEvents);
  25              options.Cookie.HttpOnly = true;
  26              options.ExpireTimeSpan = TimeSpan.FromMinutes(ValidityMinutesPeriod);
  27              options.LoginPath = "/Account/Login";
  28              options.LogoutPath = "/Account/Logout";
  29              options.Cookie = new CookieBuilder
  30              {
  31                  Name = IdentifierCookieName,
  32                  IsEssential = true // required for auth to work without explicit user consent; adjust to suit your privacy policy
  33                  };
  34          });
  35  
  36          services.AddScoped<RevokeAuthenticationEvents>();
  37  
  38          return services;
  39      }
```

## `src/Web/Configuration/RevokeAuthenticationEvents.cs` (F-06, F-18)

Custom events type replaces Identity's default validate-principal events.

Lines 11–34:

```csharp
  11  //TODO : replace IMemoryCache with a distributed cache if you are in multi-host scenario
  12  public class RevokeAuthenticationEvents : CookieAuthenticationEvents
  13  {
  14      private readonly IMemoryCache _cache;
  15      private readonly ILogger _logger;
  16  
  17      public RevokeAuthenticationEvents(IMemoryCache cache, ILogger<RevokeAuthenticationEvents> logger)
  18      {
  19          _cache = cache;
  20          _logger = logger;
  21      }
  22  
  23      public override async Task ValidatePrincipal(CookieValidatePrincipalContext context)
  24      {
  25          var userId = context.Principal?.Claims.First(c => c.Type == ClaimTypes.Name);
  26          var identityKey = context.Request.Cookies[ConfigureCookieSettings.IdentifierCookieName];
  27  
  28          if (_cache.TryGetValue($"{userId?.Value}:{identityKey}", out var revokeKeys))
  29          {
  30              _logger.LogDebug($"Access has been revoked for: {userId?.Value}.");
  31              context.RejectPrincipal();
  32              await context.HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
  33          }
  34      }
```

## `src/Web/Areas/Identity/Pages/Account/Logout.cshtml.cs` (F-06, F-18)

Revocation list held in process memory.

Lines 13–13:

```csharp
  13  //TODO : replace IMemoryCache by distributed cache if you are in multi-host scenario
```

Lines 31–51:

```csharp
  31      public async Task<IActionResult> OnPost(string? returnUrl = null)
  32      {
  33          await _signInManager.SignOutAsync();
  34          await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
  35          var userId = _signInManager.Context.User.Claims.First(c => c.Type == ClaimTypes.Name);
  36          var identityKey = _signInManager.Context.Request.Cookies[ConfigureCookieSettings.IdentifierCookieName];
  37          _cache.Set($"{userId.Value}:{identityKey}", identityKey, new MemoryCacheEntryOptions
  38          {
  39              AbsoluteExpiration = DateTime.Now.AddMinutes(ConfigureCookieSettings.ValidityMinutesPeriod)
  40          });
  41  
  42          _logger.LogInformation("User logged out.");
  43          if (returnUrl != null)
  44          {
  45              return LocalRedirect(returnUrl);
  46          }
  47          else
  48          {
  49              return RedirectToPage("/Index");
  50          }
  51      }
```

## `src/Web/Services/BasketViewModelService.cs` (F-07, F-19)

First(...) throws when a basket references a deleted product; GET creates a basket row.

Lines 28–75:

```csharp
  28      public async Task<BasketViewModel> GetOrCreateBasketForUser(string userName)
  29      {
  30          var basketSpec = new BasketWithItemsSpecification(userName);
  31          var basket = (await _basketRepository.FirstOrDefaultAsync(basketSpec));
  32  
  33          if (basket == null)
  34          {
  35              return await CreateBasketForUser(userName);
  36          }
  37          var viewModel = await Map(basket);
  38          return viewModel;
  39      }
  40  
  41      private async Task<BasketViewModel> CreateBasketForUser(string userId)
  42      {
  43          var basket = new Basket(userId);
  44          await _basketRepository.AddAsync(basket);
  45  
  46          return new BasketViewModel()
  47          {
  48              BuyerId = basket.BuyerId,
  49              Id = basket.Id,
  50          };
  51      }
  52  
  53      private async Task<List<BasketItemViewModel>> GetBasketItems(IReadOnlyCollection<BasketItem> basketItems)
  54      {
  55          var catalogItemsSpecification = new CatalogItemsSpecification(basketItems.Select(b => b.CatalogItemId).ToArray());
  56          var catalogItems = await _itemRepository.ListAsync(catalogItemsSpecification);
  57  
  58          var items = basketItems.Select(basketItem =>
  59          {
  60              var catalogItem = catalogItems.First(c => c.Id == basketItem.CatalogItemId);
  61  
  62              var basketItemViewModel = new BasketItemViewModel
  63              {
  64                  Id = basketItem.Id,
  65                  UnitPrice = basketItem.UnitPrice,
  66                  Quantity = basketItem.Quantity,
  67                  CatalogItemId = basketItem.CatalogItemId,
  68                  PictureUrl = _uriComposer.ComposePicUri(catalogItem.PictureUri),
  69                  ProductName = catalogItem.Name
  70              };
  71              return basketItemViewModel;
  72          }).ToList();
  73  
  74          return items;
  75      }
```

## `src/ApplicationCore/Services/OrderService.cs` (F-07)

Same pattern at checkout.

Lines 30–52:

```csharp
  30      public async Task CreateOrderAsync(int basketId, Address shippingAddress)
  31      {
  32          var basketSpec = new BasketWithItemsSpecification(basketId);
  33          var basket = await _basketRepository.FirstOrDefaultAsync(basketSpec);
  34  
  35          Guard.Against.Null(basket, nameof(basket));
  36          Guard.Against.EmptyBasketOnCheckout(basket.Items);
  37  
  38          var catalogItemsSpecification = new CatalogItemsSpecification(basket.Items.Select(item => item.CatalogItemId).ToArray());
  39          var catalogItems = await _itemRepository.ListAsync(catalogItemsSpecification);
  40  
  41          var items = basket.Items.Select(basketItem =>
  42          {
  43              var catalogItem = catalogItems.First(c => c.Id == basketItem.CatalogItemId);
  44              var itemOrdered = new CatalogItemOrdered(catalogItem.Id, catalogItem.Name, _uriComposer.ComposePicUri(catalogItem.PictureUri));
  45              var orderItem = new OrderItem(itemOrdered, basketItem.UnitPrice, basketItem.Quantity);
  46              return orderItem;
  47          }).ToList();
  48  
  49          var order = new Order(basket.BuyerId, shippingAddress, items);
  50  
  51          await _orderRepository.AddAsync(order);
  52      }
```

## `src/Infrastructure/Data/Migrations/CatalogContextModelSnapshot.cs` (F-07)

BasketItems.CatalogItemId has no foreign key or index.

Lines 52–77:

```csharp
  52              modelBuilder.Entity("Microsoft.eShopWeb.ApplicationCore.Entities.BasketAggregate.BasketItem", b =>
  53                  {
  54                      b.Property<int>("Id")
  55                          .ValueGeneratedOnAdd()
  56                          .HasColumnType("int");
  57  
  58                      SqlServerPropertyBuilderExtensions.UseIdentityColumn(b.Property<int>("Id"), 1L, 1);
  59  
  60                      b.Property<int>("BasketId")
  61                          .HasColumnType("int");
  62  
  63                      b.Property<int>("CatalogItemId")
  64                          .HasColumnType("int");
  65  
  66                      b.Property<int>("Quantity")
  67                          .HasColumnType("int");
  68  
  69                      b.Property<decimal>("UnitPrice")
  70                          .HasColumnType("decimal(18,2)");
  71  
  72                      b.HasKey("Id");
  73  
  74                      b.HasIndex("BasketId");
  75  
  76                      b.ToTable("BasketItems");
  77                  });
```

Lines 197–204:

```csharp
 197              modelBuilder.Entity("Microsoft.eShopWeb.ApplicationCore.Entities.BasketAggregate.BasketItem", b =>
 198                  {
 199                      b.HasOne("Microsoft.eShopWeb.ApplicationCore.Entities.BasketAggregate.Basket", null)
 200                          .WithMany("Items")
 201                          .HasForeignKey("BasketId")
 202                          .OnDelete(DeleteBehavior.Cascade)
 203                          .IsRequired();
 204                  });
```

## `infra/core/database/sqlserver/sqlserver.bicep` (F-08, F-09)

Firewall open to almost all addresses; app user is db_owner; database tier not set; tool downloaded at deploy time.

Lines 16–43:

```bicep
  16  resource sqlServer 'Microsoft.Sql/servers@2022-05-01-preview' = {
  17    name: name
  18    location: location
  19    tags: tags
  20    properties: {
  21      version: '12.0'
  22      minimalTlsVersion: '1.2'
  23      publicNetworkAccess: 'Enabled'
  24      administratorLogin: sqlAdmin
  25      administratorLoginPassword: sqlAdminPassword
  26    }
  27  
  28    resource database 'databases' = {
  29      name: databaseName
  30      location: location
  31    }
  32  
  33    resource firewall 'firewallRules' = {
  34      name: 'Azure Services'
  35      properties: {
  36        // Allow all clients
  37        // Note: range [0.0.0.0-0.0.0.0] means "allow all Azure-hosted clients only".
  38        // This is not sufficient, because we also want to allow direct access from developer machine, for debugging purposes.
  39        startIpAddress: '0.0.0.1'
  40        endIpAddress: '255.255.255.254'
  41      }
  42    }
  43  }
```

Lines 81–95:

```bicep
  81      scriptContent: '''
  82  wget https://github.com/microsoft/go-sqlcmd/releases/download/v0.8.1/sqlcmd-v0.8.1-linux-x64.tar.bz2
  83  tar x -f sqlcmd-v0.8.1-linux-x64.tar.bz2 -C .
  84  
  85  cat <<SCRIPT_END > ./initDb.sql
  86  drop user ${APPUSERNAME}
  87  go
  88  create user ${APPUSERNAME} with password = '${APPUSERPASSWORD}'
  89  go
  90  alter role db_owner add member ${APPUSERNAME}
  91  go
  92  SCRIPT_END
  93  
  94  ./sqlcmd -S ${DBSERVER} -d ${DBNAME} -U ${SQLADMIN} -i ./initDb.sql
  95      '''
```

## `src/Infrastructure/Data/CatalogContextSeed.cs` (F-08, F-22)

Migrate() at start-up; the retry path rethrows even after a successful retry.

Lines 12–58:

```csharp
  12      public static async Task SeedAsync(CatalogContext catalogContext,
  13          ILogger logger,
  14          int retry = 0)
  15      {
  16          var retryForAvailability = retry;
  17          try
  18          {
  19              if (catalogContext.Database.IsSqlServer())
  20              {
  21                  catalogContext.Database.Migrate();
  22              }
  23  
  24              if (!await catalogContext.CatalogBrands.AnyAsync())
  25              {
  26                  await catalogContext.CatalogBrands.AddRangeAsync(
  27                      GetPreconfiguredCatalogBrands());
  28  
  29                  await catalogContext.SaveChangesAsync();
  30              }
  31  
  32              if (!await catalogContext.CatalogTypes.AnyAsync())
  33              {
  34                  await catalogContext.CatalogTypes.AddRangeAsync(
  35                      GetPreconfiguredCatalogTypes());
  36  
  37                  await catalogContext.SaveChangesAsync();
  38              }
  39  
  40              if (!await catalogContext.CatalogItems.AnyAsync())
  41              {
  42                  await catalogContext.CatalogItems.AddRangeAsync(
  43                      GetPreconfiguredItems());
  44  
  45                  await catalogContext.SaveChangesAsync();
  46              }
  47          }
  48          catch (Exception ex)
  49          {
  50              if (retryForAvailability >= 10) throw;
  51  
  52              retryForAvailability++;
  53              
  54              logger.LogError(ex.Message);
  55              await SeedAsync(catalogContext, logger, retryForAvailability);
  56              throw;
  57          }
  58      }
```

## `infra/main.bicep` (F-04, F-09)

Only the Web app is provisioned, on a B1 plan.

Lines 47–65:

```bicep
  47  // The application frontend
  48  module web './core/host/appservice.bicep' = {
  49    name: 'web'
  50    scope: rg
  51    params: {
  52      name: !empty(webServiceName) ? webServiceName : '${abbrs.webSitesAppService}web-${resourceToken}'
  53      location: location
  54      appServicePlanId: appServicePlan.outputs.id
  55      keyVaultName: keyVault.outputs.name
  56      runtimeName: 'dotnetcore'
  57      runtimeVersion: '8.0'
  58      tags: union(tags, { 'azd-service-name': 'web' })
  59      appSettings: {
  60        AZURE_SQL_CATALOG_CONNECTION_STRING_KEY: 'AZURE-SQL-CATALOG-CONNECTION-STRING'
  61        AZURE_SQL_IDENTITY_CONNECTION_STRING_KEY: 'AZURE-SQL-IDENTITY-CONNECTION-STRING'
  62        AZURE_KEY_VAULT_ENDPOINT: keyVault.outputs.endpoint
  63      }
  64    }
  65  }
```

Lines 120–132:

```bicep
 120  // Create an App Service Plan to group applications under the same payment plan and SKU
 121  module appServicePlan './core/host/appserviceplan.bicep' = {
 122    name: 'appserviceplan'
 123    scope: rg
 124    params: {
 125      name: !empty(appServicePlanName) ? appServicePlanName : '${abbrs.webServerFarms}${resourceToken}'
 126      location: location
 127      tags: tags
 128      sku: {
 129        name: 'B1'
 130      }
 131    }
 132  }
```

## `azure.yaml` (F-09)

Only the Web project is deployed.

Lines 1–8:

```yaml
   1  # yaml-language-server: $schema=https://raw.githubusercontent.com/wbreza/azure-dev/main/schemas/v1.0/azure.yaml.json
   2  
   3  name: eShopOnWeb
   4  services:
   5    web:
   6      project: ./src/Web
   7      language: csharp
   8      host: appservice
```

## `src/BlazorAdmin/wwwroot/appsettings.json` (F-09)

The admin client's API address defaults to localhost.

Lines 1–5:

```json
   1  {
   2    "baseUrls": {
   3      "apiBase": "https://localhost:5099/api/",
   4      "webBase": "https://localhost:44315/"
   5    },
```

## `infra/core/host/appservice.bicep` (F-09, F-10)

Verbose file logs and detailed errors; Application Insights optional and not wired.

Lines 5–6:

```bicep
   5  // Reference Properties
   6  param applicationInsightsName string = ''
```

Lines 77–88:

```bicep
  77    resource configLogs 'config' = {
  78      name: 'logs'
  79      properties: {
  80        applicationLogs: { fileSystem: { level: 'Verbose' } }
  81        detailedErrorMessages: { enabled: true }
  82        failedRequestsTracing: { enabled: true }
  83        httpLogs: { fileSystem: { enabled: true, retentionInDays: 1, retentionInMb: 35 } }
  84      }
  85      dependsOn: [
  86        configAppSettings
  87      ]
  88    }
```

## `src/Web/HealthChecks/HomePageHealthCheck.cs` (F-10, F-23)

URL built from the incoming Host header; new HttpClient per call; health tied to a product name.

Lines 18–34:

```csharp
  18      public async Task<HealthCheckResult> CheckHealthAsync(
  19          HealthCheckContext context,
  20          CancellationToken cancellationToken = default(CancellationToken))
  21      {
  22          var request = _httpContextAccessor.HttpContext?.Request;
  23          string myUrl = request?.Scheme + "://" + request?.Host.ToString();
  24  
  25          var client = new HttpClient();
  26          var response = await client.GetAsync(myUrl);
  27          var pageContents = await response.Content.ReadAsStringAsync();
  28          if (pageContents.Contains(".NET Bot Black Sweatshirt"))
  29          {
  30              return HealthCheckResult.Healthy("The check indicates a healthy result.");
  31          }
  32  
  33          return HealthCheckResult.Unhealthy("The check indicates an unhealthy result.");
  34      }
```

## `src/Web/HealthChecks/ApiHealthCheck.cs` (F-10)

The API health check calls the configured API base address (localhost by default).

Lines 19–33:

```csharp
  19      public async Task<HealthCheckResult> CheckHealthAsync(
  20          HealthCheckContext context,
  21          CancellationToken cancellationToken = default(CancellationToken))
  22      {
  23          string myUrl = _baseUrlConfiguration.ApiBase + "catalog-items";
  24          var client = new HttpClient();
  25          var response = await client.GetAsync(myUrl);
  26          var pageContents = await response.Content.ReadAsStringAsync();
  27          if (pageContents.Contains(".NET Bot Black Sweatshirt"))
  28          {
  29              return HealthCheckResult.Healthy("The check indicates a healthy result.");
  30          }
  31  
  32          return HealthCheckResult.Unhealthy("The check indicates an unhealthy result.");
  33      }
```

## `src/Web/appsettings.json` (F-23)

AllowedHosts is nested inside Logging, so host filtering is not configured.

Lines 10–20:

```json
  10    "CatalogBaseUrl": "",
  11    "Logging": {
  12      "IncludeScopes": false,
  13      "LogLevel": {
  14        "Default": "Warning",
  15        "Microsoft": "Warning",
  16        "System": "Warning"
  17      },
  18      "AllowedHosts": "*"
  19    }
  20  }
```

## `src/PublicApi/Middleware/ExceptionMiddleware.cs` (F-11)

Raw exception messages returned to callers.

Lines 31–53:

```csharp
  31      private async Task HandleExceptionAsync(HttpContext context, Exception exception)
  32      {
  33          context.Response.ContentType = "application/json";
  34  
  35          if (exception is DuplicateException duplicationException)
  36          {
  37              context.Response.StatusCode = (int)HttpStatusCode.Conflict;
  38              await context.Response.WriteAsync(new ErrorDetails()
  39              {
  40                  StatusCode = context.Response.StatusCode,
  41                  Message = duplicationException.Message
  42              }.ToString());
  43          }
  44          else
  45          {
  46              context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;
  47              await context.Response.WriteAsync(new ErrorDetails()
  48              {
  49                  StatusCode = context.Response.StatusCode,
  50                  Message = exception.Message
  51              }.ToString());
  52          }
  53      }
```

## `src/PublicApi/Program.cs` (F-11)

Swagger UI in every environment.

Lines 150–176:

```csharp
 150  if (app.Environment.IsDevelopment())
 151  {
 152      app.UseDeveloperExceptionPage();
 153  }
 154  
 155  app.UseMiddleware<ExceptionMiddleware>();
 156  
 157  app.UseHttpsRedirection();
 158  
 159  app.UseRouting();
 160  
 161  app.UseCors(CORS_POLICY);
 162  
 163  app.UseAuthorization();
 164  
 165  // Enable middleware to serve generated Swagger as a JSON endpoint.
 166  app.UseSwagger();
 167  
 168  // Enable middleware to serve swagger-ui (HTML, JS, CSS, etc.), 
 169  // specifying the Swagger JSON endpoint.
 170  app.UseSwaggerUI(c =>
 171  {
 172      c.SwaggerEndpoint("/swagger/v1/swagger.json", "My API V1");
 173  });
 174  
 175  app.MapControllers();
 176  app.MapEndpoints();
```

## `src/Infrastructure/Services/EmailSender.cs` (F-13)

No e-mail is ever sent.

Lines 8–15:

```csharp
   8  public class EmailSender : IEmailSender
   9  {
  10      public Task SendEmailAsync(string email, string subject, string message)
  11      {
  12          // TODO: Wire this up to actual email sending logic via SendGrid, local SMTP, etc.
  13          return Task.CompletedTask;
  14      }
  15  }
```

## `src/Web/Areas/Identity/Pages/Account/Register.cshtml.cs` (F-13)

Six-character minimum; user signed in before any confirmation.

Lines 49–53:

```csharp
  49          [Required]
  50          [StringLength(100, ErrorMessage = "The {0} must be at least {2} and at max {1} characters long.", MinimumLength = 6)]
  51          [DataType(DataType.Password)]
  52          [Display(Name = "Password")]
  53          public string? Password { get; set; }
```

Lines 71–89:

```csharp
  71              var user = new ApplicationUser { UserName = Input?.Email, Email = Input?.Email };
  72              var result = await _userManager.CreateAsync(user, Input?.Password!);
  73              if (result.Succeeded)
  74              {
  75                  _logger.LogInformation("User created a new account with password.");
  76  
  77                  var code = await _userManager.GenerateEmailConfirmationTokenAsync(user);
  78                  var callbackUrl = Url.Page(
  79                      "/Account/ConfirmEmail",
  80                      pageHandler: null,
  81                      values: new { userId = user.Id, code = code },
  82                      protocol: Request.Scheme);
  83  
  84                  Guard.Against.Null(callbackUrl, nameof(callbackUrl));
  85                  await _emailSender.SendEmailAsync(Input!.Email!, "Confirm your email",
  86                      $"Please confirm your account by <a href='{HtmlEncoder.Default.Encode(callbackUrl)}'>clicking here</a>.");
  87  
  88                  await _signInManager.SignInAsync(user, isPersistent: false);
  89                  return LocalRedirect(returnUrl);
```

## `src/Web/Controllers/UserController.cs` (F-13)

[Authorize] and [AllowAnonymous] on the same actions; a bearer token is handed to the browser.

Lines 35–58:

```csharp
  35      [HttpGet]
  36      [Authorize]
  37      [AllowAnonymous]
  38      public async Task<IActionResult> GetCurrentUser() =>
  39          Ok(await CreateUserInfo(User));
  40  
  41      [Route("Logout")]
  42      [HttpPost]
  43      [Authorize]
  44      [AllowAnonymous]
  45      public async Task<IActionResult> Logout()
  46      {
  47          await _signInManager.SignOutAsync();
  48          await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
  49          var userId = _signInManager.Context.User.Claims.First(c => c.Type == ClaimTypes.Name);
  50          var identityKey = _signInManager.Context.Request.Cookies[ConfigureCookieSettings.IdentifierCookieName];
  51          _cache.Set($"{userId.Value}:{identityKey}", identityKey, new MemoryCacheEntryOptions
  52          {
  53              AbsoluteExpiration = DateTime.Now.AddMinutes(ConfigureCookieSettings.ValidityMinutesPeriod)
  54          });
  55  
  56          _logger.LogInformation("User logged out.");
  57          return Ok();
  58      }
```

Lines 100–101:

```csharp
 100          var token = await _tokenClaimsService.GetTokenAsync(claimsPrincipal.Identity.Name);
 101          userInfo.Token = token;
```

## `src/PublicApi/CatalogItemEndpoints/CatalogItemListPagedEndpoint.cs` (F-15)

One-second delay on every call; anonymous endpoint.

Lines 29–54:

```csharp
  29      public void AddRoute(IEndpointRouteBuilder app)
  30      {
  31          app.MapGet("api/catalog-items",
  32              async (int? pageSize, int? pageIndex, int? catalogBrandId, int? catalogTypeId, IRepository<CatalogItem> itemRepository) =>
  33              {
  34                  return await HandleAsync(new ListPagedCatalogItemRequest(pageSize, pageIndex, catalogBrandId, catalogTypeId), itemRepository);
  35              })
  36              .Produces<ListPagedCatalogItemResponse>()
  37              .WithTags("CatalogItemEndpoints");
  38      }
  39  
  40      public async Task<IResult> HandleAsync(ListPagedCatalogItemRequest request, IRepository<CatalogItem> itemRepository)
  41      {
  42          await Task.Delay(1000);
  43          var response = new ListPagedCatalogItemResponse(request.CorrelationId());
  44  
  45          var filterSpec = new CatalogFilterSpecification(request.CatalogBrandId, request.CatalogTypeId);
  46          int totalItems = await itemRepository.CountAsync(filterSpec);
  47  
  48          var pagedSpec = new CatalogFilterPaginatedSpecification(
  49              skip: request.PageIndex * request.PageSize,
  50              take: request.PageSize,
  51              brandId: request.CatalogBrandId,
  52              typeId: request.CatalogTypeId);
  53  
  54          var items = await itemRepository.ListAsync(pagedSpec);
```

## `src/PublicApi/CatalogItemEndpoints/CatalogItemListPagedEndpoint.ListPagedCatalogItemRequest.cs` (F-15)

Lines 10–16:

```csharp
  10      public ListPagedCatalogItemRequest(int? pageSize, int? pageIndex, int? catalogBrandId, int? catalogTypeId)
  11      {
  12          PageSize = pageSize ?? 0;
  13          PageIndex = pageIndex ?? 0;
  14          CatalogBrandId = catalogBrandId;
  15          CatalogTypeId = catalogTypeId;
  16      }
```

## `src/ApplicationCore/Specifications/CatalogFilterPaginatedSpecification.cs` (F-15)

Page size 0 becomes int.MaxValue.

Lines 8–19:

```csharp
   8      public CatalogFilterPaginatedSpecification(int skip, int take, int? brandId, int? typeId)
   9          : base()
  10      {
  11          if (take == 0)
  12          {
  13              take = int.MaxValue;
  14          }
  15          Query
  16              .Where(i => (!brandId.HasValue || i.CatalogBrandId == brandId) &&
  17              (!typeId.HasValue || i.CatalogTypeId == typeId))
  18              .Skip(skip).Take(take);
  19      }
```

## `src/BlazorAdmin/Services/CatalogItemService.cs` (F-15)

The admin list requests the catalogue without a page size.

Lines 78–95:

```csharp
  78      public async Task<List<CatalogItem>> List()
  79      {
  80          _logger.LogInformation("Fetching catalog items from API.");
  81  
  82          var brandListTask = _brandService.List();
  83          var typeListTask = _typeService.List();
  84          var itemListTask = _httpService.HttpGet<PagedCatalogItemResponse>($"catalog-items");
  85          await Task.WhenAll(brandListTask, typeListTask, itemListTask);
  86          var brands = brandListTask.Result;
  87          var types = typeListTask.Result;
  88          var items = itemListTask.Result.CatalogItems;
  89          foreach (var item in items)
  90          {
  91              item.CatalogBrand = brands.FirstOrDefault(b => b.Id == item.CatalogBrandId)?.Name;
  92              item.CatalogType = types.FirstOrDefault(t => t.Id == item.CatalogTypeId)?.Name;
  93          }
  94          return items;
  95      }
```

## `src/Web/Pages/Basket/Index.cshtml.cs` (F-19)

Ten-year anonymous basket cookie.

Lines 28–31:

```csharp
  28      public async Task OnGet()
  29      {
  30          BasketModel = await _basketViewModelService.GetOrCreateBasketForUser(GetOrSetBasketCookieAndUserName());
  31      }
```

Lines 68–98:

```csharp
  68      private string GetOrSetBasketCookieAndUserName()
  69      {
  70          Guard.Against.Null(Request.HttpContext.User.Identity, nameof(Request.HttpContext.User.Identity));
  71          string? userName = null;
  72  
  73          if (Request.HttpContext.User.Identity.IsAuthenticated)
  74          {
  75              Guard.Against.Null(Request.HttpContext.User.Identity.Name, nameof(Request.HttpContext.User.Identity.Name));
  76              return Request.HttpContext.User.Identity.Name!;
  77          }
  78  
  79          if (Request.Cookies.ContainsKey(Constants.BASKET_COOKIENAME))
  80          {
  81              userName = Request.Cookies[Constants.BASKET_COOKIENAME];
  82  
  83              if (!Request.HttpContext.User.Identity.IsAuthenticated)
  84              {
  85                  if (!Guid.TryParse(userName, out var _))
  86                  {
  87                      userName = null;
  88                  }
  89              }
  90          }
  91          if (userName != null) return userName;
  92  
  93          userName = Guid.NewGuid().ToString();
  94          var cookieOptions = new CookieOptions { IsEssential = true };
  95          cookieOptions.Expires = DateTime.Today.AddYears(10);
  96          Response.Cookies.Append(Constants.BASKET_COOKIENAME, userName, cookieOptions);
  97  
  98          return userName;
```

## `src/Web/Configuration/ConfigureCookieSettings.cs` (F-19)

Consent check commented out.

Lines 15–21:

```csharp
  15          services.Configure<CookiePolicyOptions>(options =>
  16          {
  17                  // This lambda determines whether user consent for non-essential cookies is needed for a given request.
  18                  //TODO need to check that.
  19                  //options.CheckConsentNeeded = context => true;
  20                  options.MinimumSameSitePolicy = SameSiteMode.Strict;
  21          });
```

## `src/Web/Pages/Basket/Checkout.cshtml.cs` (Overview)

Checkout records a fixed placeholder address (values redacted).

Lines 55–58:

```csharp
  55              var updateModel = items.ToDictionary(b => b.Id.ToString(), b => b.Quantity);
  56              await _basketService.SetQuantities(BasketModel.Id, updateModel);
  57              await _orderService.CreateOrderAsync(BasketModel.Id, new Address(/* fixed placeholder address, redacted */));
  58              await _basketService.DeleteBasketAsync(BasketModel.Id);
```

## `src/ApplicationCore/ApplicationCore.csproj` (F-21)

The core project references the UI-shared project.

Lines 8–18:

```xml
   8  	<ItemGroup>
   9  		<PackageReference Include="Ardalis.GuardClauses" />
  10  		<PackageReference Include="Ardalis.Result" />
  11  		<PackageReference Include="Ardalis.Specification" />		
  12  		<PackageReference Include="System.Security.Claims" />
  13  		<PackageReference Include="System.Text.Json" />
  14  	</ItemGroup>
  15  
  16  	<ItemGroup>
  17  	  <ProjectReference Include="..\BlazorShared\BlazorShared.csproj" />
  18  	</ItemGroup>
```

## `src/Web/Controllers/Api/BaseApiController.cs` (F-21)

Lines 1–9:

```csharp
   1  using Microsoft.AspNetCore.Mvc;
   2  
   3  namespace Microsoft.eShopWeb.Web.Controllers.Api;
   4  
   5  // No longer used - shown for reference only if using full controllers instead of Endpoints for APIs
   6  [Route("api/[controller]/[action]")]
   7  [ApiController]
   8  public class BaseApiController : ControllerBase
   9  { }
```

## `tests/FunctionalTests/Web/WebTestFixture.cs` (F-16)

Functional tests use the EF Core in-memory provider.

Lines 15–50:

```csharp
  15  public class TestApplication : WebApplicationFactory<IBasketViewModelService>
  16  {
  17      private readonly string _environment = "Development";
  18  
  19      protected override IHost CreateHost(IHostBuilder builder)
  20      {
  21          builder.UseEnvironment(_environment);
  22  
  23          // Add mock/test services to the builder here
  24          builder.ConfigureServices(services =>
  25          {
  26              var descriptors = services.Where(d =>
  27                                                  d.ServiceType == typeof(DbContextOptions<CatalogContext>) ||
  28                                                  d.ServiceType == typeof(DbContextOptions<AppIdentityDbContext>))
  29                                              .ToList();
  30  
  31              foreach (var descriptor in descriptors)
  32              {
  33                  services.Remove(descriptor);
  34              }
  35  
  36              services.AddScoped(sp =>
  37              {
  38                  // Replace SQLite with in-memory database for tests
  39                  return new DbContextOptionsBuilder<CatalogContext>()
  40                  .UseInMemoryDatabase("InMemoryDbForTesting")
  41                  .UseApplicationServiceProvider(sp)
  42                  .Options;
  43              });
  44              services.AddScoped(sp =>
  45              {
  46                  // Replace SQLite with in-memory database for tests
  47                  return new DbContextOptionsBuilder<AppIdentityDbContext>()
  48                  .UseInMemoryDatabase("Identity")
  49                  .UseApplicationServiceProvider(sp)
  50                  .Options;
```

## `src/PublicApi/MappingProfile.cs` (F-14)

The entire AutoMapper configuration: three maps.

Lines 9–19:

```csharp
   9  public class MappingProfile : Profile
  10  {
  11      public MappingProfile()
  12      {
  13          CreateMap<CatalogItem, CatalogItemDto>();
  14          CreateMap<CatalogType, CatalogTypeDto>()
  15              .ForMember(dto => dto.Name, options => options.MapFrom(src => src.Type));
  16          CreateMap<CatalogBrand, CatalogBrandDto>()
  17              .ForMember(dto => dto.Name, options => options.MapFrom(src => src.Brand));
  18      }
  19  }
```

## `src/Web/Pages/Index.cshtml` (F-17)

No heading, image without alt, image submit button without alt, labels from CSS.

Lines 6–23:

```cshtml
   6  <section class="esh-catalog-hero">
   7      <div class="container">
   8          <img class="esh-catalog-title" src="~/images/main_banner_text.png" />
   9      </div>
  10  </section>
  11  <section class="esh-catalog-filters">
  12      <div class="container">
  13          <form method="get">
  14              <label class="esh-catalog-label" data-title="brand">
  15                  <select asp-for="@Model.CatalogModel.BrandFilterApplied" asp-items="@Model.CatalogModel.Brands" class="esh-catalog-filter"></select>
  16              </label>
  17              <label class="esh-catalog-label" data-title="type">
  18                  <select asp-for="@Model.CatalogModel.TypesFilterApplied" asp-items="@Model.CatalogModel.Types" class="esh-catalog-filter"></select>
  19              </label>
  20              <input class="esh-catalog-send" type="image" src="~/images/arrow-right.svg" />
  21          </form>
  22      </div>
  23  </section>
```

## `src/Web/Pages/Basket/Index.cshtml` (F-17)

Quantity input without a label.

Lines 37–40:

```cshtml
  37                              <section class="esh-basket-item esh-basket-item--middle col-xs-2">
  38                                  <input type="hidden" name="@("Items[" + i + "].Id")" value="@item.Id" />
  39                                  <input type="number" class="esh-basket-input" min="0" name="@("Items[" + i + "].Quantity")" value="@item.Quantity" />
  40                              </section>
```

## `src/Web/Pages/Admin/Index.cshtml` (F-17)

Zoom disabled; no lang attribute.

Lines 12–17:

```cshtml
  12  <!DOCTYPE html>
  13  <html>
  14  <head>
  15      <meta charset="utf-8" />
  16      <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />
  17      <title>@ViewData["Title"] - Microsoft.eShopOnWeb</title>
```

## `src/Web/wwwroot/css/shared/components/identity/identity.scss` (F-17)

Account menu opens on hover only.

Lines 29–45:

```scss
  29      &-drop {
  30          background: $color-background-brighter;
  31          height: 10px;
  32          width: 10rem;
  33          overflow: hidden;
  34          padding: .5rem;
  35          position: absolute;
  36          right: 0;
  37          top: 2.5rem;
  38          transition: height $animation-speed-default;
  39      }
  40  
  41      &:hover &-drop {
  42          border: $border-light solid $color-foreground-bright;
  43          height: 10rem;
  44          transition: height $animation-speed-default;
  45      }
```

## `src/Web/Views/Shared/_LoginPartial.cshtml` (F-17)

Lines 5–31:

```cshtml
   5              <form asp-area="Identity" asp-page="/Account/Logout" method="post" id="logoutForm" class="navbar-right">
   6                  <section class="esh-identity-section">
   7                      <div class="esh-identity-name">@Context.User.Identity.Name</div>
   8                      <img class="esh-identity-image" src="~/images/arrow-down.png">
   9                  </section>
  10                  <section class="esh-identity-drop">
  11                      @if (User.IsInRole("Administrators"))
  12                      {
  13                          <a class="esh-identity-item"
  14                             asp-page="/Admin/Index">
  15                              <div class="esh-identity-name esh-identity-name--upper">Admin</div>
  16                          </a>
  17                      }
  18                      <a class="esh-identity-item"
  19                         asp-controller="Order"
  20                         asp-action="MyOrders">
  21                          <div class="esh-identity-name esh-identity-name--upper">My orders</div>
  22                      </a>
  23                      <a class="esh-identity-item"
  24                         asp-controller="Manage"
  25                         asp-action="MyAccount">
  26                          <div class="esh-identity-name esh-identity-name--upper">My account</div>
  27                      </a>
  28                      <a class="esh-identity-item" href="javascript:document.getElementById('logoutForm').submit()">
  29                          <div class="esh-identity-name esh-identity-name--upper">Log Out</div>
  30                          <img class="esh-identity-image" src="~/images/logout.png">
  31                      </a>
```

## `src/BlazorAdmin/Pages/CatalogItemPage/List.razor` (F-17)

Click-only table rows; image without alt.

Lines 37–42:

```razor
  37              @foreach (var item in catalogItems)
  38              {
  39                  <tr @onclick="@(() => DetailsClick(item.Id))">
  40                      <td>
  41                          <img class="img-thumbnail" src="@($"{item.PictureUri}")">
  42                      </td>
```

## `src/BlazorAdmin/BlazorAdmin.csproj` (F-21)

Compile items for files that do not exist.

Lines 18–28:

```xml
  18  	<ItemGroup>
  19  		<Compile Update="Services\CatalogItem\Delete.EditCatalogItemResult.cs">
  20  			<DependentUpon>Delete.cs</DependentUpon>
  21  		</Compile>
  22  		<Compile Update="Services\CatalogItem\GetById.EditCatalogItemResult.cs">
  23  			<DependentUpon>GetById.cs</DependentUpon>
  24  		</Compile>
  25  		<Compile Update="Services\CatalogItem\Edit.CreateCatalogItemResult.cs">
  26  			<DependentUpon>Edit.cs</DependentUpon>
  27  		</Compile>
  28  	</ItemGroup>
```

## `src/Web/Views/Order/MyOrders.cshtml` (F-21)

Cancel link to an action that does not exist; status is never 'submitted'.

Lines 25–33:

```cshtml
  25                      <section class="esh-orders-item col-xs-1">
  26                          <a class="esh-orders-link" asp-controller="Order" asp-action="Detail" asp-route-orderId="@item.OrderNumber">Detail</a>
  27                      </section>
  28                      <section class="esh-orders-item col-xs-1">
  29                          @if (item.Status.ToLower() == "submitted")
  30                      {
  31                              <a class="esh-orders-link" asp-controller="Order" asp-action="cancel" asp-route-orderId="@item.OrderNumber">Cancel</a>
  32                          }
  33                      </section>
```

## `src/Web/ViewModels/OrderViewModel.cs` (F-21)

Lines 5–13:

```csharp
   5  public class OrderViewModel
   6  {
   7      private const string DEFAULT_STATUS = "Pending";
   8  
   9      public int OrderNumber { get; set; }
  10      public DateTimeOffset OrderDate { get; set; }
  11      public decimal Total { get; set; }
  12      public string Status => DEFAULT_STATUS;
  13      public Address? ShippingAddress { get; set; }
```

## `src/BlazorAdmin/Pages/CatalogItemPage/List.razor.cs` (F-22)

async void event handler.

Lines 43–46:

```csharp
  43      private async void DetailsClick(int id)
  44      {
  45          await DetailsComponent.Open(id);
  46      }
```

