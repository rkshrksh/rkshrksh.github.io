+++
title = "End-to-End Tracking in Azure Functions"
date = "2025-07-19"
description = "Learn how to implement end-to-end request tracking using TrackingID in Azure Functions and Service Bus with structured logging and scoped diagnostics."
tags = [
    "azure-functions",
    "service-bus",
    "tracking",
    "logging",
    "distributed-systems"
]
+++

# End-to-End Tracking in Azure Functions and Service Bus with `TrackingID`

In today's cloud-native applications, services are increasingly distributed—events trigger processes, APIs invoke background jobs, and messages flow asynchronously via queues or topics.

While this decoupling improves scalability, it makes **observability** more challenging. When something goes wrong, how do you trace the full journey of a single user's request across dozens of services?

The answer: **End-to-End Tracking** using a correlation ID or `TrackingID`. In this blog, you’ll learn how to implement request tracking in **Azure Functions** that communicate through **Azure Service Bus** using the modern SDK.

## Architecture at a Glance

1. **Producer Function**: Receives an HTTP request, generates or extracts a `TrackingID`, sends a message to Service Bus with this ID as a custom property.
2. **Consumer Function**: Listens to the Service Bus queue or topic, reads the `TrackingID`, and uses `BeginScope` for structured logging. It can optionally pass the ID further as needed.

## 1. Producer Function – Sending a Message with `TrackingID`

```csharp
using System.IO;
using System.Text;
using System.Threading.Tasks;
using Azure.Messaging.ServiceBus;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using System;

public static class ProducerFunction
{
    [FunctionName("ProducerFunction")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
        [ServiceBus("myqueue", Connection = "ServiceBusConnection")] IAsyncCollector<ServiceBusMessage> outputMessages,
        ILogger log)
    {
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

        // Extract or generate a TrackingID
        string trackingId = req.Headers.ContainsKey("TrackingID")
            ? req.Headers["TrackingID"].ToString()
            : Guid.NewGuid().ToString();

        // Create and send the Service Bus message with custom property
        var message = new ServiceBusMessage(Encoding.UTF8.GetBytes(requestBody));
        message.ApplicationProperties["TrackingID"] = trackingId;

        log.LogInformation("Sending message with TrackingID: {TrackingID}", trackingId);

        await outputMessages.AddAsync(message);

        return new OkObjectResult(new
        {
            Status = "Message sent",
            TrackingID = trackingId
        });
    }
}
```

**Highlights:**
- Uses the `Azure.Messaging.ServiceBus` SDK.
- Attaches `TrackingID` as a custom application property.
- Logs the `TrackingID` for traceability.

## 2. Consumer Function – Receiving and Logging with `TrackingID`

This consumer reads from the queue/topic, extracts the `TrackingID`, and uses `BeginScope` to enrich logs across the entire scope.

```csharp
using System;
using System.Collections.Generic;
using System.Text;
using System.Threading.Tasks;
using Azure.Messaging.ServiceBus;
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;

public static class ConsumerFunction
{
    [FunctionName("ConsumerFunction")]
    public static async Task Run(
        [ServiceBusTrigger("myqueue", Connection = "ServiceBusConnection")] ServiceBusReceivedMessage message,
        ILogger log)
    {
        // Extract TrackingID from the message properties
        string trackingId = message.ApplicationProperties.ContainsKey("TrackingID")
            ? message.ApplicationProperties["TrackingID"].ToString()
            : "N/A";

        // Begin a logging scope that includes the TrackingID
        using (log.BeginScope(new Dictionary<string, object> { ["TrackingID"] = trackingId }))
        {
            log.LogInformation("Received message with TrackingID: {TrackingID}", trackingId);

            // Simulate message processing
            string body = Encoding.UTF8.GetString(message.Body);
            log.LogInformation("Processing message body: {Body}", body);

            // Pass trackingId downstream if needed
            // For example, add it to outgoing HTTP headers or service calls
        }

        await Task.CompletedTask;
    }
}
```

**Scope Logging Bonus:**  
By using `log.BeginScope`, every log inside that scope gets auto-tagged with `TrackingID`. This is powerful with log aggregators like Application Insights, Elastic Stack, or Datadog.

## Propagating `TrackingID` to Downstream Services

Once you've got the `TrackingID`, you can:
- Send it as a header in HTTP requests:
  ```
  httpClient.DefaultRequestHeaders.Add("TrackingID", trackingId);
  ```
- Use it in telemetry or performance metrics
- Store it in databases or audit trails

Keeping track of the ID across services provides **unified visibility**, even on large-scale systems.

## Testing It Locally

1. **Run the Azure Functions** project with both functions.
2. Send a request using `curl` or Postman to the producer:
   ```
   curl -X POST http://localhost:7071/api/ProducerFunction -d '{"message":"hello"}' -H "Content-Type: application/json"
   ```
3. Watch the logs in both producer and consumer. You should see consistent `TrackingID` values.

## Conclusion

With just a few lines of code, you can significantly improve observability in your distributed Azure solution:
- Use `TrackingID` as a unique correlation ID
- Forward it through Service Bus with `ApplicationProperties`
- Use `BeginScope` to enrich logs at every stage

This lightweight pattern is easy to add, doesn’t impact performance, and unlocks powerful traceability across your event-driven architecture.

## What’s Next?

- Integrate Application Insights for centralized tracking
- Extend this pattern to handle retries and dead-letter messages
- Use Middleware or DelegatingHandlers to forward `TrackingID` in outgoing HTTP requests automatically

Happy tracing! 
