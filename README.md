# Flex Device SDK for Node.js

The Device SDK allows field devices to send Dynamic Demand data to the Open Energi cloud (Flex) and subscribe to Portfolio Management signals.

## Installation

`npm install oe-flex-sdk`.

## Usage

### Connecting to the Message Broker

Given a Hub URL, Device Id and SAS token:

```javascript
var DeviceClient = require('flex-device-client');

   
client = DeviceClient("Hub URL", "<Device Id>", "<SAS Token>");
client.connect();

```

### Sending a Message

Refer to *Message.md* for details on the different message types.

**Reading**

```javascript
var Reading = require('flex-message').Reading;

msg = Reading()
		.type(Reading.Types.POWER)
		.entity("l1234")
		.value(13.4);

client.publish(msg);
```

Even though the example above uses a reading type from an enumeration, any string less than 64 characters in length can be passed into the `type` method. Note that types are not case-sensitive and will be lowercased during ingestion.

Note that even if the `publish` method returns successfully, the message is not guaranteed to have been delivered. To be certain you need to use a callback to correlate acknowledged messages from the broker with published messages from your device - see "Acknowledgement" below.

**Event**

```javascript
var Event = require('flex-message').Event;

msg = Event()
		.type("state-of-charge")
		.level(Event.Levels.WARN)
		.entity("s12")
		.value("State of charge below 10% for the last 5 minutes");

client.publish(msg); 
```

Note that although the above example uses a custom event type, the `Event.Types` class contains the FFR event types documented in *Message.md*.

Note also that even if the `publish` method returns successfully, the message is not guaranteed to have been delivered. To be certain you need to use a callback to correlate acknowledged messages from the broker with published messages from your device - see "Acknowledgement" below.

**Schedule**

*Note yet implemented - coming soon*

### Receiving Acknowledgements

To receive an acknowledgement when your message has been successfully received, set the `on_publish` property of the client. 

```javascript
client.onPublish = function(message){
	console.log("Message delivered", message);	
};
```

### Overriding the Message Timestamp

By default, the `timestamp` field of the message will be set to the current system time when the message constructor (eg. `Reading()`) is invoked. You can override this with a Date:

```javascript
msg = Reading().timestamp(new Date());
```
    
### Setting the `created_at` field:

By default the `created_at` field is not populated. To set the `created_at` field using a `datetime` object:

```javascript
msg = Event().createdAt(new Date());
```

## Receiving Messages

Devices are by default subscribed to Portfolio Management signals. These are further documented in *Message.md*. There are two callbacks: one for signal messages and one for schedule signal messages.

```javascript
client.onSignal(function(signal){
	console.log("Signal", signal);
	console.log("Signal current value", signal.currentValue());
});
client.onScheduleSignal(function(signal){
	console.log("Signal", signal);
	console.log("Time and value of the signal at next interval", signal.getNextChange());
});
```

**Disabling message subscription**

If you do not want to subscribe to cloud-to-device messages, you can call the `disableSubscription()` method. It can later be re-enabled with the `enableSubscription()` method.

```javascript  
client = DeviceClient("<Hub URL>", "<Device Id>", "<SAS Token>")
   
client.disableSubscription();
client.connect();
```