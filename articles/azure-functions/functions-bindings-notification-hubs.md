<properties
	pageTitle="Azure Functions Notification Hub binding | Microsoft Azure"
	description="Understand how to use Azure Notification Hub binding in Azure Functions."
	services="functions"
	documentationCenter="na"
	authors="wesmc7777"
	manager="erikre"
	editor=""
	tags=""
	keywords="azure functions, functions, event processing, dynamic compute, serverless architecture"/>

<tags
	ms.service="functions"
	ms.devlang="multiple"
	ms.topic="reference"
	ms.tgt_pltfrm="multiple"
	ms.workload="na"
	ms.date="10/18/2016"
	ms.author="wesmc"/>

# Azure Functions Notification Hub output binding

[AZURE.INCLUDE [functions-selector-bindings](../../includes/functions-selector-bindings.md)]

This article explains how to configure and code Azure Notification Hub bindings in Azure Functions. 

[AZURE.INCLUDE [intro](../../includes/functions-bindings-intro.md)] 

Your functions can send push notifications using a configured Azure Notification Hub with a very few lines of code. However, the Azure Notification Hub must be configured for the Platform Notifications Services (PNS) you want to use. For more information on configuring an Azure Notification Hub and developing a client applications that register to receive notifications, see [Getting started with Notification Hubs](../notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md) and click your target client platform at the top.

The notifications you send can be native notifications or template notifications. Native notifications target a specific notification platform as configured in the `platform` property of the output binding. A template notification can be used to target multiple platforms.   

## function.json for Azure Notification Hub output binding

The function.json file provides the following properties:

