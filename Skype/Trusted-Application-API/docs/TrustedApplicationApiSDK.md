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

To use the SDK, you require to first create a `ClientPlatformSettings` Object which holds the settings and an implementation of `IPlatformServiceLogger` class which provides logging. 

```
var platformSettings = new ClientPlatformSettings(AAD_ClientSecret, Guid.Parse(AAD_ClientId));

// This is an implementation of the IPlatformServiceLogger interface
var logger = new PlatformServiceLogger();
```

The AAD client id and secret are those values received from the registration portal. Now create the Client Platform object.

```
var platform = new ClientPlatform(platformSettings, logger);
```

In order to use platform service, you need to create a event channel that implements the `IEventChannel` interface. For the next part, `EventChannel` is a class that implements that interface. We create an instance for the event channel.

```
IEventChannel eventChannel = new EventChannel();
```

We need to create an application endpoint for the use with platform service. First, we create the endpoint settings object

```
var endpointSettings = new ApplicationEndpointSettings(new SipUri("sip:example@contoso.com"));
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

## Check if a feature is available or not

All `IPlatformResource` objects will have a *Supports* method which can be used to check if the application has links available for the operation.

For example, to check if the application can start an `AudioVideo` call, developer can do this

```
var communication = applicationEndpoint.Application.Communication;

if (!communication.Supports(CommunicationCapability.StartAudioVideo))
{
    return;
}
```

If the app ends up calling the API, we throw `CapabilityNotAvailableException`.

## Examples

### Create AdHocMeeting

To create an adhoc meeting, first create a AdhocMeetingCreationInput object and then call create adhoc meeting on endpoint.

```
var input = new AdhocMeetingCreationInput("subject");
var adhocMeeting = await applicationEndpoint.Application.CreateAdhocMeetingAsync(input).ConfigureAwait(false);
```

### Send a P2P Message

To send a P2P message to `sip:target@contoso.com`, we first create an invitation and then wait for its completion

```
var invitation = await applicationEndpoint.Application.Communication.StartMessagingAsync(
                            subject: "Subject",
                            to: new SipUri("sip:target@contoso.com"),
                            callbackUrl: callbackUri.ToString(),
                            loggingContext: loggingContext).ConfigureAwait(false);


// Wait for user to accept the invitation
var conversation = await invitation.WaitForInviteCompleteAsync().ConfigureAwait(false);
```

To send a message and receive messages, we can do the following
```
// Send a message
await conversation.MessagingCall.SendMessagingAsync("Example message").ConfigureAwait(false);

// Receive a message
conversation.MessagingCall.IncomingMessageReceived += (sender, args) => {
    var msg = Encoding.UTF8.GetString(incomingMessageEventArgs.PlainMessage.Message);
    var receivedFrom = args.FromParticipantName;
    
    // Do something with the message.
}
```

### Listen for incoming calls

You can listen to new incoming calls by registering a callback on the endpoint.

```
applicationEndpoint.HandleIncomingAudioVideoCall += On_AudioVideoCall_Received;

async void On_AudioVideoCall_Received(object sender, IncomingInviteEventArgs<IAudioVideoInvitation> e) 
{
    await e.NewInvite.AcceptAsync().ConfigureAwait(false);
    var conversation = e.NewInvite.RelatedConversation;

    // Do something with the conversation
}
```

## Exceptions

- A `PlatformServiceClientInvalidOperationException` is thrown when an invalid operation within platform service is executed.

- A `CapabilityNotAvailableException` is thrown when a capability is used while unavailable.
