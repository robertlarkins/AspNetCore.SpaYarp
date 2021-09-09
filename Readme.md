# AspNetCore.SpaYarp
[![NuGet](https://img.shields.io/nuget/vpre/AspNetCore.SpaYarp)](https://www.nuget.org/packages/AspNetCore.SpaYarp/)

With  [ASP.NET Core Preview 4](https://devblogs.microsoft.com/aspnet/asp-net-core-updates-in-net-6-preview-4/#improved-single-page-app-spa-templates), the ASP.NET Core team introduced a [new experience for SPA templates](https://github.com/dotnet/aspnetcore/issues/27887).
The main benefit of this new experience is that it�s possible to start and stop the backend and client projects independently.
This is a very welcome change and speeds up the development process. But it also includes another more controversial change.
The old templates served the client application as part of the ASP.NET Core host and forwarded the requests to the SPA.
With the new templates, the URL of the SPA is used to run the application, and requests to the backend get forwarded by a built-in proxy of the SPA dev server.

_AspNetCore.SpaYarp_ uses a different approach. Instead of using the SPA dev server to forward requests to the host/backend, it uses [YARP](https://microsoft.github.io/reverse-proxy/index.html) to forward all requests that can't be handled by the host to the client application.
It works similar to the old templates, but with the advantage of the new templates to start and stop the backend and client projects independently.

The following graphic shows the differences:

![Overview](Overview.drawio.png "Overview")

To get more insights you can read my blog post [An alternative approach to the ASP.NET Core SPA templates using YARP](https://guidnew.com/en/blog/an-alternative-approach-to-the-asp-net-core-spa-templates-using-yarp).

## Running the sample

To run the sample, open the solution in Visual Studio and start the `AspNetAngularSpaYarp` project.
It reuses an existing SPA dev server if the client app is already running (started manually in a terminal or VS Code) or it starts a new one.

## Debugging

It's possible to debug the .NET code and the SPA at the same time with different editors. To use Visual Studio Code to debug the SPA, create a `launch.json` file as [described in the docs](https://code.visualstudio.com/docs/nodejs/angular-tutorial#_debugging-angular).
But instead of the URL of the SPA, the URL of the .NET host must be used.  

This is the `launch.json` used in this sample:
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "pwa-msedge",
      "request": "launch",
      "name": "Launch Edge against localhost",
      "url": "https://localhost:7113",
      "webRoot": "${workspaceFolder}"
    }
  ]
}
```


## Using AspNetCore.SpaYarp

This is what the application startup looks like:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();

// Like with Microsoft.AspNetCore.SpaProxy, a 'spa.proxy.json' file gets generated based on the values in the project file (SpaRoot, SpaProxyClientUrl, SpaProxyLaunchCommand).
// This file gets not published when using "dotnet publish".
// The services get not added and the proxy is not used when the file does not exist.
builder.Services.AddSpaYarp();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller}/{action=Index}/{id?}");

// The middlewares get only added if the SpaProxyLaunchManager service is available.
app.UseSpaYarp();

// If the SPA proxy is used, this will never be reached.
app.MapFallbackToFile("index.html");

app.Run();
```
