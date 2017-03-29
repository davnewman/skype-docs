# Trusted Application API - SDK

This article will act as an introduction to the Trusted Application API SDK that can be used to communicate with **Skype for Business - Platform Service** (called platform service from here on)

The quick start samples can be used to learn more about the code beyond that is mentioned here.

## Installation

The Trusted Application API SDK (called sdk from here on) is available as a prerelease in nuget.org. It can be installed in your project with the following command.

```
Install-Package Microsoft.SkypeforBusiness.TrustedApplicationAPI.SDK -Pre
```

Note that the solution must target **.NET Framework 4.6.2 or higher** for the SDK to work.

## Application Registration

Any application that communicates with Platform Service must be registered in [Registration Portal](https://aka.ms/skypeappregistration). You must note the AAD Client ID and AAD Client Secret as received in the portal.

## First Steps

To use the SDK, you require to first create a `ClientPlatformSettings` Object.

```
var platformSettings = new ClientPlatformSettings(AAD_ClientSecret, Guid.Parse(AAD_ClientId));
```

The AAD client id and secret are those values received from the registration portal. Now create the Client Platform object.

```
var platform = new ClientPlatform(platformSettings);
```

In order to use platform service, you need to create a event channel that implements the `IEventChannel` interface. For the next part, `EventChannel` is a class that implements that interface. We create an instance for the event channel.

```
var eventChannel = new EventChannel();
```

We need to create an application endpoint for the use with platform service. First, we create the endpoint settings object

```
var endpointSettings = new ApplicationEndpointSettings("sip:example@contoso.com");
```

The sip uri is the uri that is associated with the application endpoint. Then, we create the endpoint

```
var applicationEndpoint = new ApplicationEndpoint(platform, endpointSettings, eventChannel);
```

We now initialize the application and the endpoint to start communication with platform service.

```
await applicationEndpoint.InitializeAsync().ConfigureAwait(false);
await applicationEndpoint.InitializeApplicationAsync().ConfigureAwait(false);
```

The application and the endpoint are now initialized and could be used for any communication that is required.

## Example

### Create AdHocMeeting

To create an adhoc meeting, first create a AdhocMeetingCreationInput object and then call create adhoc meeting on endpoint.

```
var input = new AdhocMeetingCreationInput("subject");
var adhocMeeting = await applicationEndpoint.Application.CreateAdhocMeetingAsync(input).ConfigureAwait(false);
```


