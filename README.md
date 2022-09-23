# Welcome

Welcome to the Gojek VerneMQ documentation! This project is a fork of the awesome ❤️  [MQTT broker](https://github.com/vernemq/vernemq) of the same name. We want to add some functionalities that we think can be useful for specific use-cases. This is a reference guide for most of the available features and options of VerneMQ. The [Getting Started guide](getting-started.md) might be a good entry point.

## Summary of primary additions over core VerneMQ (1.11.0)

We have added the following features over the core VerneMQ (1.11.0) features. These features have been added as a result of few drawbacks that we felt should be addressed better.
- This project provides an option to disable persistence: [no op engine](configuration/noopengine.md)
- [Redis based subscription store](configuration/routing.md)
- [jwt based authentication and token based ACLs](configuration/customauth.md)
- [Events sidecar plugin](plugindevelopment/eventssidecarplugins.md)


## How to help improve this documentation

The [VerneMQ Documentation project](https://github.com/gojekfarm/vmq-docs) is an open-source effort, and your contributions are very welcome and appreciated. 
You can contribute on all levels:
- Language, style and typos
- Fixing obvious documentation errors and gaps
- Providing more details and/or examples for specific topics
- Extending the documentation where you find this useful to do
