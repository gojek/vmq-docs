# Auth using files

## Custom Auth Plugin

We introduced a new plugin that adds to the current capabilities of VerneMQ. These include a JWT based authentication and an ACL validator that takes inspiration from ACL auth provided by the core project.

```text
plugins.vmq_gojek_auth = on
```

## Authentication

You have the option of configuring jwt based authentication for clients. You need to connect with a password that can be verified with the `secret_key`.
The plugin matches the value of the `rid` key against the username to verify if this is an authentic user.


```text
vmq_gojek_auth.enable_jwt_auth = true
vmq_gojek_auth.secret_key = some_cool_secret
```

## Authorization

ACL based authorization mechanism is also provided that extends the current scope:

VerneMQ periodically checks the specified ACL file.

```text
vmq_gojek_auth.enable_acl_hooks = true
```

The check interval defaults to 10 seconds and can also be defined in the `vernemq.conf`.

```text
vmq_gojek_auth.acl_reload_interval = 10
vmq_gojek_auth.acl_file = ./etc/vmq.acl
```

Setting the `acl_reload_interval = 0` disables automatic reloading.

{% hint style="info" %}
Both configuration parameters can also be changed at runtime using the `vmq-admin` script.
{% endhint %}

### Managing the ACL entries

Topic access is added with lines of the format:

```text
topic [read|write] <topic> label <acl_name>
```

{% hint style="info" %}
Only one space should be put between the topic and the preceeding keyword. Extra spaces will be interpreted as part of the topic! Also note that the ACL parser doesn't accept empty lines between entries.
{% endhint %}

The access type is controlled using `read` or `write`. If not provided then read an write access is granted for the `topic`. The `topic` can use the MQTT subscription wildcards `+` or `#`.

To make it easier to identify ACL entries, we can assign a name to them by adding a value after the `label` keyword. This label will be mapped with the topic and referred to as `acl_name` during the ACL parsing. The `acl_name` is added following events hooks such as `on_subscribe`, `on_publish`, `on_deliver`, `on_delivery_complete`, and `on_message_drop`. This allows for better instrumentation and observability.

If the `label <name>` is not provided to the entry, the ACL parser will map the `acl_name` as an empty value with the topic.

The first set of topics are applied to all anonymous clients \(assuming `allow_anonymous = on`\). User specific ACLs are added after a user line as follows \(this is the username not the client id\):

```text
user <username>
```

It is also possible to define ACLs based on pattern substitution within the the topic. The form is the same as for the topic keyword, but using pattern as the keyword.

```text
pattern [read|write] <topic> label <acl_name>
```

The patterns available for substitution are:

> * `%c` to match the client id of the client
> * `%u` to match the username of the client

The substitution pattern must be the only text for that level of hierarchy. Pattern ACLs apply to all users even if the **user** keyword has previously been given.


This project extends these capabilities provided by vmq_acl . It is possible to only match certain parts of client-id or username using token as the keyword.

```text
token [read|write] <topic> label <acl_name>
```

For example:

> * `%(c, :, 3)` to match the 3rd token extracted from client id of the client after delimiting using `:`.
> * `%(u, -, 2)` to match the 2nd token from username of the client after delimiting using `-`.

The semantics are as follows:
```text
%((username or client-id identifier), delimiter, token number)
```

### Simple ACL Example

```text
# ACL for anonymous clients
topic bar label topic_bar
topic write foo label topic_foo
topic read open_to_all label open
token read foo/%(u, :, 3)/bar label foo_client_bar


# ACL for user 'john'
user john
topic foo label john_foo
topic read bar label john_bar
topic write open_to_all label john_open_to_all
```

Anonymous users are allowed to

* publish & subscribe to topic bar and labelled as `topic_bar`.
* publish to topic foo and labelled as `topic_foo`. 
* subscribe to topic open_to_all and labelled as `open`.
* subscribe to topic foo/<whatever their 3rd token is after delimiting using>/bar and labelled as `foo_client_bar`.

User john is allowed to

* publish & subscribe to topic foo and labelled as `john_foo`.
* subscribe to topic bar and labelled as `john_bar`.
* publish to topic open_to_all and labelled as `john_open_to_all`.

Note that the ACL file parser is picky on a couple of things. If you doubt whether your ACLs are fully applied, check for the following.

* Whitespaces at the end of topic names are part of the topic
* Avoid empty lines in the ACL file.
* But add a newline at the end of the ACL file.
