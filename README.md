# Apizee web-agent web application

This **web-agent web application** is intended to be integrated within any third party web application through an i-frame.

## Getting started

The application is hosted [here](https://kmoyse-apizee.github.io/web-agent/). As is, it does not much. Some url parameters must be set to control it.

A mandatory one, *Ak* : is the **apiKey**, which you can get from [ApiRtc](https://apirtc.com).

Then *cN* specifies the **Conversation** **name**.

Finally *c* and *j*, set to true allow to both **connect** ApiRtc platform and **join** the **Conversation**.

So specifying [?aK=myDemoApiKey&c=true&j=true&cN=Test](https://kmoyse-apizee.github.io/web-agent?aK=myDemoApiKey&c=true&j=true&cN=Test) shall display more.

**Note** : _myDemoApiKey_ shall be used for this demo only, and may not allow to use all features (such as short-messages invitation for example).

## Integrate as iframe

To integrate the application with html through iframe you should do something like :

```html
<iframe
  src="https://kmoyse-apizee.github.io/web-agent?aK=myDemoApiKey&c=true&j=true&cN=Test"
  height="720px"
  width="100%"
  referrerpolicy="no-referrer"
  sandbox="allow-forms allow-popups allow-popups-to-escape-sandbox allow-scripts allow-same-origin allow-downloads"
  allow="geolocation;autoplay;microphone;camera;display-capture;midi;encrypted-media;clipboard-write;"
></iframe>
```

## Url parameters

| Parameter | stand for            | Description                                                                   |
| --------- | -------------------- | ----------------------------------------------------------------------------- |
| aK        | apiKey               | your [ApiRtc](https://apirtc.com) **apiKey**, mandatory                       |
| aU        | assistedUrl          | url of the web-assisted web application                                       |
| cN        | conversationName     | the **ApiRtc** **Conversation** **name**                                      |
| cU        | cloudUrl             | the cloud url, defaults to https://cloud.apirtc.com                           |
| iI        | installationId       | used a header for local-storage keys                                          |
| gN        | guestName            | name to be pre-set in the invitation form                                     |
| gP        | guestPhone           | phone number to be pre-set in the invitation form                             |
| iU        | invitationServiceUrl | url of the invitation service                                                 |
| l         | locale               | to force locale to fr or en                                                   |
| lL        | logLevel             | can be debug, info, warn, error                                               |
| uId       | userId               | id of the user-agent that the application will use to connect with **ApiRtc** |

All parameters are optional, except **aK**.

## Dynamic control

Using url parameters to provide information to the i-framed web-agent application is good for static and/or default values. But the values can't be changed without reloading.

For example, the hosting application may need to change the **Conversation** **name** without reloading the i-framed web-agent application.

Also, the web-agent application has real-time features that the hosting application may need to be notified of. For example, when a snapshot is taken, this is up to the hosting application to handle it (simple display it, or upload it to a database server, ...).

To enable such real-time interaction, the web-agent application implements a dual channel communication, using standard [Window: postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) mechanism.

### web-agent to host communication

In order for host application to receive messages from web-agent, it needs to register a **window** listener for _message_ events:

```js
const receiveMessage = (event) => {
  if (event.origin !== IFRAME_HOST) return;

  const message = event.data;

  switch (message.type) {
    case "ready": {
      // web-agent app is ready to receive messages
      break;
    }
    case "snapshot": {
      const dataUrl = message.dataUrl;
      // display or store the snapshot dataUrl.
      break;
    }
    ...
    default:
      console.warn(`Unhandled message.type ${message.type}.`);
  }
};

window.addEventListener("message", receiveMessage);
```

The actual **message** is an object in _event.data_. A **message** has a _type_ field, and other fields depending on the _type_ value.

#### Complete list of messages

| message type       | field(s)         | Description                                                                   |
| ------------------ | ---------------- | ----------------------------------------------------------------------------- |
| ready              | N/A              | notifies when web-agent is ready to receive messages                          |
| snapshot           | contact, dataUrl | notifies when a snapshot taken on a **Stream** from a **Contact** is received |
| subscribed_streams | length           | fired every time the number of subscribed streams changes                     |

### Host to web-agent communication

To post a message, get a handle on the iframe object and use postMessage like:

```js
iframe.contentWindow.postMessage(
  {
    type: "conversation",
    name: "new_conversation_name"
  },
  IFRAME_HOST
);
```

#### Complete list of messages

| message type       | field(s)       | Description                     |
| ------------------ | -------------- | ------------------------------- |
| configuration      | data:AppConfig | configures application          |
| connect            | N/A            | connection with apirtc platform |
| conversation       | name:string    | set **Conversation** name       |
| disconnect         | N/A            | disconnect from apirtc platform |
| guest_data         | data:UserData  | set guest data                  |
| join               | N/A            | join **Conversation**           |
| leave              | N/A            | leave **Conversation**          |
| user_data          | data:UserData  | set user data                   |

TODO: provide links to a auto-generated doc for AppConfig, UserData types

Note: host application must wait for having received the 'ready' message from web-agent before posting messages.

## Sample

Visit [sample page](https://kmoyse-apizee.github.io/web-agent/sample.html) for a demonstration of dynamic control of the i-framed web-agent app !

The code is available in *sample/sample.html* file.