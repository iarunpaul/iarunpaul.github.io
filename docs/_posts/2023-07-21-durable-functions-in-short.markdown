---
layout: post
title:  "Azure Durable Functions in short"
date:   2023-07-21 03:46:00 +0530
categories: jekyll update
published: true
---
<div style="text-align: center;">
  <img src="\images\2023-0721-durable-functions-in-short\Title.jpg" style="width: 500px; height: auto;">
</div>

---

<br>
<br>

Azure Durable functions are meant for handling workflows that takes longer to finish.
It also helps to make the run stateful, so that it can track the status of a run and restart from where it stops, if at all.

And you guessed it right, that's why we assign always a storage account in association with a durable function when you deploy the function in Azure.
The storage account holds the data to determine the state of the system.
Thankfully you don't really need to configure literally anything at all when you start to develop your solution.

Visual Studio makes it even easier with the Azure function boilerplate templates and in-built tools to locally develop and run a function.

No wait, just jump in and create a project using Visual Studio.

> **Prerequisites**  
>Visual Studio 2022  
>Dotnet 6 Runtime
>Azure SDK

Use function boiler template in Visual Studio

---
  
<br>
 
![image](\images\2023-0721-durable-functions-in-short\Screenshot1.png){: width=600px }

<br>


<br>


Select the option of Durable Function.

---

<br>
 
![image](\images\2023-0721-durable-functions-in-short\Screenshot3.png){: width=600px }

<br>




Check the `Azurite` tool when you create your project. This will help to simulate the Storage Account in local, when you develop the function.

---
<br>
 
![image](\images\2023-0721-durable-functions-in-short\Screenshot2.png){: width=600px }

<br>

Other options:
- StorageSimulator
- Use connection string of a Storage Account itself.



<br>
The solution looks like similar to this 👇

---
 <br> 
![image](\images\2023-0721-durable-functions-in-short\Screenshot4.png){: width=600px }


<br>

The local.settings.json

---

<br>
```csharp
{
    "IsEncrypted": false,
    "Values": {
        "AzureWebJobsStorage": "UseDevelopmentStorage=true",
        "FUNCTIONS_WORKER_RUNTIME": "dotnet"
    }
}

```
`"AzureWebJobsStorage": "UseDevelopmentStorage=true"` is the line that compensates for the local Storage Account (lack of Storage Account 😊).

<br>

The Function1.cs has got 3 functions.

---

<br>
```csharp
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.DurableTask;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs.Host;
using Microsoft.Extensions.Logging;

namespace FunctionApp1
{
    public static class Function1
    {
        [FunctionName("Function1")]
        public static async Task<List<string>> RunOrchestrator(
            [OrchestrationTrigger] IDurableOrchestrationContext context)
        {
            var outputs = new List<string>();

            // Replace "hello" with the name of your Durable Activity Function.
            outputs.Add(await context.CallActivityAsync<string>(nameof(SayHello), "Tokyo"));
            outputs.Add(await context.CallActivityAsync<string>(nameof(SayHello), "Seattle"));
            outputs.Add(await context.CallActivityAsync<string>(nameof(SayHello), "London"));

            // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
            return outputs;
        }

        [FunctionName(nameof(SayHello))]
        public static string SayHello([ActivityTrigger] string name, ILogger log)
        {
            log.LogInformation("Saying hello to {name}.", name);
            return $"Hello {name}!";
        }

        [FunctionName("Function1_HttpStart")]
        public static async Task<HttpResponseMessage> HttpStart(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequestMessage req,
            [DurableClient] IDurableOrchestrationClient starter,
            ILogger log)
        {
            // Function input comes from the request content.
            string instanceId = await starter.StartNewAsync("Function1", null);

            log.LogInformation("Started orchestration with ID = '{instanceId}'.", instanceId);

            return starter.CreateCheckStatusResponse(req, instanceId);
        }
    }
}

```

Start with the *Starter* function with attribute `[FunctionName("Function1_HttpStart")]`.
This is a normal `Http` triggered azure function, and is used to invoke the *Orchestrator* which is actually the ***Durable*** function or we can say the brain of the durable function logic.

