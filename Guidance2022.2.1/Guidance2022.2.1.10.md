# Release notes for version 2022.2.1.10

# !!! This is not yet released!!!

- Guidance startup message.
- NSwag and code generation.
- Change Localhost to Development
- Add Csp
  - Yarp and micro-services
  - Blazor => todo.
- Remove Server info in header.
- Fix Blazor Program.cs issue => MSAL config.

## Guidance startup message.

Now when a new Blank solution is created the message in the window will inform that the Guidance x is ready to be used.<br>
Before it was not the case and introduces doubt if the Guidance is activated or not...

## Change Localhost to Development.

.NET by default implements information about the environment based on the IHostEnvironment.

4 extension methods exist.
- IsDevelopment()
- IsStaging()
- IsProduction()
- IsEnvironment("name of the environment").

The Guidance is not fixing the name of the environment but for the local development machine.<br>
To be compliant with the standard and some code generated in .NET examples the Localhost has been replaced by Development!

=> It means that you will have to update your current project
appsettings.Localhost.json to appsettings.Development.json

Change the ASPNETCORE_ENVIRONMENT to Development in launchSettings.json

```json

{
  "$schema": "http://json.schemastore.org/launchsettings.json",
  "profiles": {
    "Kestrel": {
      "commandName": "Project",
      "launchBrowser": true,
      "launchUrl": "swagger/facade",
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": ==> "Development" <>==
      },
      "dotnetRunMessages": true,
      "applicationUrl": "https://localhost:44349"
    }
  }
}

```

Finally there is also the nswag files where the aspNetCoreEnvironment property must be changed from Localhost to Development.

```json
{
  "runtime": "Net60",
  "defaultVariables": null,
  "documentGenerator": {
    "aspNetCoreToOpenApi": {
      "project": "",
      ...
      ==> "aspNetCoreEnvironment": "Development", <==
      "createWebHostBuilderMethod": null,
      "startupType": null,
      "allowNullableBodyParameters": true,
      "output": null,
      "outputType": "Swagger2",
      ],
      "assemblyConfig": "",
      "referencePaths": [],
      "useNuGetCache": false
    }
  },
  "codeGenerators": {
    "openApiToCSharpClient": {
    }
  }
}


```


## Add Csp

If you want to protect your service and Blazor application from exploit (via the browser) you have to define some headers and also manage the Content-Security-Policy.<br>
This is an endpoint added by your service that will be used by your browser to inform what is allowed or not to do!<br>
This file will block for example inline scripts and protect you against script injection...

To define this I use the nuget package NetEscapades.AspNetCore.SecurityHeaders.

You can find more info [here](https://github.com/andrewlock/NetEscapades.AspNetCore.SecurityHeaders).

A new class is generated by the Guidance and define the default settings for a Yarp and a service. There are differences because a Yarp is exposed to the browser and this is not the target of a service (behind the reverse proxy).

To use this definition in the progam.cs, you will find the following code.

```csharp

	var app = builder.Build();

	==> app.UseSecurityHeaderCSP(Configuration, builder.Environment); <==

	app.UseRouting();


```

This code will define the common rules. Up to you to adapt it based on the web pages you display.<br>

Currently we have 4 different pages in our Yarp front end service.
1. The root with the .NET Welcone page.
2. The NSwag Swagger pages.
3. The HealthCheck pages.
4. The Hangfire dashboard.

Each pages have some scripts and styles and images and etc...

By design the scripts generated are not allowed and a sha256 hash is generated. By addind this in the file, you authorize the browser to accept those script or css or...<br>

### NSwag issue.

This is fine but the current Swagger generator used by the Guidance, NSwag generates an html page containing an inline script which is containing the list of services. As this is adapted each time a new service is created we have no possibility to fix this definitively.<br>

Use of Report in Development...




## Remove Server info in header.

To improve the opacity regarding the information we provide, one information which is sent by a service is the server info in the headers.<br>
For Kestrel, the headers will contains the information that the host is powered by Kestrel. This information is sensitive because it gives the fact that we are running a .NET application and with Kestrel.<br>
Any vulnerabilities based on the this information can be exploited by hackers.

This is why in the startup the following code has been added.

```csharp

	var builder = WebApplication.CreateBuilder(args);
	// Remove sensitive information that can be exploit by hackers.
	==> builder.WebHost.ConfigureKestrel(options => options.AddServerHeader = false); <==
	builder.Host

```
