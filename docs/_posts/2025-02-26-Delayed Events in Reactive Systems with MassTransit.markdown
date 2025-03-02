---
layout: post
title:  "Delayed Events in Reactive Systems with MassTransit"
date:   2025-02-26 15:25:00 +0530
categories: jekyll update
published: true
tags: [Events]
description: "A developer-friendly breakdown of delayed jobs and its use cases in event driven applications."
---
<div style="text-align: center;">
  <img src="\images\2025-02-26-delayed-jobs\DALL·E 2025-02-26 11.37.54 - A conceptual illustration representing delayed events in reactive systems using MassTransit. The image features a futuristic digital network with glow.webp" style="width: 500px; height: auto;">
</div>
<div style="text-align: center;">
Courtesy: DALL-E</div>



---

<br>
<br>


## **Introduction**

While exploring reactive architectures, I came across an insightful blog post by **Antônio Falcão Jr** on [Delayed Events in Reactive Systems](https://www.linkedin.com/pulse/delayed-events-reactive-systems-ant%C3%B4nio-falc%C3%A3o-jr-ibubf/?trackingId=%2BYAEk%2BwKTzuLc1LGQeqFWA%3D%3D). It resonated deeply with me as it highlights an elegant alternative to scheduled jobs and batch processing by leveraging **delayed events**.

This post explains how we can apply **Delayed Events** in a **MassTransit-based .NET application**, using an **Inventory Reservation System** as an example.

---

## **What Are Delayed Events?**

### **The Problem with Traditional Batch Processing**
In traditional systems, we use batch jobs or scheduled tasks to update state based on time-based conditions. For example, in an **e-commerce system**, an item reserved in a shopping cart should be released if the customer doesn’t proceed to checkout within 10 minutes. The traditional approach might:

- Use a scheduled job running every X minutes to check for expired reservations.
- Perform large database queries, affecting performance.
- Lead to **inconsistent states** if a batch fails midway.

### **Reactive Alternative: Event-Driven Scheduling**
With **Delayed Events**, each entity manages its own lifecycle:

1. **When a customer adds an item to the cart**, an `InventoryReserved` event is published.
2. A **delayed** `InventoryReleased` event is scheduled **10 minutes later**.
3. If checkout happens before expiration, we cancel the scheduled event.
4. If the customer doesn’t checkout, the **scheduled event automatically triggers** and releases the inventory.

This approach avoids **batch processing overhead** and ensures **scalability and resilience**.

---

## **Implementation Using MassTransit**

### **1️⃣ Define the Events**
We start by defining two events:

- `InventoryReserved` is triggered when an item is reserved.
- `InventoryReleased` is scheduled to trigger after 10 minutes.

```csharp
public record InventoryReserved(Guid CartId, DateTime ExpirationTime);

public record InventoryReleased(Guid CartId);
```

---

### **2️⃣ Publish the `InventoryReserved` Event**
When an item is added to the cart, we publish an event and schedule inventory release.

```csharp
public class CartService
{
    private readonly IPublishEndpoint _publishEndpoint;

    public CartService(IPublishEndpoint publishEndpoint)
    {
        _publishEndpoint = publishEndpoint;
    }

    public async Task AddToCartAsync(Guid cartId)
    {
        var expirationTime = DateTime.UtcNow.AddMinutes(10);
        
        await _publishEndpoint.Publish(new InventoryReserved(cartId, expirationTime));

        Console.WriteLine($"[Cart {cartId}] Inventory reserved. Will expire at {expirationTime}");
    }
}
```

---

### **3️⃣ Handle `InventoryReserved` and Schedule a Delayed Event**
MassTransit provides a **message scheduler** to delay event execution.

```csharp
public class InventoryReservedConsumer : IConsumer<InventoryReserved>
{
    private readonly IMessageScheduler _scheduler;

    public InventoryReservedConsumer(IMessageScheduler scheduler)
    {
        _scheduler = scheduler;
    }

    public async Task Consume(ConsumeContext<InventoryReserved> context)
    {
        var cartId = context.Message.CartId;
        var expirationTime = context.Message.ExpirationTime;

        await _scheduler.SchedulePublish(expirationTime, new InventoryReleased(cartId));

        Console.WriteLine($"[Cart {cartId}] Delayed InventoryReleased event scheduled at {expirationTime}");
    }
}
```

---

### **4️⃣ Process the `InventoryReleased` Event**
When the delayed event triggers, we release inventory.

```csharp
public class InventoryReleasedConsumer : IConsumer<InventoryReleased>
{
    public async Task Consume(ConsumeContext<InventoryReleased> context)
    {
        var cartId = context.Message.CartId;

        Console.WriteLine($"[Cart {cartId}] Inventory released due to timeout.");

        await ReleaseInventory(cartId);
    }

    private Task ReleaseInventory(Guid cartId)
    {
        Console.WriteLine($"[Inventory] Released stock for cart {cartId}");
        return Task.CompletedTask;
    }
}
```

---

### **5️⃣ Configure MassTransit with RabbitMQ**
We configure MassTransit to use **RabbitMQ and a Message Scheduler**.

```csharp
public static class MassTransitConfig
{
    public static void ConfigureServices(IServiceCollection services)
    {
        services.AddMassTransit(x =>
        {
            x.AddConsumer<InventoryReservedConsumer>();
            x.AddConsumer<InventoryReleasedConsumer>();

            x.UsingRabbitMq((context, cfg) =>
            {
                cfg.UseMessageScheduler(new Uri("queue:scheduler"));
                
                cfg.ReceiveEndpoint("inventory-reserved", e =>
                {
                    e.ConfigureConsumer<InventoryReservedConsumer>(context);
                });

                cfg.ReceiveEndpoint("inventory-released", e =>
                {
                    e.ConfigureConsumer<InventoryReleasedConsumer>(context);
                });
            });
        });
    }
}
```

---


*Edit on original post to include the `Inventory Released` consumption and cancellation:*


> Thanks for **David Nguyen** [@hpcsc](https://disqus.com/by/hpcsc/) for raising the comment.


### **5️⃣ Cancel the Scheduled Event on Checkout**

```csharp
public class CartService
{
    private readonly IPublishEndpoint _publishEndpoint;
    private readonly IMessageScheduler _scheduler;
    private readonly ICacheService _cacheService;

    public CartService(IPublishEndpoint publishEndpoint, IMessageScheduler scheduler, ICacheService cacheService)
    {
        _publishEndpoint = publishEndpoint;
        _scheduler = scheduler;
        _cacheService = cacheService;
    }

    public async Task CheckoutAsync(Guid cartId)
    {
        Console.WriteLine($"[Cart {cartId}] Checkout completed!");

        var tokenId = _cacheService.GetScheduledToken(cartId);

        if (tokenId != null)
        {
            await _scheduler.CancelScheduledPublish(tokenId.Value);
            Console.WriteLine($"[Cart {cartId}] Inventory release event canceled.");
        }

        await _publishEndpoint.Publish(new OrderConfirmed(cartId));
    }
}
```

---

## **Why This Approach?**
✔ **Avoids batch processing overhead**
✔ **No need for expensive database queries**
✔ **Each event is processed independently**
✔ **Highly scalable and resilient**
✔ **Easier debugging and monitoring**

---

## **Final Thoughts**
By using **Delayed Events in MassTransit**, we can design more **reactive, scalable, and fault-tolerant** systems without relying on traditional batch processing. Thanks to **Antônio Falcão Jr** for talking about this pattern—it’s a game-changer for **Event-Driven Architecture and Domain-Driven Design**.

Are you using **MassTransit** or **event scheduling** in your projects? Let me know your thoughts in the comments! 🚀

---

## **Further Reading**
- [Antônio Falcão Jr’s Original Blog Post](https://www.linkedin.com/pulse/delayed-events-reactive-systems-ant%C3%B4nio-falc%C3%A3o-jr-ibubf/?trackingId=%2BYAEk%2BwKTzuLc1LGQeqFWA%3D%3D)
- [MassTransit Documentation](https://masstransit.io)
- [RabbitMQ Delayed Messaging Plugin](https://www.rabbitmq.com/dlx.html)


Peace....🍀
