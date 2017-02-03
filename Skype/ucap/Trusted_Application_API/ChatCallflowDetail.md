# Customer chat care call flow
 
## Call flow 

To start chat, the anonymous chat client pings the Service Application (SA), which further communicates with the Trusted Application API as follow: 

1. The Service Application (SA) discovers the location of the Trusted Application API (API)

    **a. Discovery Request**
    ```
    GET https://noammeetings.resources.lync.com/platformservice/discover
    ```
    **b. Discovery Response** -  Link to Trusted Applications API
    ```
    200 OK,"ms:rtc:saas:applications":{"href":"https://ring2noammeetings.resources.lync.com/platformService/v1/applications"}
    ```

2. The Service Application gets the capabilities
       
   **a. GET request with Trusted Applications API link** -
   We send GET request with Trusted Applications API link  received from the previous discovery Request. This GET request must be authenticated with a **valid OAuth token.**
    
    ```
    // A request without a valid OAuth token will return '401 Unauthorized response'.
    
    GET https://ring2noammeetings.resources.lync.com/platformService/v1/applications

    Response - 401 Unauthorized response
    ```
     
    **b. Capabilities GET Request with a valid OAuth token.**
    ```
    GET https://ring2noammeetings.resources.lync.com/platformService/v1/applications
    Authorization: Bearer XXXX
    ```
   **c. Capabilities Response** - Anonymous application tokens capability (ms:rtc:saas:anonApplicationTokens resource).

    ```
    200 OK,ms:rtc:saas:anonApplicationTokens": {
    "href": "/platformservice/v1/applications/1627259584/anonApplicationTokens?endpointId=sip:helpdesk@contoso.com" }
    ```
3. POST on anonApplicationTokens resource to get the token and a **[ms:rtc:saas:discover](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_discover.html)** link that can be shared with the anonymous chat client.

    **a. POST request on anonApplicationTokens resource**    
    ```
    Post /platformservice/v1/applications/1627259584/anonApplicationTokens?endpointId=sip:helpdesk@contos.com
    "applicationSessionId": "kkkk",   "allowedOrigins": "https://contoso.com;https://litware.com"}
    ```
    **b. Response**
    ```
    200 OK with token and discovery link
    "token": "
    psat=eyJ0eX....", "expiryTime": "2016-06-27T01:42:13.094Z",
    "ms:rtc:saas:discover": 
    {"href": "https://noammeetings.resources.lync.com/platformService/discover?anonymousContext=psat%253deyJ0eX..."}
    ```

    **c. Send ms:rtc:saas:discover url and token to the anonymous chat client**

    ![UCAP initiated callflow](images/UCAP Anon P2P_1.jpg)

4. The anonymous chat client uses the **[ms:rtc:saas:discover](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_discover.html)** link to get the anonApplications resource, which can be used to establish a ucwa session with the Trusted Application API
   
   **a. GET request on ms:rtc:saas:discover url to find "anonApplications" token**
   ```
   Get /platformservice/discover?anonymousContext=psat%253deyJ0e...
   Origin : http://contoso.com
   ```
   **b. Response - anonApplications resource**
   ```
   200 OK,anonApplications": {"href": "/ucwa/psanon/v1/applications"}
   ```
5. The chat client establishes a ucwa session with the API and uses the token from step 3, to authenticate itself.
    
    **a. Create Ucwa application at _anonApplications_ link with POST request**
    ```
    Post /ucwa/psanon/v1/applications
    "anonymousDisplayName": "Janardhan"}

    Response - 201 Created
    ```
6. The chat client sends a messaging invitation to the helpdesk uri: sip:helpdesk@contoso.com
    
    **a. POST request for chat support to helpdesk's sip uri. Custom content can be also be sent here.**
    ```
    POST /ucwa/psanon/v1/applications/101485875823/communication/messagingInvitations
    {"operationId":"ucwap2pimop","subject":"$(TrackableSubject)","to":"sip:helpdesk@contoso.com","customContent":"CID:2"}
    
    Response - 201 Created
    ```
7. The Trusted Application API sends a callback to the Service Application on receiving this messaging invitation, along with the two links: **[ms:rtc:saas:startAdhocMeeting](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_startAdhocMeeting.html)** and **[ms:rtc:saas:acceptAndBridge](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_acceptAndBridge.html)**.

    **a. Callback to Service Application** - It indicates that a **[ms:rtc:saas:messagingInvitation](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_messagingInvitation.html)** was started. It has the options to start an on-demand meeting and a way to bridge the messaging invitation with the on-demand meeting.
    This also includes the custom content sent from the chat client.
    ```
    POST https://litware.com/callback
    "rel": "ms:rtc:saas:messagingInvitation"  "type": "started"     
    "ms:rtc:saas:startAdhocMeeting": {"href": "/platformservice/tgt-c39794177c465e6c922bd0737e01fd03/v1/applications/1627259584
    /communication/onlineMeetingInvitations/startAdhocMeeting?endpointId=sip:helpdesk@contoso.com"}
    "ms:rtc:saas:acceptAndBridge": {"href": "/platformservice/tgt-c39794177c465e6c922bd0737e01fd03/v1/applications/1627259584
    /communication/messagingInvitations/b46270e0-fc92-43c5-b89c-4d6d0f2a16af/acceptAndBridge?endpointId=sip:helpdesk@contoso.com"} 
    "customContent": "href": "data:foo/foo,ThisIsARandomCustomContent"                              
    ```
   ![UCAP initiated callflow](images/UCAP Anon P2P_2.jpg)