The starter function has a special parameter starter of type `IDurableOrchestrationClient` and has got an attriibute `[DurableClient]` and this *Starter* is used as a durable client to call the *Orchestrator* function.

---

```csharp
// Function input comes from the request content.
            string instanceId = await starter.StartNewAsync("Function1", null);
```
<br>
The first parameter is the name of the function to be called. Here the boiler plate *Orchestrator* name is `Function1`.

Then we pass a `null` value, if the instance Id can be randomized. This case an instance value is auto-generated and passed along to the **Durable** function.


I can also pass the request parameter, if I want to.


I have re-written the code a little bit.
And kept the different types of functions a bit organized.
I have moved the HTTP functions to a `HttpFunctions.cs` and folderized.

---
<br>
```aspx-csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs.Extensions.DurableTask;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

namespace DurableVideoProcessor.HttpFunctions
{
    public static class HttpFunction
    {

        [FunctionName("HttpStarterFunction")]
        public static async Task<IActionResult> HttpStart(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequest req,
            [DurableClient] IDurableOrchestrationClient starter,
            ILogger log)
        {
            var video = req.GetQueryParameterDictionary()["video"];
            // Function input comes from the request content.
            string instanceId = await starter.StartNewAsync("DurableVideoProcessorOrchestrator", null, video);

            log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

            return starter.CreateCheckStatusResponse(req, instanceId);
        }
    }
}

```
<br>
Same goes for `OrchestratorFunction.cs` 

---
<br>
```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.DurableTask;

namespace DurableVideoProcessor.OrchestratorFunctions
{
    public static class OrchestratorFunction
    {
        [FunctionName(nameof(DurableVideoProcessorOrchestrator))]
        public static async Task<List<string>> DurableVideoProcessorOrchestrator(
        [OrchestrationTrigger] IDurableOrchestrationContext context)
        {
            var outputs = new List<string>();

            // Replace "hello" with the name of your Durable Activity Function.
            outputs.Add(await context.CallActivityAsync<string>(nameof(ActivityFunctions.ActivityFunction.SayHello), "Tokyo"));
            outputs.Add(await context.CallActivityAsync<string>(nameof(ActivityFunctions.ActivityFunction.SayHello), "Seattle"));
            outputs.Add(await context.CallActivityAsync<string>(nameof(ActivityFunctions.ActivityFunction.SayHello), "London"));

            // returns ["Hello Tokyo!", "Hello Seattle!", "Hello London!"]
            return outputs;
        }
    }


}

```
<br>
Similarly, for `ActivityFunctions` functions:

---
<br>

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.DurableTask;
using Microsoft.Extensions.Logging;

namespace DurableVideoProcessor.ActivityFunctions
{
    public static class ActivityFunction
    {

        [FunctionName(nameof(SayHello))]
        public static string SayHello([ActivityTrigger] string name, ILogger log)
        {
            log.LogInformation($"Saying hello to {name}.");
            return $"Hello {name}!";
        }
    }
}
```
<br>
Solution looks much cleaner now.

---
<br>

![image](\images\2023-0721-durable-functions-in-short\Screenshot5.png)
<br>

The solution looks like similar to this 👆

---
  
<br>
 Let's simulate now some realistic video processing workflows. 

We are going to define some simple workflow activity functions, related to the video processing....

Let's update the activity functions.

Rename the Activity Functions class as `ProcessVideo` and add 3 activities as workflow functions.
<br>

---
<br>

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.DurableTask;
using Microsoft.Extensions.Logging;
using System;
using System.IO;
using System.Threading.Tasks;

namespace DurableVideoProcessor.ActivityFunctions
{
    public static class ProcessVideo
    {

        [FunctionName(nameof(Transcode))]
        public static async Task<string> Transcode([ActivityTrigger] string inputVideo, ILogger log)
        {
            log.LogInformation($"Transcoding {inputVideo}.");
            // Simulte transcoding
            await Task.Delay(5000);
            return $"{Path.GetFileNameWithoutExtension(inputVideo)}-transcoded.mp4";
        }

        [FunctionName(nameof(Thumbnail))]
        public static async Task<string> Thumbnail([ActivityTrigger] string inputVideo, ILogger log)
        {
            log.LogInformation($"Thumbnailing {inputVideo}.");
            // Simulte thumbnailing
            await Task.Delay(5000);
            return $"{Path.GetFileNameWithoutExtension(inputVideo)}-thumbnailed.mp4";
        }

        [FunctionName(nameof(PrependIntro))]
        public static async Task<string> PrependIntro([ActivityTrigger] string inputVideo, ILogger log)
        {
            var introLocation = Environment.GetEnvironmentVariable("IntroLocation");
            log.LogInformation($"Prepending intro into {introLocation} to {inputVideo}.");
            // Simulte transcoding
            await Task.Delay(5000);
            return $"{Path.GetFileNameWithoutExtension(inputVideo)}-withintro.mp4";
        }
    }
}
```
<br>
Let's build the code for `Orchestrator` now....

