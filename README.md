# EasyMQTT

EasyMQTT is a lightweight wrapper around MQTT for Arduino-based robots and IoT devices.  
It simplifies device registration, connection management, heartbeat handling, and action-based communication between robots and clients.

The library was designed for small distributed robot systems controlled over MQTT.

## Features

- Easy WiFi + MQTT setup
- Automatic device registration
- Unique device IDs based on hardware serial data
- Built-in heartbeat/status system
- Simple client claiming / locking mechanism
- Timeout handling for disconnected controllers
- Action-based topic subscriptions
- Helper methods for payload parsing

## Dependencies

- WiFi101
- PubSubClient
- ArduinoJson

## Example Topic Structure

```text
robot/Robot-1A2B3C/register
robot/Robot-1A2B3C/status
robot/Robot-1A2B3C/control/claim
robot/Robot-1A2B3C/control/acknowledge
robot/Robot-1A2B3C/action/forward
robot/Robot-1A2B3C/ping
```

## Basic Usage
```c++
#include "EasyMQTT.h"

const char* actions[] = {
  "forward",
  "back",
  "left",
  "right"
};

EasyMQTT mqtt(
  "WIFI_NAME",
  "WIFI_PASSWORD",
  IPAddress(192,168,0,10),
  1883,
  "robot",
  actions,
  4
);

void setup() {
  mqtt.begin();

  mqtt.setCallback(callback);
}

void loop() {
  mqtt.mqtt_loop();
}

void callback(char* topic, uint8_t* payload, unsigned int length) {
  String topicStr = String(topic);
  String payloadStr = mqtt.convertPayloadToString(payload, length);

  mqtt.manageConnection(topicStr, payloadStr);

  if (mqtt.clientMatches(payloadStr)) {
    // Handle actions here
  }
}
```

## Device Registration

When a device connects, it automatically publishes a retained registration message:
```json
{
  "id": "Robot-1A2B3C",
  "type": "robot",
  "actions": ["forward", "left", "right"]
}
```

## Connection Model

Before sending actions, a client has to claim control of the robot:
```text
<type>/<id>/control/claim
```
The robot acknowledges the connection on:
```text
<type>/<id>/control/acknowledge
```

While connected, the client must periodically send pings or actions to keep the connection alive.

## Status Messages

The library periodically publishes retained status messages:
```json
{
  "online": true,
  "locked": false
}
```
or
```json
{
  "online": true,
  "locked": true
}
```
## Intended Use

EasyMQTT was built for:

- small robots
- distributed robotics experiments
- MQTT-controlled devices
- web-based robot control systems
- educational projects
