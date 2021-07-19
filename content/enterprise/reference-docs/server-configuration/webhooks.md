---
title: Webhooks
menus:
  reference-docs:
    parent: Server Config
---

- The feature and the two events `document.published` and `document.unpublished` were added in [`release-2020-05`](https://github.com/livingdocsIO/livingdocs-release-notes/blob/master/releases/release-2020-05.md)
- Four new webhooks added in `release-2021-03`.
  - `document.update`
  - `mediaLibraryEntry.create`
  - `mediaLibraryEntry.archive`
  - `mediaLibraryEntry.update`
- One new webhook added in `release-2021-09`
  - `mediaLibraryEntry.revoke`

Webhooks are registered HTTP endpoints that are called on specific events.
You can configure multiple webhooks that are called on only one or multiple events.

## Configuration

The feature is enabled by default, you can disable it in your [server configuration]({{< ref "/enterprise/reference-docs/server-configuration#webhooks" >}}).

In the project menu, select "Project Setup", then select "Webhooks" in the "Configuration" section of the menu. You can activate/deactivate all your configured webhooks using the setting "deliver webhooks on events".

{{< img src="webhooks.png" alt="Webhooks Configuration" >}}


Add a new Webhook by clicking "Add Webhook" or edit an existing one by clicking on it's entry in the list. This shows a form to configure the webhook.

- `handle` needs to be unique on your project
- `label` is used for UI only
- `description` is only visible in this form and can be used to document the reason for this webhook or other humans to talk to for questions about the endpoint
- `url` will be called with a POST request when a selected event happens
- `secret` is used to sign the request. Use this to [secure your webhooks](#securing-your-webhooks)
- `active` needs to be `true` for this webhook to be called
- `events` let's you define which events should trigger your webhook

For testing purposes the service at https://webhook.site may come in handy. It gives you an URL you can use to send webhooks to and look at all the requests in a webinterface.

{{< img src="images/webhook-detail.png" alt="Webhooks Configuration" >}}

### Configuration storage

The configuration of webhooks is stored in `channelConfig.settings.webhooks`.

```js
webhooks: {
  active: true
  configurations: [{
    handle: 'my-webhook',
    label: 'My Webhook',
    description: 'A description for future self and coworkers',
    url: 'https://example.com/my-webhook-endpoint',
    secret: 'a-secret-token-to-sign-the-request'
    active: true
    events: [
      'document.publish',
      'document.unpublish',
      {
        name: 'document.update', 
        changeFilter: {
          metadataProperties: [
            'title'
          ]
        }
      },
      'mediaLibraryEntry.create',
      'mediaLibraryEntry.archive',
      'mediaLibraryEntry.revoke',
      'mediaLibraryEntry.update'
    ]
  }
}]
```

## Payload
The payload sent to your webhook endpoints looks like this. The `deliveryId` is unique for every call.

`document.publish`
```json
{
  "event": "document.publish",
  "deliveryId": "qy8qoxQPVCES1VDneg4FE",
  "projectId": 3,
  "projectHandle": "service",
  "webhookHandle": "handle",
  "documentId": 179,
  "publicationEvent": {
    "createdAt": "2021-02-16T18:04:50.203Z",
    "contentType": "regular",
    "documentType": "article",
    "eventType": "publish",
    "publicationId": 174,
    "projectId": 3,
    "channelId": 4,
    "documentId": 179
  }
}
```

`document.unpublish`
```json
{
  "event": "document.unpublish",
  "deliveryId": "p3QA1OhXkQGJepVTkdd1b",
  "projectId": 3,
  "projectHandle": "service",
  "webhookHandle": "handle",
  "documentId": 179,
  "publicationEvent": {
    "createdAt": "2021-02-16T18:04:51.285Z",
    "contentType": "regular",
    "documentType": "article",
    "eventType": "unpublish",
    "publicationId": 174,
    "projectId": 3,
    "channelId": 4,
    "documentId": 179
  }
}
```

`document.update`
```json
{
  "event": "document.update",
  "deliveryId": "B_-8BAfgvuJRKk7m_RSls",
  "projectId": 3,
  "projectHandle": "service",
  "webhookHandle": "handle",
  "documentId": 179,
  "metadataPropertyChanges": [
    "title"
  ]
}
```

`mediaLibraryEntry.create`
```json
{
  "event": "mediaLibraryEntry.create",
  "deliveryId": "f8K1HZ-_fTIf5kT698NqG",
  "projectId": 3,
  "projectHandle": "service",
  "webhookHandle": "handle",
  "mediaId": "PNIi08x4UdEA"
}
```

`mediaLibraryEntry.update`
```json
{
  "event": "mediaLibraryEntry.update",
  "deliveryId": "_p6aKe9Len6WnlvANDJSz",
  "projectId": 3,
  "projectHandle": "service",
  "webhookHandle": "handle",
  "mediaId": "PNIi08x4UdEA"
}
```

`mediaLibraryEntry.archive`
```json
{
  "event": "mediaLibraryEntry.archive",
  "deliveryId": "gmWn5OfOsZzEiY3FNRgA4",
  "projectId": 3,
  "projectHandle": "service",
  "webhookHandle": "handle",
  "mediaId": "PNIi08x4UdEA"
}
```

`mediaLibraryEntry.revoke`
```json
{
  "event": "mediaLibraryEntry.revoke",
  "deliveryId": "J1FrTJTNKKRoGmjBlQ3_e",
  "projectId": 3,
  "projectHandle": "service",
  "webhookHandle": "handle",
  "mediaId": "PNIi08x4UdEA"
}
```

## Securing your webhooks
If you have defined a `secret` for your webhook, Livingdocs uses this to create a signature of the payload and sends it with the request in the HTTP header `x-livingdocs-signature`.
The signature is created using HMAC-SHA256 and will be sent in `x-livingdocs-signature` in the form `sha256=<hex digest>` for example `sha256=d8a47af83666a771d57117aa28ef8d3243a3de43`.

Here is sample code in JavaScript to validate the signature in your endpoint:

```js
// set payload to the body received with the request to your endpoint
const payload = request.body
const signature = request.headers['x-livingdocs-signature']

// secret needs to be the same you configured with the webhook in livingdocs
const secret = 'a-secret-token-to-sign-the-request' // you should not hardcode this but read it from an environment variable

// compute the hmac on the received payload using the same secret
const crypto = require('crypto')
const hmac = crypto.createHmac('sha256', secret)
const payloadSignature = Buffer.from(`sha256=${hmac.update(payload).digest('hex')}`, 'utf8')
const checksum = Buffer.from(signature, 'utf8')

// use timingSafeEqual to compare the signature sent from livingdocs with the computed checksum
if (crypto.timingSafeEqual(payloadSignature, checksum)) {
  // payload is valid, move on
} else {
  // abort, this is not a valid request
}
```
