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
topic [read|write] <topic>
```

{% hint style="info" %}
Only one space should be put between the topic and the preceeding keyword. Extra spaces will be interpreted as part of the topic! Also note that the ACL parser doesn't accept empty lines between entries.
{% endhint %}

The access type is controlled using `read` or `write`. If not provided then read an write access is granted for the `topic`. The `topic` can use the MQTT subscription wildcards `+` or `#`.

The first set of topics are applied to all anonymous clients \(assuming `allow_anonymous = on`\). User specific ACLs are added after a user line as follows \(this is the username not the client id\):

```text
user <username>
```

It is also possible to define ACLs based on pattern substitution within the the topic. The form is the same as for the topic keyword, but using pattern as the keyword.

```text
pattern [read|write] <topic>
```

The patterns available for substitution are:

> * `%c` to match the client id of the client
> * `%u` to match the username of the client

The substitution pattern must be the only text for that level of hierarchy. Pattern ACLs apply to all users even if the **user** keyword has previously been given.


This project extends these capabilities provided by vmq_acl . It is possible to only match certain parts of client-id or username using token as the keyword.

```text
token [read|write] <topic>
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
topic bar
topic write foo
topic read open_to_all
token read foo/%(u, :, 3)/bar


# ACL for user 'john'
user john
topic foo
topic read baz
topic write open_to_all
```

Anonymous users are allowed to

* publish & subscribe to topic bar.
* publish to topic foo.
* subscribe to topic open_to_all.
* subscribe to topic foo/<whatever their 3rd token is after delimiting using>/bar.

User john is allowed to

* publish & subscribe to topic foo.
* subscribe to topic baz.
* publish to topic open_to_all.

Note that the ACL file parser is picky on a couple of things. If you doubt whether your ACLs are fully applied, check for the following.

* Whitespaces at the end of topic names are part of the topic
* Avoid empty lines in the ACL file.
* But add a newline at the end of the ACL file.