---
<br>

```csharp
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.DurableTask;
using Microsoft.Extensions.Logging;
using Microsoft.Identity.Client;

namespace DurableVideoProcessor.OrchestratorFunctions
{
    public static class OrchestratorFunction
    {
        [FunctionName(nameof(DurableVideoProcessorOrchestrator))]
        public static async Task<object> DurableVideoProcessorOrchestrator(
        [OrchestrationTrigger] IDurableOrchestrationContext context, ILogger log)
        {            
            log = context.CreateReplaySafeLogger(log);
            var videoLocation = context.GetInput<string>();

            log.LogInformation("Orchestrator starts the transcode and goes to sleep...");
            var transcodelLocation = await context.CallActivityAsync<string>(nameof(ActivityFunctions.ProcessVideo.Transcode), videoLocation);
           
            log.LogInformation("Orchestrator starts the thumbnailing and goes to sleep...");
            var thumpnailLocation = await context.CallActivityAsync<string>(nameof(ActivityFunctions.ProcessVideo.Thumbnail), transcodelLocation);
            
            log.LogInformation("Orchestrator starts the intro preluding and goes to sleep...");
            var introLocation = await context.CallActivityAsync<string>(nameof(ActivityFunctions.ProcessVideo.PrependIntro), transcodelLocation);

            return new
            {
                Transcoded = transcodelLocation,
                Thumpnail = thumpnailLocation,
                Intro = introLocation
            };
        }
    }


}
```
<br>
The Starter function looks almost the same...

---
<br>
```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs.Extensions.DurableTask;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Azure.WebJobs;
using System.Threading.Tasks;
using Microsoft.Extensions.Logging;

namespace DurableVideoProcessor.HttpFunctions
{
    public static class HttpFunction
    {

        [FunctionName("HttpStarterFunction")]
        public static async Task<IActionResult> HttpStart(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequest req,
            [DurableClient] IDurableOrchestrationClient starter,
            ILogger log)
        {
            var inputVideo = req.GetQueryParameterDictionary()["video"];
            // Function input comes from the request content.
            string instanceId = await starter.StartNewAsync("DurableVideoProcessorOrchestrator", null, inputVideo);

            log.LogInformation($"Started orchestration with ID = '{instanceId}'.");

            return starter.CreateCheckStatusResponse(req, instanceId);
        }
    }
}

```
<br>

Okay...It's time to run the function in Visual Studio.

---
<br>

```csharp

Azure Functions Core Tools
Core Tools Version:       4.0.5198 Commit hash: N/A  (64-bit)
Function Runtime Version: 4.21.1.20667

[2023-07-21T19:32:50.122Z] Found C:\My\Code\source\repos\Azure Functions\DurableVideoProcessor\DurableVideoProcessor.csproj. Using for user secrets file configuration.

Functions:

        HttpStarterFunction: [GET,POST] http://localhost:7153/api/HttpStarterFunction

        DurableVideoProcessorOrchestrator: orchestrationTrigger

        PrependIntro: activityTrigger

        Thumbnail: activityTrigger

        Transcode: activityTrigger

For detailed output, run func with --verbose flag.
[2023-07-21T19:32:59.160Z] Host lock lease acquired by instance ID '000000000000000000000000A8EB91B9'.
```
<br>
Thankfully, it gives the trigger url to start the function 

---
<br>

```c
 curl  http://localhost:7153/api/HttpStarterFunction?video=http://this-is-a-test.com

```
<br>
You can see the output terminal logs, saying the execution of the workflow in order....


