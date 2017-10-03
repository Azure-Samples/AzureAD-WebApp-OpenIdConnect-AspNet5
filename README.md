---
services: active-directory
platforms: dotnet
author: jmprieur
---

# Integrating Azure AD into an ASP.NET Core web app

This sample shows how to build a .NET MVC web app that uses OpenID Connect to sign-in users from a single Azure Active Directory tenant using the ASP.NET Core OpenID Connect middleware.

For more information on how the protocols work in this scenario and other scenarios, see [Authentication Scenarios for Azure AD](http://go.microsoft.com/fwlink/?LinkId=394414).

## How to run this sample

To run this sample:
- Install .NET Core for Windows by following the instructions at [.NET and C# - Get Started in 10 Minutes](https://www.microsoft.com/net/core). In addition to developing on Windows, you can develop on [Linux](https://www.microsoft.com/net/core#linuxredhat), [Mac](https://www.microsoft.com/net/core#macos), or [Docker](https://www.microsoft.com/net/core#dockercmd).
- An Azure Active Directory (Azure AD) tenant. For more information on how to obtain an Azure AD tenant, see [How to get an Azure AD tenant](https://azure.microsoft.com/documentation/articles/active-directory-howto-tenant/).

### Step 1: Register the sample with your Azure Active Directory tenant

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Navigate to the Azure Active Directory blade.
3. From the sidebar, select **App registrations**.
4. Select **New application registration** and provide a friendly name for the app, app type, and sign-on URL:
   **Name**: **WebApp-OpenIDConnect-DotNet**
   **Application Type**: **Web app / API**
   **Sign-on URL**: `http://localhost:5000/signin-oidc`
   Select **Create** to register the app.
5. On the **Properties** blade, set the **Logout URL** to `http://localhost:5000/signout-oidc` and select **Save**.
6. From the Azure portal, note the following information:
   The Tenant domain: See the **App ID URI** base URL. For example: `contoso.onmicrosoft.com`
   The Tenant ID: See the **Endpoints** blade. Record the GUID from any of the endpoint URLs. For example: `da41245a5-11b3-996c-00a8-4d99re19f292`
   The Application ID (Client ID): See the **Properties** blade. For example: `ba74781c2-53c2-442a-97c2-3d60re42f403`

### Step 2: Create the sample

This sample was created from the 2.0 [dotnet new mvc](https://docs.microsoft.com/dotnet/core/tools/dotnet-new?tabs=netcore2x) template with `SingleOrg` authentication. You can create the sample from the command line or clone/download this repository:

- To create the sample from the command line, execute the following command:

  ```console
  dotnet new mvc --auth SingleOrg --client-id <CLIENT_ID_(APP_ID)> --tenant-id <TENANT_ID> --domain <TENANT_DOMAIN>
  ```
  Use the values that you recorded from the Azure portal for \<CLIENT\_ID\_(APP\_ID)>, \<TENANT\_ID>, and \<TENANT\_DOMAIN>.

- To clone/download this sample, execute the following command from your shell or command line:

  ```console
  git clone https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-aspnetcore.git
  ```

  In the **appsettings.json* file, provide values for the `Domain`, `TenantId`, and `ClientID` that you recorded earlier from the Azure portal.

If you aren't using [Visual Studio](https://www.visualstudio.com/vs/) as your development environment and created the project with the `dotnet new mvc` command:

- Add Bower to the project file:
  ```xml
  <Target Name="PreBuildScript" BeforeTargets="PrepareForBuild">
    <Exec Command="bower install" />
  </Target>
  ```
- Add `BundlerMinifer.Core` to the `ItemGroup` containing `DotNetCliToolReference` references:
  ```xml
  <DotNetCliToolReference Include="BundlerMinifier.Core" Version="2.5.357" />
  ```
- Add the `dotnet bundle` command target to bundle and minify scripts and styles in the app:
  ```xml
  <Target Name="PrePublishScript" BeforeTargets="PrepareForPublish">
    <Exec Command="dotnet bundle" />
  </Target>
  ```

### Step 3: Run the sample

Build the solution and run it.

Make a request to the app. The app immediately attempts to authenticate you via Azure AD. Sign in with your Global Administrator account.

## About The code

This sample shows how to use the OpenID Connect ASP.NET Core middleware to sign-in users from a single Azure AD tenant. The middleware is initialized in the `Startup.cs` file by passing it the Client ID of the app and the URL of the Azure AD tenant where the app is registered, which is read from the `appsettings.json` file. The middleware takes care of:
- Downloading the Azure AD metadata, finding the signing keys, and finding the issuer name for the tenant.
- Processing OpenID Connect sign-in responses by validating the signature and issuer in an incoming JWT, extracting the user's claims, and putting the claims in `ClaimsPrincipal.Current`.
- Integrating with the session cookie ASP.NET Core middleware to establish a session for the user. 

You can trigger the middleware to send an OpenID Connect sign-in request by decorating a class or method with the `[Authorize]` attribute or by issuing a challenge (see the `AccountController.cs` file):

```csharp
return Challenge(
    new AuthenticationProperties { RedirectUri = redirectUrl }, 
    OpenIdConnectDefaults.AuthenticationScheme);
```

Similarly, you can send a signout request:

```csharp
return SignOut(
    new AuthenticationProperties { RedirectUri = callbackUrl }, 
    CookieAuthenticationDefaults.AuthenticationScheme, 
    OpenIdConnectDefaults.AuthenticationScheme);
```

The middleware in this project is created as a part of the open source [ASP.NET Security](https://github.com/aspnet/Security) project.
