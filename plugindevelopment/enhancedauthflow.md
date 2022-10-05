# Enhanced Auth Flow

VerneMQ supports [enhanced authentication](http://docs.oasis-open.org/mqtt/mqtt/v5.0/cs02/mqtt-v5.0-cs02.html#_Toc514345528) flows or SASL style authentication for MQTT 5.0 sessions. The enhanced authentication mechanism can be used for initial authentication when the client connects or to re-authenticate clients at a later point.

![](../.gitbook/assets/enhanced_authflow5.svg)

The `on_auth_m5` hook allows the plugin to implement SASL style authentication flows by either accepting, rejecting \(disconnecting the client\) or continue the flow. The `on_auth_m5` hook is specified in the Erlang behaviour [on\_auth\_m5\_hook](https://github.com/gojekfarm/vernemq_dev/blob/master/src/on_auth_m5_hook.erl) in the [vernemq\_dev](https://github.com/gojekfarm/vernemq_dev) repo.