---
<br>
```c
[2023-07-21T19:38:09.654Z] Executing 'HttpStarterFunction' (Reason='This function was programmatically called via the host APIs.', Id=b8694f9a-bb71-408e-a467-80225a2b7171)
[2023-07-21T19:38:10.312Z] Started orchestration with ID = '3f1b0acbe5fa4f4b9214da00e2b5b244'.
[2023-07-21T19:38:10.387Z] Executed 'HttpStarterFunction' (Succeeded, Id=b8694f9a-bb71-408e-a467-80225a2b7171, Duration=783ms)
[2023-07-21T19:38:10.608Z] Executing 'DurableVideoProcessorOrchestrator' (Reason='(null)', Id=effd49ae-1aa9-40cb-9b71-73cdf70f67a9)
[2023-07-21T19:38:10.687Z] Orchestrator starts the transcode and goes to sleep...
[2023-07-21T19:38:10.730Z] Executed 'DurableVideoProcessorOrchestrator' (Succeeded, Id=effd49ae-1aa9-40cb-9b71-73cdf70f67a9, Duration=127ms)
[2023-07-21T19:38:11.057Z] Executing 'Transcode' (Reason='(null)', Id=282af283-d69f-4fc4-a63c-b0ca86ae839e)
[2023-07-21T19:38:11.112Z] Transcoding http://this-is-a-test.com.
[2023-07-21T19:38:16.117Z] Executed 'Transcode' (Succeeded, Id=282af283-d69f-4fc4-a63c-b0ca86ae839e, Duration=5110ms)
[2023-07-21T19:38:16.413Z] Executing 'DurableVideoProcessorOrchestrator' (Reason='(null)', Id=f2af3e03-3f46-41e7-afe3-3ffa81a630aa)
[2023-07-21T19:38:16.429Z] Orchestrator starts the thumbnailing and goes to sleep...
[2023-07-21T19:38:16.435Z] Executed 'DurableVideoProcessorOrchestrator' (Succeeded, Id=f2af3e03-3f46-41e7-afe3-3ffa81a630aa, Duration=21ms)
[2023-07-21T19:38:16.611Z] Executing 'Thumbnail' (Reason='(null)', Id=673ae0ce-97cb-42fe-baed-1a65bba26644)
[2023-07-21T19:38:16.631Z] Thumbnailing this-is-a-test-transcoded.mp4.
[2023-07-21T19:38:21.686Z] Executed 'Thumbnail' (Succeeded, Id=673ae0ce-97cb-42fe-baed-1a65bba26644, Duration=5075ms)
[2023-07-21T19:38:21.987Z] Executing 'DurableVideoProcessorOrchestrator' (Reason='(null)', Id=b91c1c8c-2161-447c-a445-2bb1fd026612)
[2023-07-21T19:38:21.993Z] Orchestrator starts the intro preluding and goes to sleep...
[2023-07-21T19:38:21.995Z] Executed 'DurableVideoProcessorOrchestrator' (Succeeded, Id=b91c1c8c-2161-447c-a445-2bb1fd026612, Duration=9ms)
[2023-07-21T19:38:22.162Z] Executing 'PrependIntro' (Reason='(null)', Id=3deccb47-e058-4b33-899d-14b1a6337eb3)
[2023-07-21T19:38:22.236Z] Prepending intro into  to this-is-a-test-transcoded.mp4.
[2023-07-21T19:38:27.271Z] Executed 'PrependIntro' (Succeeded, Id=3deccb47-e058-4b33-899d-14b1a6337eb3, Duration=5110ms)
[2023-07-21T19:38:27.555Z] Executing 'DurableVideoProcessorOrchestrator' (Reason='(null)', Id=eef8a294-9a59-40c0-b903-1b7b13ec003a)
[2023-07-21T19:38:27.575Z] Executed 'DurableVideoProcessorOrchestrator' (Succeeded, Id=eef8a294-9a59-40c0-b903-1b7b13ec003a, Duration=20ms)

```
<br>

We can see that we could you could easily develop and run a simple long running(in short) workflow using a durable function implementation.

Please try out different complex patterns in durable function to make use of the full power.

***Happy coding....***