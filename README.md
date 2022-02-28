# Webhooks
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
6. You are ready to use the module.

## How to use the module

After configuration, you can use the module to send notifications via a webhook to the consumers. As mentioned before, this all happens asynchronously via the Mx9 task queue. This section has some more information on how to use the module.

#### Available microflow actions
- SUB_WebhookBody_Create: Use this action to create a JSON structure for your webhook. This JSON structure will be sent in each callout. It only has the minimal information, but you are of course able to modify this if needed.
- SUB_Webhook_FilterList: Use this action to apply filtering on the callouts to be sent. An example might be that clients only want to receive their own notifications, not notifications of objects owned by other clients. This microflow should be used to filter such scenario's.
- SUB_Client_GetCurrentClient: Currently, each webhook is associated with User. If you have another Entity that represents your API clients, you can use this microflow to select that entity instead.
- SUB_Callout_Retry: This microflow will be triggered if a callout fails. By default, the callout will be retried once. If you want a more sophisticated retry mechanism, you can add that in this microflow. TIP: Mendix 9.10 and above support a retry mechanism out of the box, the suggestion would be to switch to that Mendix version and enable that retry mechanism in the mentioned microflow.
- SUB_Webhook_SendCallout: This microflow will actually send the callouts. The callouts are all sent asynchronously.

#### Webhook lifecycle
