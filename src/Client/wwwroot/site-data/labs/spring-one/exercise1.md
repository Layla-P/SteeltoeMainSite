﻿[vs-new-proj]: /site-data/labs/spring-one/images/vs-new-proj.png "New visual studio web project"
[vs-name-proj]: /site-data/labs/spring-one/images/vs-configure-project.png "Name project"
[vs-create-proj]: /site-data/labs/spring-one/images/vs-create-project.png "Create an api project"
[vs-add-endpointcore]: /site-data/labs/spring-one/images/vs-add-endpointcore.png "Endpointcode nuget dependency"
[vs-add-dynamiclogger]: /site-data/labs/spring-one/images/vs-add-dynamiclogger.png "Dynamiclogger nuget dependency"
[vs-add-tracingcore]: /site-data/labs/spring-one/images/vs-add-tracingcore.png "TracingCode nuget dependency"
[vs-run-application]: /site-data/labs/spring-one/images/vs-run-application.png "Run the project"
[run-weatherforecast]: /site-data/labs/spring-one/images/weatherforecast-endpoint.png "Weatherforecast endpoint"
[health-endpoint]: /site-data/labs/spring-one/images/health-endpoint.png "Health endpoint"
[info-endpoint]: /site-data/labs/spring-one/images/info-endpoint.png "Info endpoint"
[trace-log]: /site-data/labs/spring-one/images/trace-log.png "Trace logs"

[home-page-link]: /labs/spring-one
[exercise-1-link]: /labs/spring-one/exercise1
[exercise-2-link]: /labs/spring-one/exercise2
[exercise-3-link]: /labs/spring-one/exercise3
[exercise-4-link]: /labs/spring-one/exercise4
[exercise-5-link]: /labs/spring-one/exercise5

## Getting to know Steeltoe

### Goal

Understand how Steeltoe is distributed (Nuget) and how one adds components into an existing application.

### Expected Results

Begin building an API that will be enhanced with more components in the next exercise(s).

### Get Started

Let's start by creating a brand new .NET Core webapi project. If you're using Visual Studio, choose `File > New > Project`

|![vs-new-proj] Choose ASP.NET Core Web Application from the default templates. |![vs-name-proj] The default project name WebApplication1 will be used throughout, but you can rename.|![vs-create-proj] Choose an application type of API, everything else can keep its default value.|
|:--|

Or if you prefer the dotnet cli:

```powershell
dotnet new webapi -n WebApplication1
cd WebApplication1
```

Once created, open the new project in your IDE of choice (we will be using Visual Studio throughout this lab). The first action is to bring in the Steeltoe packages to the app. You can do this by right clicking on the project name in the solution explorer and choose `Manage NuGet packages...`. In the package manger window choose `Browse`, search for `Steeltoe.Management.Endpointcore`, and install.
	
![vs-add-endpointcore]

Then search for the `Steeltoe.Extensions.Logging.DynamicLogger` package and install.

![vs-add-dynamiclogger]

Finally the `Steeltoe.Management.TracingCore` package and install.

![vs-add-tracingcore]

You could have done all this in the cli:

```powershell
dotnet add package Steeltoe.Management.Endpointcore
dotnet add package Steeltoe.Extensions.Logging.DynamicLogger
dotnet add package Steeltoe.Management.TracingCore
```

Steeltoe features are broken up into packages, giving you the option to only bring in and extend the dependencies needed. As we implement each package within the application we'll discuss why these packages were chosen.

Open `Program.cs` and append the adding statements to the host builder. Visual Studio should prompt to add the `using Steeltoe.Management.Endpoint` direction.

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
	Host.CreateDefaultBuilder(args)
		.ConfigureWebHostDefaults(webBuilder => {
			webBuilder.UseStartup<Startup>();
		})

		//Steeltoe actuator packages
		.AddHealthActuator()
		.AddInfoActuator()
		.AddLoggersActuator()
		;
```

We've implemented 3 features within the application by adding these actuators.
- The health actuator will add a new endpoint at `/actuators/health`. Internally this function uses .NET's IHealthContributor to "decide" if everything is reporting good health and responds with HTTP 200 status. Also within the response body there is a json formatted message to accomodate a deeper check that specfic platforms like Cloud Foundry and Kubernetes do.
- The info actuator adds a new endpoint at `/actuators/info`. This function gathers all kinds of information like versioing information, select package information, and DLL info. Everything is formatted as json and included in the response.
- The loggers actuator enables enhanced log message details via ILogger.

Now open `Startup.cs` and add distributed tracing features, Visual Studio should prompt you to add the `using Steeltoe.Management.Tracing` direction.

```csharp
public void ConfigureServices(IServiceCollection services) {
	services.AddControllers();

	//Steeltoe distributed tracing
	services.AddDistributedTracing(Configuration);
}
```

With the addition of distributed tracing option, under the covers Steeltoe uses the OpenTelemetry specification to generate spans and traces throughout the application, as requests are recieved. No additional configuration is needed.

Also having the combination of the logging actuator and distributed tracing implemented, Steeltoe will automatically append the application name, span Id, and trace Id on log messages when possible. This can be very handy when debugging a specific happening and error in production.

To see the trace logging in action, lets add a log message in `Controllers\WeatherForecastController.cs` controller. Append the below message as the first line with the 'Get' function.

```csharp
[HttpGet]
public IEnumerable<WeatherForecast> Get() {
	//Testing Steeltoe logging with distributed tracing
	_logger.LogInformation("Hi there");
		
	//...
}
```

With the packages implemented in host builder, distributed tracing activated, and a sample log message being written to console, we are ready to see everything in action. Start the application by clicking the `Debug > Start Debugging` top menu item.

![vs-run-application]

Or use the dotnet cli:
```powershell
dotnet run
```

Once started your default browser should open and automatically load the weather forecast endpoint.

![run-weatherforecast]

Let's look at the health endpoint. Replace `WeatherForecast` with `/actuators/health` in the browser address bar. The health page will load with json formatted info.

![health-endpoint]

As we discussed above, the page loaded with a status of 200 and output information to help application platforms gain a deeper "knowledge" of app health.

Now navigate to the info endpint by replacing `health` with `info` in the address bar.

![info-endpoint]

Relevant app info is output, with json formatting.

Finally lets look at the log message that was written by going back to Visual Studio (keep the app running) and locate the Output window. Choose `Webapplication1 - ASP.NET Core Web Server` in the "from" dropdown and scroll to the bottom of the log. (If you're using the dotnet cli, the logs should be output in the same window you ran the app.)

![trace-log]

Locate the "Hi there" log message. Notice the additional information prepended to the messsage.
- The first item is the application's name
- Second is the OpenTelemetry generated span id
- Third is the OpenTelemetry generated trace id

### Summary

These are the basics of any cloud ready microservice. Logging and debugging are significantly different than a traditional IIS environment. But! A developer shouldn't be spending tons of time coding these buildplate-type things. Heeelllo Steeltoe!

|[<< Back to Introduction][home-page-link]|[Next Exercise >>][exercise-2-link]|
|:--|--:|