- `name` : Variable name used in function code for the notification hub message.
- `type` : must be set to *"notificationHub"*.
- `tagExpression` : Tag expressions allow you to specify that notifications be delivered to a set of devices who have registered to receive notifications that match the tag expression.  For more information, see [Routing and tag expressions](../notification-hubs/notification-hubs-tags-segment-push-message.md).
- `hubName` : Name of the notification hub resource in the Azure portal.
- `connection` : This connection string must be an **Application Setting** connection string set to the *DefaultFullSharedAccessSignature* value for your notification hub.
- `direction` : must be set to *"out"*. 
- `platform` : The platform property indicates the notification platform your notification will target. Must be one of the following values:
	- `template` : This is the default if the platform property is omitted from the output binding. Template notifications can be used to target any platform configured on the Azure Notification Hub. For more information on using templates in general to send cross platform notifications with an Azure Notification Hub, see [Templates](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md).
	- `apns` : Apple Push Notification Service. For more information on configuring the notification hub for APNS and receiving the notification in a client app, see [Sending push notifications to iOS with Azure Notification Hubs](../notification-hubs/notification-hubs-ios-apple-push-notification-apns-get-started.md) 
	- `adm` : [Amazon Device Messaging](https://developer.amazon.com/device-messaging). For more information on configuring the notification hub for ADM and receiving the notification in a Kindle app, see [Getting Started with Notification Hubs for Kindle apps](../notification-hubs/notification-hubs-kindle-amazon-adm-push-notification.md) 
	- `gcm` : [Google Cloud Messaging](https://developers.google.com/cloud-messaging/). Firebase Cloud Messaging, which is the new version of GCM, is also supported. For more information on configuring the notification hub for GCM/FCM and receiving the notification in an Android client app, see [Sending push notifications to Android with Azure Notification Hubs](../notification-hubs/notification-hubs-android-push-notification-google-fcm-get-started.md)
	- `wns` : [Windows Push Notification Services](https://msdn.microsoft.com/en-us/windows/uwp/controls-and-patterns/tiles-and-notifications-windows-push-notification-services--wns--overview) targeting Windows platforms. Windows Phone 8.1 and later is also supported by WNS. For more information on configuring the notification hub for WNS and receiving the notification in an Universal Windows Platform (UWP) app, see [Getting started with Notification Hubs for Windows Universal Platform Apps](../notification-hubs/notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md)
	- `mpns` : [Microsoft Push Notification Service](https://msdn.microsoft.com/en-us/library/windows/apps/ff402558.aspx). This platform supports Windows Phone 8 and earlier Windows Phone platforms. For more information on configuring the notification hub for MPNS and receiving the notification in an Windows Phone app, see [Sending push notifications with Azure Notification Hubs on Windows Phone](../notification-hubs/notification-hubs-windows-mobile-push-notifications-mpns.md)
 
Example function.json:

	{
	  "bindings": [
	    {
	      "name": "notification",
	      "type": "notificationHub",
	      "tagExpression": "",
	      "hubName": "my-notification-hub",
	      "connection": "MyHubConnectionString",
		  "platform": "gcm",
	      "direction": "out"
	    }
	  ],
	  "disabled": false
	}

## Azure Notification Hub connection string setup

To use a Notification hub output binding, you must configure the connection string for the hub. You can do this on the *Integrate* tab by selecting your notification hub or creating a new one. 

You can also manually add a connection string for an existing hub by adding a connection string for the *DefaultFullSharedAccessSignature* to your notification hub. This connection string provides your function access permission to send notification messages. The *DefaultFullSharedAccessSignature* connection string value can be accessed from the **keys** button in the main blade of your notification hub resource in the Azure portal. To manually add a connection string for your hub, use the following steps: 

1. On the **Function app** blade of the Azure portal, click **Function App Settings > Go to App Service settings**.

2. In the **Settings** blade, click **Application Settings**.

3. Scroll down to the **Connection strings** section, and add an named entry for *DefaultFullSharedAccessSignature* value for you notification hub. Change the type to **Custom**.
4. Reference your connection string name in the output bindings. Similar to **MyHubConnectionString** used in the example above.

## Azure Notification Hub code example for a Node.js timer trigger 

This example sends a notification for a [template registration](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md) that contains `location` and `message`.

	module.exports = function (context, myTimer) {
	    var timeStamp = new Date().toISOString();
	   
	    if(myTimer.isPastDue)
	    {
	        context.log('Node.js is running late!');
	    }
	    context.log('Node.js timer trigger function ran!', timeStamp);  
	    context.bindings.notification = {
	        location: "Redmond",
	        message: "Hello from Node!"
	    };
	    context.done();
	};

## Azure Notification Hub code example for a F# timer trigger

This example sends a notification for a [template registration](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md) that contains `location` and `message`.

	let Run(myTimer: TimerInfo, notification: byref<IDictionary<string, string>>) =
	    notification = dict [("location", "Redmond"); ("message", "Hello from F#!")]

## Azure Notification Hub code example for a C# queue trigger

#### APNS native notification example

This example shows how to use types defined in the [Microsoft Azure Notification Hubs Library](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/) to send a native APNS notification. 

	#r "Microsoft.Azure.NotificationHubs"
	#r "Newtonsoft.Json"
	
	using System;
	using Microsoft.Azure.NotificationHubs;
	using Newtonsoft.Json;
	
	public static async Task Run(string myQueueItem, IAsyncCollector<Notification> notification, TraceWriter log)
	{
	    log.Info($"C# Queue trigger function processed: {myQueueItem}");
	
		// In this example the queue item is a new user to be processed in the form of a JSON string with 
		// a "name" value.
		//
		// The JSON format for a native APNS notification is ...
		// { "aps": { "alert": "notification message" }}  

	    log.Info($"Sending APNS notification of a new user");	
	    dynamic user = JsonConvert.DeserializeObject(myQueueItem);	
	    string apnsNotificationPayload = "{\"aps\": {\"alert\": \"A new user wants to be added (" + 
											user.name + ")\" }}";
	    log.Info($"{apnsNotificationPayload}");
	    await notification.AddAsync(new AppleNotification(apnsNotificationPayload));	    
	}

#### GCM native notification example

This example shows how to use types defined in the [Microsoft Azure Notification Hubs Library](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/) to send a native GCM notification. 

	#r "Microsoft.Azure.NotificationHubs"
	#r "Newtonsoft.Json"
	
	using System;
	using Microsoft.Azure.NotificationHubs;
	using Newtonsoft.Json;
	
	public static async Task Run(string myQueueItem, IAsyncCollector<Notification> notification, TraceWriter log)
	{
	    log.Info($"C# Queue trigger function processed: {myQueueItem}");
	
		// In this example the queue item is a new user to be processed in the form of a JSON string with 
		// a "name" value.
		//
		// The JSON format for a native GCM notification is ...
		// { "data": { "message": "notification message" }}  

	    log.Info($"Sending GCM notification of a new user");	
	    dynamic user = JsonConvert.DeserializeObject(myQueueItem);	
	    string gcmNotificationPayload = "{\"data\": {\"message\": \"A new user wants to be added (" + 
											user.name + ")\" }}";
	    log.Info($"{gcmNotificationPayload}");
	    await notification.AddAsync(new GcmNotification(gcmNotificationPayload));	    
	}

#### WNS native notification example

This example shows how to use types defined in the [Microsoft Azure Notification Hubs Library](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/) to send a native WNS toast notification. 

	#r "Microsoft.Azure.NotificationHubs"
	#r "Newtonsoft.Json"
	
	using System;
	using Microsoft.Azure.NotificationHubs;
	using Newtonsoft.Json;
	
	public static async Task Run(string myQueueItem, IAsyncCollector<Notification> notification, TraceWriter log)
	{
	    log.Info($"C# Queue trigger function processed: {myQueueItem}");
	
		// In this example the queue item is a new user to be processed in the form of a JSON string with 
		// a "name" value.
		//
		// The XML format for a native WNS toast notification is ...
		// <?xml version="1.0" encoding="utf-8"?>
		// <toast>
		// 	 <visual>
		//     <binding template="ToastText01">
		//       <text id="1">notification message</text>
		//     </binding>
		//   </visual>
		// </toast>

	    log.Info($"Sending WNS toast notification of a new user");	
	    dynamic user = JsonConvert.DeserializeObject(myQueueItem);	
	    string wnsNotificationPayload = "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
										"<toast><visual><binding template=\"ToastText01\">" +
											"<text id=\"1\">" + 
												"A new user wants to be added (" + user.name + ")" + 
											"</text>" +
										"</binding></visual></toast>";

	    log.Info($"{wnsNotificationPayload}");
	    await notification.AddAsync(new WindowsNotification(wnsNotificationPayload));	    
	}


#### Template example using an out parameter 

This example sends a notification for a [template registration](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md) that contains a `message` place holder in the template.

	using System;
	using System.Threading.Tasks;
	using System.Collections.Generic;
	 
	public static void Run(string myQueueItem,  out IDictionary<string, string> notification, TraceWriter log)
	{
	    log.Info($"C# Queue trigger function processed: {myQueueItem}");
        notification = GetTemplateProperties(myQueueItem);
	}
	 
	private static IDictionary<string, string> GetTemplateProperties(string message)
	{
	    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
	    templateProperties["message"] = message;
	    return templateProperties;
	}

#### Asynchronous template example

If you are using asynchronous code, you won't be able use out parameters. In this case use `IAsyncCollector` to return your template notification. The following code is an asynchronous example of the code above. 

	using System;
	using System.Threading.Tasks;
	using System.Collections.Generic;
	
	public static async Task Run(string myQueueItem, IAsyncCollector<IDictionary<string,string>> notification, TraceWriter log)
	{
	    log.Info($"C# Queue trigger function processed: {myQueueItem}");
	
	    log.Info($"Sending Template Notification to Notification Hub");
	    await notification.AddAsync(GetTemplateProperties(myQueueItem));    
	}
	
	private static IDictionary<string, string> GetTemplateProperties(string message)
	{
	    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
	    templateProperties["user"] = "A new user wants to be added : " + message;
	    return templateProperties;
	}

#### Template example using JSON 

This example sends a notification for a [template registration](../notification-hubs/notification-hubs-templates-cross-platform-push-messages.md) that contains a `message` place holder in the template using a valid JSON string.

	using System;
	 
	public static void Run(string myQueueItem,  out string notification, TraceWriter log)
	{
		log.Info($"C# Queue trigger function processed: {myQueueItem}");
		notification = "{\"message\":\"Hello from C#. Processed a queue item!\"}";
	}


#### Template example using Notification Hubs library types

This example shows how to use types defined in the [Microsoft Azure Notification Hubs Library](https://www.nuget.org/packages/Microsoft.Azure.NotificationHubs/). 


	#r "Microsoft.Azure.NotificationHubs"

	using System;
	using System.Threading.Tasks;
	using Microsoft.Azure.NotificationHubs;
	 
	public static void Run(string myQueueItem,  out Notification notification, TraceWriter log)
	{
	   log.Info($"C# Queue trigger function processed: {myQueueItem}");
	   notification = GetTemplateNotification(myQueueItem);
	}

	private static TemplateNotification GetTemplateNotification(string message)
	{
	    Dictionary<string, string> templateProperties = new Dictionary<string, string>();
	    templateProperties["message"] = message;
	    return new TemplateNotification(templateProperties);
	}

## Next steps

[AZURE.INCLUDE [next steps](../../includes/functions-bindings-next-steps.md)]
