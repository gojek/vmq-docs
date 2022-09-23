---
description: How to implement VerneMQ plugins using a TCP sidecar
---

# Webhooks

The VerneMQ Events sidecar plugin provides an easy and flexible way to relay events on VerneMQ using tcp hooks. 

The idea of VerneMQ Events sidecar id very simple: you can register an TCP endpoint with a VerneMQ plugin hook and whenever the hook \(such as `auth_on_register`\) is called, the VerneMQ Events sidecar dispatches a TCP request to the registered endpoint. Note that unlike the webhook plugin, this is an async plugin which only relays the events and no control flow is done based on responses returned.

## Configuring webhooks

To enable webhooks make sure to set:

```text
plugins.vmq_events_sidecar = on
```

And then each webhook can be configured like this:

```text
vmq_events_sidecar.hostname = 127.0.0.1
vmq_events_sidecar.port = 8890
vmq_events_sidecar.pool_size = 100
vmq_events_sidecar.hooks = on_register,on_subscribe
```

It is also possible to dynamically register webhooks at run-time:

```text
$ vmq-admin events enable register hook=on_register
```

See which endpoints are registered:

```text
$ vmq-admin events show
```

And finally deregistering an endpoint:

```text
$ vmq-admin events disable hook=on_register
```

{% hint style="info" %}
We recommend placing the endpoint implementation locally on each VerneMQ node such that each request can go over localhost without being subject to network issues.
{% endhint %}

## Connection pool configuration

The plugin uses by default a connection pool containing maximally 100 connections. This can be changed by setting `vmq_events_sidecar.pool_size` to a different value.

## Sidecar specs

For detailed information about the hooks and when they are called, see the sections [Session Lifecycle](sessionlifecycle.md), [Subscribe Flow](subscribeflow.md) and [Publish Flow](publishflow.md).

The tcp requests use the following codec:
```text
ByteOrder:           binary.BigEndian,
LengthFieldOffset:   0,
LengthFieldLength:   4,
LengthAdjustment:    0,
InitialBytesToStrip: 4,
```

### on_client_gone


Sidecar payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnClientGone {
    google.protobuf.Timestamp timestamp = 1;
    string client_id = 2;
    string mountpoint = 3;
}

```

### on_client_offline

Payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnClientOffline {
    google.protobuf.Timestamp timestamp = 1;
    string client_id = 2;
    string mountpoint = 3;
}

```


### on_client_wakeup

Payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnClientWakeUp {
    google.protobuf.Timestamp timestamp = 1;
    string client_id = 2;
    string mountpoint = 3;
}

```


### on\_deliver

Payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnDeliver {
    google.protobuf.Timestamp timestamp = 1;
    string username = 2;
    string client_id = 3;
    string mountpoint = 4;
    string topic = 5;
    int32 qos = 6;
    bool is_retain = 7;
    bytes payload = 8;
}

```

### on\_delivery_complete

Payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnDeliveryComplete {
  google.protobuf.Timestamp timestamp = 1;
  string username = 2;
  string client_id = 3;
  string mountpoint = 4;
  string topic = 5;
  int32 qos = 6;
  bool is_retain = 7;
  bytes payload = 8;
}

```

### on\_offline_message

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnOfflineMessage {
    google.protobuf.Timestamp timestamp = 1;
    string client_id = 2;
    string mountpoint = 3;
    int32 qos = 4;
    string topic = 5;
    bytes payload = 6;
    bool retain = 7;
}

```

### on_subscribe
Payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnSubscribe {
    google.protobuf.Timestamp timestamp = 1;
    string client_id = 2;
    string mountpoint = 3;
    string username = 4;
    repeated TopicInfo topics = 5;
}

message TopicInfo {
    string topic = 1;
    int32 qos = 2;
}

```

### on\_unsubscribe

Payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnUnsubscribe {
    google.protobuf.Timestamp timestamp = 1;
    string username = 2;
    string client_id = 3;
    string mountpoint = 4;
    repeated string topics = 5;
}

```

### on\_publish

Payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

option go_package = "source.golabs.io/courier/apis-go/eventssidecar/v1";

package eventssidecar.v1;

message OnPublish {
    google.protobuf.Timestamp timestamp = 1;
    string username = 2;
    string client_id = 3;
    string mountpoint = 4;
    int32 qos = 5;
    string topic = 6;
    bytes payload = 7;
    bool retain = 8;
}

```

## on_register

Payload format:
```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnRegister {
    google.protobuf.Timestamp timestamp = 1;
    string peer_addr = 2;
    int32 peer_port = 3;
    string username = 4;
    string mountpoint = 5;
    string client_id = 6;
    map<string, string> user_properties = 7;
}

```

### on_session_expired

Payload format:
```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";

package eventssidecar.v1;

message OnSessionExpired {
    google.protobuf.Timestamp timestamp = 1;
    string client_id = 2;
    string mountpoint = 3;
}
```
