# DependencyInjection.Extensions.Parameterization

This package allows you to register parameters for Microsoft.Extensions.DependencyInjection.
It works like this:

```csharp
// Register
.AddSingleton<App>(pb => pb
    .Type<ServiceA>()
    .Type<ServiceB>()
    .Value("inject parameter"))
  
// Usage
public App(IService serviceA, IService serviceB, string input)
{
    serviceA // ServiceA
    serviceB // ServiceB
    input // inject parameter
}
```

It supports `Singleton`, `Scoped` and `Transient` lifetime scopes. It also provides functionality to inject `IOptions<>` with a custom key.


Here is a full example:
```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using DependencyInjection.Extensions.Parameterization;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Options;

namespace ConsoleApp1
{
    public interface IService { }
    public class ServiceA : IService { }
    public class ServiceB : IService { }
    public class ConfigSetting {
        public string Name { get; set; }
    }

    public class App
    {
        private readonly IService serviceA;
        private readonly IService serviceB;
        private readonly string input;
        private readonly IOptions<ConfigSetting> options;

        public App(IService serviceA, IService serviceB, string input, IOptions<ConfigSetting> options)
        {
            this.serviceA = serviceA;
            this.serviceB = serviceB;
            this.input = input;
            this.options = options;
        }

        public Task RunAsync(string[] args)
        {
            Console.WriteLine($"ServiceA: {serviceA}");
            Console.WriteLine($"ServiceB: {serviceB}");
            Console.WriteLine($"input: {input}");
            if (options != null)
            {
                Console.WriteLine($"options: {options.Value.Name}");
            }
            return Task.CompletedTask;
        }
    }

    public class Program
    {
        private static ServiceProvider BuildServiceProvider()
        {
            var configurationRoot = new ConfigurationBuilder()
                .SetBasePath(Path.Combine(AppContext.BaseDirectory))
                .AddJsonFile("appsettings.json", optional: false)
                .Build();

            return new ServiceCollection()
                .AddSingleton(configurationRoot) // Optional registration, only used by `.Options<>`
                .AddSingleton<ServiceA>()
                .AddSingleton<ServiceB>()

                .AddSingleton<App>(pb => pb
                    .Type<ServiceA>()
                    .Type<ServiceB>()
                    .Options<ConfigSetting>("ConfigSetting2")
                    .Value("inject parameter"))

                .BuildServiceProvider();
        }

        static async Task Main(string[] args)
        {
            var serviceProvider = BuildServiceProvider();
            var application = serviceProvider.GetRequiredService<App>();
            await application.RunAsync(args);
        }
    }
}
```
