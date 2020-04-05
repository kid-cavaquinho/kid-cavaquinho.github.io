---
layout:     post
title:      Using ASP.NET Core configuration mechanism with valid objects 
date:       2018-09-21 10:42:00
summary:    "Use Microsoft.Extensions.Configuration effectively"
categories: options-pattern validation
---

Applications often demand configuration values, those can either be connection strings, logging level settings or specific tokens, for example. While developing with .NET Core framework the configuration mechanism is based at [__Microsoft.Extensions.Configuration__](https://github.com/aspnet/Configuration). This is a replacement for ```System.Configuration``` namespace. In this post, I will try to explain how to use the first effectively.

## Host

This journey starts with a configuration and launch of a [host](https://docs.microsoft.com/en-gb/aspnet/core/fundamentals/host/index?view=aspnetcore-2.1). The host is responsible for the application startup, the request processing pipeline and provides extensibility to manage configurations. In the case of ASP.NET Core the host is based on [```IWebHostBuilder```](https://docs.microsoft.com/en-gb/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder?view=aspnetcore-2.1).

## IWebHostBuilder

Since ASP.NET Core 2.0 the host is created with pre-configured defaults at the application entry point. ```Program.cs``` contains a ```WebHost.CreateDefaultBuilder``` method that automatically produces a ```IWebHostBuilder``` instance as you will notice [here](https://github.com/aspnet/MetaPackages/blob/master/src/Microsoft.AspNetCore/WebHost.cs#L148). ```CreateDefaultBuilder``` executes other tasks among which are, use Kestrel as the web server, register the configuration sources for file providers (JSON) with multiple environment support, register user secrets for the current assembly, register environment variables and command-line arguments if not null or even logging configuration.

The configurations can be overridden through ```ConfigureAppConfiguration```, see an example below.

{% highlight csharp lineanchors %}
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
    .ConfigureAppConfiguration((builderContext, config) =>
    {
        config.Sources.Clear();
        config.AddIniFile("appsettings.ini", optional: true, reloadOnChange: true);
    })
    .UseStartup<Startup>();
{% endhighlight %}

## Configuration providers

There is flexibility to choose several types of default configuration providers, such as:

- File configuration (JSON, XML, INI)
- Environment variables
- Command-line (command-line arguments)
- Memory (In-memory collections)
- [Azure Key Vault](https://docs.microsoft.com/en-gb/aspnet/core/security/key-vault-configuration?view=aspnetcore-2.1)

You can create your own custom configuration providers using the interfaces ```IConfigurationSource``` and ```IConfigurationProvider```.

## Specific environments

Multiple files to handle specific environment configurations can be easily added. In my opinion, this enables a clean and better organized configuration, you can add configuration files for specific environments of the application deployment lifecycle in your root folder.

This is possible because of the [following](https://github.com/aspnet/MetaPackages/blob/master/src/Microsoft.AspNetCore/WebHost.cs#L170). The environment is read from the __ASPNETCORE_ENVIRONMENT__ variable that is set in the ```launchSettings.json``` file.

## Overriding values

In the screenshot there is a file with no environment value: ```appsettings.json```. This file should hold default values that do not change based on environment. Other values present in the environment specific JSON files are overridden while the application starts.

It's important to mention that despite the chosen provider all your configuration sources will come down to flatten __key/value__ pairs.

## Reading values

Currently the ```appsettings.Development.json``` contains the following section.

{% highlight json lineanchors %}
{
    "MySettings": {
        "Setting1": "0123456789",
        "Setting2": "what-an-awesome-value"
    }
}
{% endhighlight %}

You should __never store sensitive information in your configuration files__. Take advantage of [user secrets](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets), for example.

Read a value in ```HomeController``` can be achieved by using ```IConfiguration``` interface.
{% highlight csharp lineanchors %}
public class HomeController : Controller
{
    private readonly IConfiguration _configuration;

    public HomeController(IConfiguration configuration)
    {
         _configuration = configuration;
    }

    public IActionResult Index()
    {
        var setting = _configuration.GetValue<string>("MySettings:Setting1");
        return View();
    }
}
{% endhighlight %}

The example displays the extension method ```GetValue<T>``` extracting the value with the specific key from the JSON file. One could also simply access the property directly.

{% highlight csharp lineanchors %}
var setting = _configuration["MySettings:Setting1"];
{% endhighlight %}

Make notice that the __keys__ are __case-insensitive__ and follow the hierarchy specified on the JSON file using delimiter character  __":"__. See [documentation](https://docs.microsoft.com/en-gb/dotnet/api/microsoft.extensions.configuration.iconfiguration?view=aspnetcore-2.1) of ```IConfiguration```.

## Reading values with strongly typed objects

I've created a strongly typed configuration class: ```MySettings.cs``` with a structure that matches the same section in the JSON configuration file displayed previously.

{% highlight csharp lineanchors %}
public class MySettings
{
    public int Setting1 { get; set; }

    public string Setting2 { get; set; }
}
{% endhighlight %}

Using ```MySettings``` is possible with the extension method ```Bind()```.

{% highlight csharp lineanchors %}
var mySettings = new MySettings();
_configuration.GetSection("MySettings").Bind(mySettings);
{% endhighlight %}

Quite often used is the [```Options```](https://docs.microsoft.com/en-gb/aspnet/core/fundamentals/configuration/options) pattern. This leverages strongly typed objects to represent a group of related settings described in the used sources.

The ```Startup``` class contains a ```ConfigureServices``` method where you should register your desired configuration. This is provided through the method ```services.Configure<T>```.

{% highlight csharp lineanchors %}
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<CookiePolicyOptions>(options =>
        {
            options.CheckConsentNeeded = context => true;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Latest);
        services.Configure<MySettings>(Configuration.GetSection("MySettings"));
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        // ...
    }
}
{% endhighlight %}

Read a value in ```HomeController``` can be achieved by using ```IOptions<T>``` in the constructor, in this example ```T``` of type ```MySettings```.

{% highlight csharp lineanchors %}
public class HomeController : Controller
{
    private readonly MySettings _mySettings;

    public HomeController(IOptions<MySettings> mySettings)
    {
        _mySettings = mySettings.Value;
   }

    public IActionResult Index()
    {
        ViewData["Setting"] = _mySettings.Setting1;

        return View();
    }
}
{% endhighlight %}

## Say no to IOptions dependencies

One could say that the classes that access your configurations should not be dependent on ```IOptions<T>```, but instead on your configuration classes itself. To achieve this explicitly register ```MySettings``` object on the ```ConfigureServices``` method as a singleton.

{% highlight csharp lineanchors %}
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<CookiePolicyOptions>(options =>
        {
            options.CheckConsentNeeded = context => true;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Latest);

        services.Configure<MySettings>(Configuration.GetSection("MySettings"));
        services.AddSingleton(resolver => resolver.GetRequiredService<IOptions<MySettings>>().Value);
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        // ...
    }
}
{% endhighlight %}

Read the value in ```HomeController``` does not dependend on ```IOptions<T>``` anymore.

{% highlight csharp lineanchors %}
public class HomeController : Controller
{
    private readonly MySettings _mySettings;

    public HomeController(MySettings mySettings)
    {
        _mySettings = mySettings;
    }

    public IActionResult Index()
    {
        ViewData["setting"] = _mySettings.Setting1;

        return View();
    }
}
{% endhighlight %}

__Flexibility__ is a key element regarding the configuration mechanism in .NET Core.

## Validation

There are a few recurrent topics I've found in projects that should require extra attention. A binding of a property fails because of a typo in your configuration source or class when using strongly typed objects. A value is removed from the configuration sources, are just some to take in consideration. __I want to accomplish a versatile direction to validate settings.__

The interface ```ISartupFilter``` allows you to configure middleware pipeline from a service resolved from the dependency injection container. ```ISartupFilter``` exists in the ```Microsoft.AspNetCore.Hosting.Abstractions``` package. Read more detailed information in the [documentation](https://docs.microsoft.com/en-gb/aspnet/core/fundamentals/startup?view=aspnetcore-2.1#extend-startup-with-startup-filters).

I've created the following startup filter, relying on a collection of ```IValidatable``` settings. This allows validation of configuration objects during the initialization of the application. When the result of the validation operation is not true an invalid operation exception is logged with the setting name, validation results and thrown.

{% highlight csharp lineanchors %}
public class SettingValidationStartupFilter : IStartupFilter
{
    private readonly ILogger<SettingValidationStartupFilter> _logger;
    private readonly IEnumerable<IValidatable> _settings;

    public SettingValidationStartupFilter(ILogger<SettingValidationStartupFilter> logger, 
    IEnumerable<IValidatable> settings)
    {
        _logger = logger;
        _settings = settings;
    }

    public Action<IApplicationBuilder> Configure(Action<IApplicationBuilder> next)
    {
        foreach (var setting in _settings)
        {
            var (results, valid) = setting.Validate();

            if (valid)
            {
                continue;
            }

            var settingName = setting.GetType().Name;
            var message = $"Invalid {settingName} configuration. {string.Join(",", results)}";
            var invalidOperationException = new InvalidOperationException(message);

            _logger.LogError(invalidOperationException, message);

            throw invalidOperationException;
        }

        return next;
    }
}
{% endhighlight %}

The ```IValidatable``` interface returns a boolean representing the success or not of the operation and a collection of validation results.

{% highlight csharp lineanchors %}
public interface IValidatable
{
    (IEnumerable<string> results, bool valid) Validate();
}
{% endhighlight %}

To keep the validation extensible and because we can validate our settings in many different ways, I've created an abstract class depending on [DataAnnotations](https://docs.microsoft.com/en-gb/dotnet/api/system.componentmodel.dataannotations?view=netcore-2.1) that inherits the ```IValidatable``` interface.

{% highlight csharp lineanchors %}
public abstract class DataAnnotationSettingValidation : IValidatable
{
    public (IEnumerable<string> results, bool valid) Validate()
    {
        IList<ValidationResult> validationResults = new List<ValidationResult>();

        if (Validator.TryValidateObject(this, 
        new ValidationContext(this, null, null), validationResults, validateAllProperties: true))
        {
            return (results: Enumerable.Empty<string>(), valid: true);
        }

        return (results: validationResults.Select(results => results.ErrorMessage), valid: false);
    }
}
{% endhighlight %}

```MySettings``` should inherit from ```DataAnnotationSettingValidation```. In this example I've used a required attribute in a property.
{% highlight csharp lineanchors %}
public sealed class MySettings : DataAnnotationSettingValidation
{
    public int Setting1 { get; set; }

    [Required]
    public string Setting2 { get; set; }
}
{% endhighlight %}

Last but not least, there is a need to register the startup filter and the desired settings as ```IValidatable``` in the ```Startup``` class on the ```ConfigureServices``` method.

{% highlight csharp lineanchors %}
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<CookiePolicyOptions>(options =>
        {
            options.CheckConsentNeeded = context => true;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Latest);

        services.AddTransient<IStartupFilter, SettingValidationStartupFilter>();

        services.Configure<MySettings>(Configuration.GetSection("MySettings"));
        services.AddSingleton(resolver => resolver.GetRequiredService<IOptions<MySettings>>().Value as MySettings);
        services.AddSingleton(resolver => resolver.GetRequiredService<IOptions<MySettings>>().Value as IValidatable);
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        // ...
    }
}
{% endhighlight %}

---

## TLDR

[Microsoft.Extensions.Configuration](https://github.com/aspnet/Configuration) is open-source, flexible and easy to use. In my opinion, definitely a step forward compared to ```System.Configuration```.

There are many available default providers and ways to read configuration data in your applications. I favor usage of strongly typed objects with ```Options``` pattern. In the presented examples, all your configuration objects can be validated during the startup of the application, preventing it to load with wrong values or errors.

Make sure you __do not store sensitive information or passwords in your configuration files.__

I've wrote a [sample application](https://github.com/antao/learning-aspnet-core-configuration) to support this post. Furthermore there is great [documentation](https://docs.microsoft.com/en-gb/aspnet/core/fundamentals/configuration/) provided by Microsoft.
