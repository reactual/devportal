---
heading: Hubspot Marketing
seo: Authenticate | Hubspot Marketing | Cloud Elements API Docs
title: Authenticate 
description: Authenticate an element instance with Hubspot Marketing
layout: sidebarelementdoc
breadcrumbs: /docs/elements.html
elementId: 43
elementKey: hubspot
parent: Back to Element Guides
order: 20
---

# Authenticate with {{page.heading}}

You can authenticate with {{page.heading}} to create your own instance of the {{page.heading}} element through the UI or through APIs. Once authenticated, you can use the element instance to access the different functionality offered by the {{page.heading}} platform.

{% include callout.html content="<strong>On this page</strong></br><a href=#authenticate-through-the-ui>Authenticate Through the UI</a></br><a href=#authenticate-through-api>Authenticate Through the API</a></br><a href=#parameters>Parameters</a></br><a href=#example-response>Example Response</a>" type="info" %}

## Authenticate Through the UI

Use the UI to authenticate with {{page.heading}} and create an element instance. {{page.heading}} authentication follows the typical OAuth 2 framework and you will need to sign in to {{page.heading}} as part of the process.

If you are configuring events, see the [Events section](events.html).

To authenticate an element instance:

1. Sign in to Cloud Elements, and then search for the element in our Elements Catalog.

    | Latest UI | Earlier UI  |
    | :------------- | :------------- |
    |  ![Search](../img/Element-Search2.png)  |  ![Search](../img/Element-Search.png)  |

3. Create an element instance.

    | Latest UI | Earlier UI  |
    | :------------- | :------------- |
    | Hover over the element card, and then click __Create Instance__.</br> ![Create Instance](../img/Create-Instance.gif)  | Click __Add Instance__.</br> ![Search](../img/Add-Instance.png)  |

5. Enter a name for the element instance.

6. Enter you Hubspot portal id. 

7. Click __Create Instance__ (latest UI) or __Next__ (earlier UI).
8. Optionally add tags in the earlier UI:
     1. On the Tag It page, enter any tags that might help further define the instance.
      * To add more than one tag, click __Add__ after each tag.
      ![Add tag](../img/Add-Tag.png)
     1. Click __Done__.
8. Provide your Hubspot credentials, and then allow the connection.
9. Note the **Token** and **ID** and save them for all future requests using the element instance.
8. Take a look at the documentation for the element resources now available to you.

## Authenticate Through API

Authenticating through API is a multi-step process that involves:

