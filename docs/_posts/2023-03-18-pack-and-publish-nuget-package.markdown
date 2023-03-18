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
![image](\images\2023-03-18-pack-and-publish-nuget-package\Configure-ClassLibrary-2023-03-18 215317.png){: width="400" }

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

It's packing time...    
 First right click the project and select the properties.

Then click on the package tab on the left pane and update all the values including the `PackageId`, `Title`, `PackageVersion`, `Author` etc.

![image](\images\2023-03-18-pack-and-publish-nuget-package\PackageProperties-2023-03-18 221813.png)

After saving again right click on the project and pack it into binary.

![image](\images\2023-03-18-pack-and-publish-nuget-package\Pack-2023-03-18 222013.png)

It will generate the binary files as shown.

![image](\images\2023-03-18-pack-and-publish-nuget-package\packed-binary-2023-03-18 222827.png)

Now its time to publish the package to the [Nuget repository](https://nuget.org).

If you don't have an account at nuget.org, you can create one for free and login to your account.

Once logged in, click the *Upload* tab and browse to the binary we have built and packed in the last step and upload it.

![image](\images\2023-03-18-pack-and-publish-nuget-package\NuGet Upload-2023-03-18 223723.png)

Now if you go to the nuget public packages, in your *Manage Nuget Packages*, you will find the NuGet package published and ready to be consumed in your projects.

We try to consume the same package we created in the upcoming blog for Logging in Console app....

***Happy coding....***