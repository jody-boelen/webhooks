# Webhook Producer
The webhooks module allows you to easily add webhooks to your Mendix application. It contains a reference REST service (excluded document) that has all the endpoints you need to enable webhooks in your app. Furthermore, it has actions that you can use to send the webhooks asynchronously using the task queue. See the how-to section for a more detailed explanation.

## Features and limitations

- Expose a REST service to manage webhooks and the webhook lifecycle.
- Send asynchronous callouts via the MX9 task queue.
- Consuming webhooks endpoints can be secured in several ways, which are all handled by the Webhooks module: Basic, ClientCredentials, CustomHeader.
- See the github repository for the openAPI definition of the webhooks API.

## Dependencies

- Mendix 9.6.6 or higher
- Community Commons Module
- Encryption Module
- Json-simple jar file

## Configuration

1. Grant the Administrator/Configurator the Administrator module role
2. Create a page that uses the SNIP_Webhook_Overview snippet from the USEME module
3. Make sure the encryption key constant is filled
4. Include the webhook PRS (PRS_Webhook_Example) or use this as a basis for your own implementation.
5. If you use PRS_Webhook_Example, give the API_User access to the APIUser module role.
6. Set the ChallengeHeaderName constant, this will be used to verify the webhook (see webhook lifecycle).
7. You are ready to use the module.

## How to use the module

After configuration, you can use the module to send notifications via a webhook to the consumers. As mentioned before, this all happens asynchronously via the Mx9 task queue. This section has some more information on how to use the module.

#### Available microflow actions
- SUB_WebhookBody_Create: Use this action to create a JSON structure for your webhook. This JSON structure will be sent in each callout. It only has the minimal information, but you are of course able to modify this if needed.
- SUB_Webhook_FilterList: Use this action to apply filtering on the callouts to be sent. An example might be that clients only want to receive their own notifications, not notifications of objects owned by other clients. This microflow should be used to filter such scenario's.
- SUB_Client_GetCurrentClient: Currently, each webhook is associated with User. If you have another Entity that represents your API clients, you can use this microflow to select that entity instead.
- SUB_Callout_Retry: This microflow will be triggered if a callout fails. By default, the callout will be retried once. If you want a more sophisticated retry mechanism, you can add that in this microflow. TIP: Mendix 9.10 and above support a retry mechanism out of the box, the suggestion would be to switch to that Mendix version and enable that retry mechanism in the mentioned microflow.
- SUB_Webhook_SendCallout: This microflow will actually send the callouts. The callouts are all sent asynchronously.

#### Webhook lifecycle
The module supports a webhook lifecycle that includes registering, updating, verifying, activating and deactivating. This section briefly covers all and might be a good reference for consumers of the webhook REST service.

##### Registering a webhook
To register a webhook, you need to provide a url that will eventually receive the callouts. Furthermore, you can add the optional Authorization object, this object holds the credentials that are required to call the url you provided. There are 3 options:
- Basic: Use this if your url requires basic username/password authorization.
- Custom: Use this if your url requires a custom header to authorize.
- ClientCredentials: Use this if you url requires a bearer token as authorization.

##### Verifying a webhook
After registering a webhook, the webhook will be unverified, which means that the url provided in the initial creation call is not verified yet. For security reasons, a verification step is required that checks whether you actually own the url you provided. Only after verification will you receive callouts to that webhook url. The verification will happen as follows:
- The client calls the verify endpoint.
- The verify endpoint will perform a GET call to the webhook url (as oppossed to the normal POST call for callouts). This GET call includes a custom header with a secret (e.g. X-SampleImplementation-Challenge). This challenge header should be determined when implementing the webhook module.
- The client needs to respond to that GET call with a 200 OK and a JSON body that contains the secret of the challenge header:
  {"Verification": "ValueFromTheChallengeHeader"}
When this process is completed, the webhook is set to Active and it is ready to receive notifications.

##### Updating a webhook
The PATCH endpoint allows for updating the webhook. Note that all potentially breaking changes will change the webhook status to Unverified. This means that you will need to verify the webhook again after PATCHing in some cases.

##### (De)activating a webhook
To temporarily stop receiving notifications, you can deactive a webhook via the /deactivate endpoint. If you want to receive notifications again, you can activate it without the need for verification via the /activate endpoint.

  