* [Getting a redirect URL](#getting-a-redirect-url). This URL sends users to the vendor to log in to their account.
* [Authenticating users and receiving the authorization grant code](#authenticating-users-and-receiving-the-authorization-grant-code). After the user logs in, the vendor makes a callback to the specified url with an authorization grant code.
* [Authenticating the element instance](#authenticating-the-element-instance). Using the authorization code from the vendor, authenticate with the vendor to create an element instance at Cloud Elements.

### Getting a Redirect URL

Use the following API call to request a redirect URL where the user can authenticate with the service provider. Replace `{keyOrId}` with the element key, `{{page.elementKey}}`.

```bash
curl -X GET "/elements/{keyOrId}/oauth/url?apiKey=<api_key>&apiSecret=<api_secret>&callbackUrl=<url>&siteAddress=<url>"
```

#### Query Parameters

| Query Parameter | Description   |
| :------------- | :------------- |
| apiKey | The key obtained from registering your app with the provider. This is the **Consumer Key** that you noted at the end of the [Service Provider Setup section](setup.html).  |
| apiSecret |  The secret obtained from registering your app with the provider.  This is the **Consumer Secret** that you noted at the end of the [Service Provider Setup section](setup.html).   |
| callbackUrl | The URL that will receive the code from the vendor to be used to create an element instance. This is the **Callback URL** that you noted at the end of the [Endpoint Setup section](setup.html).  |

#### Example cURL

```bash
curl -X GET "https://api.cloud-elements.com/elements/api-v2/elements/{{page.elementKey}}/oauth/url?apiKey=fake_api_key&apiSecret=fake_api_secret&callbackUrl=https://www.mycoolapp.com/auth&state={{page.elementKey}}" -H  "accept: application/json" -H  "content-type: application/json"
```

#### Example Response

Use the `oauthUrl` in the response to allow users to authenticate with the vendor.

```json
{
"element": "{{page.elementKey}}",
"oauthUrl":"https://app.hubspot.com/oauth/authorize?client_id=fake_api_key&redirect_uri=https%3A%2F%2Fwww.mycoolapp.com%2Fauth&state=hubspot&scope=contacts%20timeline%20files%20content%20reports%20social%20automation%20forms"
}
```

### Authenticating Users and Receiving the Authorization Grant Code

Provide the response from the previous step to the users. After they authenticate, {{page.heading}} provides the following information in the response:

* code
* state

| Response Parameter | Description   |
| :------------- | :------------- |
| code | The Authorization Grant Code required by Cloud Elements to retrieve the OAuth access and refresh tokens from the endpoint.|
| state | A customizable identifier, typically the element key (`{{page.elementKey}}`) . |

{% include note.html content="If the user denies authentication and/or authorization, there will be a query string parameter called <code>error</code> instead of the <code>code</code> parameter. In this case, your application can handle the error gracefully." %}

### Authenticating the Element Instance

Use the `/instances` endpoint to authenticate with Hubspot and create an element instance. If you are configuring events, see the [Events section](events.html).

{% include note.html content="The endpoint returns an Element Instance token upon successful completion. Retain the token for all subsequent requests involving this element instance.  " %}

To create an element instance:

1. Construct a JSON body as shown below (see [Parameters](#parameters)):

    ```json
    {
      "element": {
        "key": "{{page.elementKey}}"
      },
      "providerData": {
        "code": "<AUTHORIZATION_GRANT_CODE>"
      },
      "configuration": {
        "oauth.callback.url": "<CALLBACK_URL>",
        "oauth.api.key": "<CONSUMER_KEY>",
      	"oauth.api.secret": "<CONSUMER_SECRET>",
        "filter.response.nulls": true
      },
      "tags": [
        "<Add_Your_Tag>"
      ],
      "name": "<INSTANCE_NAME>"
    }
    ```

1. Call the following, including the JSON body you constructed in the previous step:

        POST /instances

    {% include note.html content="Make sure that you include the User and Organization keys in the header. See <a href=index.html#authenticating-with-cloud-elements>the Overview</a> for details. " %}

1. Locate the `token` and `id` in the response and save them for all future requests using the element instance.

#### Example cURL

```bash
curl -X POST \
  https://api.cloud-elements.com/elements/api-v2/instances \
  -H 'authorization: User <USER_SECRET>, Organization ,ORGANIZATION_SECRET>' \
  -H 'content-type: application/json' \
  -d '{
  "element": {
    "key": "{{page.elementKey}}"
  },
  "providerData": {
    "code": "<AUTHORIZATION_GRANT_CODE>"
  },
  "configuration": {
    "oauth.callback.url": "<CALLBACK_URL>",
    "oauth.api.key": "<CONSUMER_KEY>",
    "oauth.api.secret": "<CONSUMER_SECRET>"
  },
  "tags": [
    "For Docs"
  ],
  "name": "Hubspot_Instance"
}'
```
## Parameters

API parameters not shown in the {{site.console}} are in `code formatting`.

{% include note.html content="Event related parameters are described in <a href=events.html>Events</a>." %}

| Parameter | Description   | Data Type |
| :------------- | :------------- | :------------- |
| key | The element key.<br>{{page.elementKey}}  | string  |
|  Name</br>`name` |  The name for the element instance created during authentication.   | Body  |
| `oauth.callback.url` | The Callback URL from Salesforce. This is the Callback URL that you noted at the end of the [Endpoint Setup section](salesforce-endpoint-setup.html).  |
| `oauth.api.key` | The Consumer Key from Salesforce. This is the Consumer Key that you noted at the end of the [Endpoint Setup section](salesforce-endpoint-setup.html) |  string |
| `oauth.api.secret` | The Consumer Secret from Salesforce. This is the Consumer Secret that you noted at the end of the [Endpoint Setup section](salesforce-endpoint-setup.html)| string |
| Filter null values from the response </br>`filter.response.nulls` | *Optional*. Determines if null values in the response JSON should be filtered from the response. Yes or `true` indicates that Cloud Elements will filter null values. </br>Default: `true`.  | boolean |
| tags | *Optional*. User-defined tags to further identify the instance. | string |

## Example Response

```json
{
    "id": 123,
    "name": "test",
    "token": "redacted",
    "element": {
      "id": 43,
      "name": "Hubspot Marketing",
      "key": "hubspot",
      "description": "Add a HubSpot Instance to connect your existing HubSpot account to the Marketing Hub, allowing you to manage accounts, campaigns, contacts, leads etc. across multiple Marketing Elements. You will need your HubSpot account information to add an instance.",
      "image": "elements/provider_hubspot.png",
      "active": true,
      "deleted": false,
      "typeOauth": true,
      "trialAccount": false,
      "configDescription": "If you do not have a HubSpot account, you can create one at <a href=\"http://www.hubspot.com\" target=\"_blank\">HubSpot Signup</a>"
    },
    "provisionInteractions": [],
    "valid": true,
    "eventsEnabled": true,
    "disabled": false,
    "maxCacheSize": 0,
    "cacheTimeToLive": 0,
    "cachingEnabled": false
}
```