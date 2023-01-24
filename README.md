# Welcome

Welcome to the Gojek VerneMQ documentation! This project is a fork of the awesome ❤️  [MQTT broker](https://github.com/vernemq/vernemq) of the same name. We want to add some functionalities that we think can be useful for specific use-cases. This is a reference guide for most of the available features and options of VerneMQ. The [Getting Started guide](getting-started.md) might be a good entry point.

## Motivation of this fork
Courier, Gojek's MQTT based persistent connections stack, uses VerneMQ as its MQTT broker.
When using VerneMQ(v1.11.0) at scale our team faced numerous problems which hindered the availability, reliability and durability of the system.

Our latest [blog](https://blog.gojek.io/customising-vernemq-the-message-broker-for-our-information-superhighway/) lists all the problems and its solution in detail.

## Goal of this fork
Make the broker cluster stateless by moving the state out of the cluster.

## Summary of primary additions over core VerneMQ (1.11.0)
In order to achieve our goal, we have added the following features over the core VerneMQ (1.11.0) features. These features have been added as a result of few drawbacks that we felt should be addressed better.
- [Message passing amongst the cluster nodes is done via redis](configuration/messagepassing.md)
- [Store only offline messages instead of all the messages](configuration/genericofflinestore.md)
- [An option to disable persistence using no-op engine](configuration/noopengine.md)
- [Connection and subscription related client data is now stored in redis](configuration/routing.md)
- [JWT based authentication and token based ACL](configuration/customauth.md)
- [Events sidecar plugin using tcp](plugindevelopment/eventssidecarplugins.md)


## How to help improve this documentation

The [VerneMQ Documentation project](https://github.com/gojekfarm/vmq-docs) is an open-source effort, and your contributions are very welcome and appreciated. 
You can contribute on all levels:
- Language, style and typos
- Fixing obvious documentation errors and gaps
- Providing more details and/or examples for specific topics
- Extending the documentation where you find this useful to do
