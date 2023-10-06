# ApiRTC web-agent web application

This **web-agent web application** is intended to be integrated within any third party web application through an iframe.

## Getting started

The application is hosted [here](https://apirtc.github.io/web-agent/). Without parameters, it does not much. Url parameters must be set to control it.

The first one, _aK_ : is the **apiKey**, which can be obtained from [ApiRtc.com](https://apirtc.com).

Then, _cN_ specifies the **Conversation** **name**.

Specifying [?aK=myDemoApiKey&cN=Test](https://apirtc.github.io/web-agent?aK=myDemoApiKey&cN=Test) shall make the application ready to create an invitation link to a **Conversation** named _Test_.

**Note** : _myDemoApiKey_ shall be used for this demo only, and may not allow to use all features (such as short-messages invitation for example).

## Integrate as iframe

To integrate the application within html, use an iframe as below :

```html
<iframe
  src="https://apirtc.github.io/web-agent?aK=myDemoApiKey&cN=Test"
  height="720px"
  width="100%"
  referrerpolicy="no-referrer"
  allow="geolocation;autoplay;microphone;camera;display-capture;midi;encrypted-media;clipboard-write;"
></iframe>
```

## Url parameters

| Parameter | stands for           | Default                | Description                                                                   |
| --------- | -------------------- | ---------------------- | ----------------------------------------------------------------------------- |
| aA        | allowAudio           | true                   | set to false to disable audio usage                                           |
| aK        | apiKey               | none                   | [ApiRtc](https://apirtc.com) **apiKey**, mandatory                       |
| aU        | assistedUrl          | ASSISTED_URL           | url of the web-assisted web application                                       |
| c         | connect              | true                   | Connect with **ApiRtc** platform                                              |
| cN        | conversationName     | empty                  | the **ApiRtc** **Conversation** **name**                                      |
| cU        | cloudUrl             | CLOUD_URL              | the cloud url                                                                 |
| iI        | installationId       | INSTALLATION_ID        | used a header for local-storage keys                                          |
| j         | join                 | true                   | Join the **Conversation**                                                     |
| gN        | guestName            | empty                  | name to be pre-set in the invitation form                                     |
| gP        | guestPhone           | empty                  | phone number to be pre-set in the invitation form                             |
| iU        | invitationServiceUrl | INVITATION_SERVICE_URL | url of the invitation service                                                 |
| l         | locale               | browser config         | to force locale to _fr_ or _en_                                               |
| lL        | logLevel             | warn                   | can be debug, info, warn, error                                               |
| uId       | userId               | generated by apirtc    | id of the user-agent that the application will use to connect with **ApiRtc** |

Constants values can be found in generated doc [here](https://apirtc.github.io/web-agent/doc/modules/public_constants.html).

## Dynamic control

Using url parameters to provide information to the iframe-d **web-agent** application is good for static and/or initial values. But the values can't be changed without reloading.

However, the hosting application may need to change the **Conversation** **name**, or update guest phone number with no reload.

Also, the **web-agent** application has real-time features that the hosting application may need to be notified of. For example, when a snapshot is taken, this is up to the hosting application to handle it (just display it, or upload it to a database server).

To enable such **real-time interaction**, the **web-agent** application implements a **dual channel communication**, using standard [Window: postMessage](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) mechanism.

### web-agent to host communication

In order for host application to receive messages from **web-agent**, it has to register a **window** listener for _message_ events:

```js
const IFRAME_HOST = "https://apirtc.github.io";
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
| joined             | N/A              | **Conversation** was joined                                                   |
| left               | N/A              | **Conversation** was left                                                     |
| ready              | N/A              | notifies when **web-agent** is ready to receive messages                      |
| sms_fail           | phone,name,link  | notifies when an sms post failed                                              |
| sms_sent           | phone,name,link  | notifies an sms has been sent                                                 |
| snapshot           | contact, dataUrl | notifies when a snapshot taken on a **Stream** from a **Contact** is received |
| subscribed_streams | length           | fired every time the number of subscribed streams changes                     |

### Host to web-agent communication

Hosting application can post a message to **web-agent** by getting a handle on the iframe object and use _postMessage_:

```js
iframe.contentWindow.postMessage(
  {
    type: "conversation",
    name: "new_conversation_name",
  },
  IFRAME_HOST
);
```

#### Complete list of messages

| message type  | field(s):type  | Description                     |
| ------------- | -------------- | ------------------------------- |
| configuration | data:AppConfig | configures application          |
| connect       | N/A            | connection with apirtc platform |
| conversation  | name:string    | set **Conversation** name       |
| disconnect    | N/A            | disconnect from apirtc platform |
| guest_data    | data:UserData  | set guest data                  |
| join          | N/A            | join **Conversation**           |
| leave         | N/A            | leave **Conversation**          |
| user_data     | data:UserData  | set user data                   |

The fields complex types definitions can be found [here](https://apirtc.github.io/web-agent/doc/modules/types.html).

Note: host application must wait for having received the _ready_ message from **web-agent** before posting messages.

## Sample

Visit [sample page](https://apirtc.github.io/web-agent/sample.html) for a demonstration of dynamic control of the iframe-d **web-agent** app !

Also available on [CodePen](https://codepen.io/kmoyse/pen/wvRYdLG)