8. The Service Application(SA) sees that it has received a support request and spins up an on-demand meeting, using the link in the resource **[ms:rtc:saas:startAdhocMeeting](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_startAdhocMeeting.html)**. SA receives a callback event with conversation state as conferenced, along with an **[ms:rtc:saas:addParticipant](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_addParticipant.html)** link. This link is used to add participants to the meeting.
    
    **a. Create and join the on-demand (adhoc) meeting**
    ```
    Post /platformservice/tgt-c39794177c465e6c922bd0737e01fd03/v1/applications/1627259584
    /communication/onlineMeetingInvitations/startAdhocMeeting?endpointId=sip:helpdesk@contoso.com
    "subject": "helpcustomermeeting", "callbackUrl": "https://litware.com/callback", "operationId": "abc"
    
    Response - 201 Created
    ```
9. SA then uses the **[ms:rtc:saas:acceptAndBridge](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_acceptAndBridge.html)** capability to bridge the incoming messaging invitation into the on demand meeting that it just created.
10. SA receives a callback : **[ms:rtc:saas:conversationBridge](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_conversationBridge.html)** added event, meaning a new conversation bridge has been created. Calling a GET on this resource shows you another resource, which can be used to get or add bridge participants :**[ms:rtc:saas:bridgedParticipants](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_bridgedParticipants.html)**.
11. SA POSTs on the **[ms:rtc:saas:bridgedParticipants](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_bridgedParticipants.html)** link to add the SFB online users who are handling the chat support requests to the conversation bridge. This request includes a messageFilterState parameter set to disabled. If enabled, it means that the messages sent by the participant are not sent to the chat client, and vice versa. This request also has displayName and participantUri parameters to set the display name and indicate the sip uri of the SFB online user to add.
 
12. The SA does a POST on the **[ms:rtc:saas:sendMessage](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_sendMessage.html)** token link with a welcome text and gets callbacks informing that the message has been successfully sent.
13. The chat client receives the message in its event channel
14. SA then proceeds to add one or more participants to the meeting, using POST on the link of the resource : **[ms:rtc:saas:addParticipant](http://trustedappapi.azurewebsites.net/Resources/ms_rtc_saas_addParticipant.html)**, got in step 8. These participants are the customer service representatives that will answer the chat support request, and are SFB online users.

   ![UCAP initiated callflow](images/UCAP Anon P2P_3.jpg)

15. The SA gets a callback when the SFB Online user joined the IM meeting.
16. If the SFB Online user is sending an "Is Typing" notification, it will be conveyed to the chat client on its event channel
17. SFB online user sends a message, it will be conveyed to the chat client on its event channel
18. The chat client sends an IsTyping notification, it will be conveyed to all the SFB online users joined to the conference.
19. The chat client sends a message, it will be conveyed to all the SFB online users joined to the conference.
20. The chat client now closes the conversation
21. API sends a callback to the SA, indicating the conversation has been deleted.
22. The SA then sends a request to terminate the conference conversation with the Trusted Application API
   ![UCAP initiated callflow](images/UCAP Anon P2P_4.jpg)
 
The way the API informs the SA of any event is via the callback url. In order for the SA to keep context when starting a new invitation or message, an operation id can be passed in. This id will be sent back in the callback event.
 
The general principles, capabilities, API style, API concepts are modeled to closely follow the already released UCWA API. Please see this link for detailed info - [UCWA API reference](https://msdn.microsoft.com/en-us/skype/ucwa/ucwa2_0apireference)
 
Being familiar with the UCWA concepts greatly simplifies understanding the Trusted Application APIs
 
 
The detailed call flow is shown in the diagram below. Each of the capability related APIs, related callbacks are modeled on the UCWA capabilities and callback structure. Please refer to the [UCWA API reference](https://msdn.microsoft.com/en-us/skype/ucwa/ucwa2_0apireference) to get detailed description
about these  calls.

## Next step
The callflow describes the customer care chat scenario from the perspective of a service application. To see how client side logic is implemented, get the [Web SDK chat sample](https://github.com/OfficeDev/skype-docs/tree/master/Skype/WebSDK/samples/Chat) or review the following topic.
- [Implementing a Chat Client with the Skype Web SDK](ImplementingChatClientWithSkypeWebSDK.md)

![Trusted Application API initiated callflow](images/CallFlow1image001.png) 