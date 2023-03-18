---
layout: post
title:  "Pack and publish a NuGet package"
date:   2023-03-18 06:28:00 +0530
categories: jekyll update
# published: false
---

Lets publish a simple demo public NuGet package  to get the feel of how to get it done

> **Prerequisites**  
>Visual Studio 2022  
>Dotnet 7 Runtime

As a sample package to publish, we would create a logger package that can log both custom way and using the Microsoft Logging Extension `ILogger` logging.

Lets create a new Class Library using Visual Studio   
![image](\images\New folder\Configure-ClassLibrary-2023-03-18 215317.png){: width="250" }

The solution would like this:    
![image](\images\2023-03-18-pack-and-publish-nuget-package\SolutionStructure-2023-03-18 220026.png)

With the `Logger` class and implementations:

```csharp
using Microsoft.Extensions.Logging;

namespace DemoPackage
{
    public class Logger<T>
    {
        public void LogInformation<T>(string message)
        {
            Console.WriteLine("{0}.{1} information logged to console from DemoLogger",typeof(T), message);
        }
        public void LogDebug<T>(string message)
        {
            Console.WriteLine("{0}.{1} debug logged to console from DemoLogger", typeof(T), message);
        }
    }
    public interface IDemoLoggerFactory
    {
        public Logger<T> CreateCustomLogger<T>();
        public ILogger<T> CreateILogger<T>();
    }
    public class DemoLoggerFactory : IDemoLoggerFactory
    {

        public Logger<T> CreateCustomLogger<T>()
        {
            return new Logger<T>();
        }

        public ILogger<T> CreateILogger<T>()
        {
            var loggerFactory = LoggerFactory.Create(configure => configure.AddConsole());
            var logger = loggerFactory.CreateLogger<T>();
            return logger;
        }
    }


}
```