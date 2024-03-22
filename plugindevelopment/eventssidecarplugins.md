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

Supporting proto definitions for the events.

### disconnect_reason

```protobuf
syntax = "proto3";

package eventssidecar.v1;

enum Reason {
    REASON_UNSPECIFIED = 0;
    REASON_NOT_AUTHORIZED = 1;
    REASON_NORMAL_DISCONNECT = 2;
    REASON_SESSION_TAKEN_OVER = 3;
    REASON_ADMINISTRATIVE_ACTION = 4;
    REASON_DISCONNECT_KEEP_ALIVE = 5;
    REASON_DISCONNECT_MIGRATION = 6;
    REASON_BAD_AUTHENTICATION_METHOD = 7;
    REASON_REMOTE_SESSION_TAKEN_OVER = 8;
    REASON_MQTT_CLIENT_DISCONNECT = 9;
    REASON_RECEIVE_MAX_EXCEEDED = 10;
    REASON_PROTOCOL_ERROR = 11;
    REASON_PUBLISH_AUTH_ERROR = 12;
    REASON_INVALID_PUBREC_ERROR = 13;
    REASON_INVALID_PUBCOMP_ERROR = 14;
    REASON_UNEXPECTED_FRAME_TYPE = 15;
    REASON_EXIT_SIGNAL_RECEIVED = 16;
    REASON_TCP_CLOSED = 17;
}

```

### matched_acl

```protobuf
syntax = "proto3";

package eventssidecar.v1;

message MatchedACL {
    string name = 1;
    string pattern = 2;
}

```

Following are the proto definitions for all supported events.

### on_client_gone


Sidecar payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";
import "disconnect_reason.proto";

package eventssidecar.v1;

message OnClientGone {
    google.protobuf.Timestamp timestamp = 1;
    string client_id = 2;
    string mountpoint = 3;
    eventssidecar.v1.Reason reason = 4;
}

```

### on_client_offline

Payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";
import "disconnect_reason.proto";

package eventssidecar.v1;

message OnClientOffline {
    google.protobuf.Timestamp timestamp = 1;
    string client_id = 2;
    string mountpoint = 3;
    eventssidecar.v1.Reason reason = 4;
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
import "matched_acl.proto";

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
  MatchedACL matched_acl = 9;
  bool persisted = 10;
}

```

### on\_delivery_complete

Payload format:

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";
import "matched_acl.proto";

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
  MatchedACL matched_acl = 9;
  bool persisted = 10;
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
import "matched_acl.proto";

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
    MatchedACL matched_acl = 3;
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
import "matched_acl.proto";

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
    MatchedACL matched_acl = 9;
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

### on\_message_drop

```protobuf
syntax = "proto3";

import "google/protobuf/timestamp.proto";
import "matched_acl.proto";

package eventssidecar.v1;

message OnMessageDrop {
    google.protobuf.Timestamp timestamp = 1;
    string client_id = 2;
    string mountpoint = 3;
    int32 qos = 4;
    string topic = 5;
    bytes payload = 6;
    string reason = 7;
    MatchedACL matched_acl = 8;
}

```
## Example TCP events sidecar in Golang

Below is a very simple example of a tcp sidecar implemented in Go. It receives and logs OnSubscribe events. 
Follow this guide to generate Go code for on_subscribe event: https://developers.google.com/protocol-buffers/docs/reference/go-generated

Note that this example code uses compiled proto code for `on_subscribe` event in `protos` package. 

It runs a tcp server at port `8890` (default port for events sidecar plugin) that receives events and writes them on to a log file at `/tmp/sidecar.log`. 

```golang
package main

import (
	"encoding/binary"
	"errors"
	"fmt"
	"log"
	"net"
	"os"
	"sidecar/protos"

	"google.golang.org/protobuf/proto"
	"google.golang.org/protobuf/types/known/anypb"
)

const LOGFILE = "/tmp/sidecar.log"

func main() {
	logFile, err := os.OpenFile(LOGFILE, os.O_APPEND|os.O_RDWR|os.O_CREATE, 0644)
	if err != nil {
		log.Panic(err)
	}
	defer logFile.Close()

	// Set log out put and enjoy :)
	log.SetOutput(logFile)

	// optional: log date-time, filename, and line number
	log.SetFlags(log.Lshortfile | log.LstdFlags)

	addr := net.TCPAddr{
		Port: 8890,
		IP:   net.ParseIP("127.0.0.1"),
	}
	l, err := net.ListenTCP("tcp", &addr)
	if err != nil {
		log.Fatal(err)
	}

	d := Decoder{lengthFieldOffset: 4}

	for {
		conn, err := l.Accept()
		if err != nil {
			log.Println(err)
		}

		go func() {
			for {
				p := make([]byte, 2048)
				_, err := conn.Read(p)

				m, err := d.Decode(p)
				log.Println(m, err)
			}
		}()
	}
}

type Decoder struct {
	lengthFieldOffset int
}

func (decoder *Decoder) Decode(b []byte) (proto.Message, error) {
	payload := b[decoder.lengthFieldOffset:]

	length := binary.BigEndian.Uint32(b[:decoder.lengthFieldOffset])
	fmt.Println(length)

	if len(payload) != int(length) {
		fmt.Println("equals lengths")
		return nil, errors.New("length_field_error")
	}

	eventAny := new(anypb.Any)

	if err := proto.Unmarshal(payload, eventAny); err != nil {
		return nil, errors.New("unmarshal_error")
	}

	tm, ok := newProtoFuncMap[eventAny.GetTypeUrl()]

	if tm.name != "on_subscribe" || !ok {
		return nil, errors.New("type_mapper_not_defined")
	}

	onSubEvent := tm.new()
	if err := proto.Unmarshal(eventAny.GetValue(), onSubEvent); err != nil {
		return nil, errors.New("could_not_unmarshal_to_type_mapper")
	}

	return onSubEvent, nil
}

const typePrefix = "type.googleapis.com/eventssidecar.v1."

type TypeMapper struct {
	name string
	new  func() proto.Message
}

var newProtoFuncMap = map[string]*TypeMapper{
	typePrefix + "OnSubscribe": {
		name: "on_subscribe",
		new:  func() proto.Message { return new(protos.OnSubscribe) },
	},
}
```
