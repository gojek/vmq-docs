---
description: >-
  As well as being available as packages that can be installed directly into the
  operating systems, VerneMQ is also available as a Docker image. Below is an
  example of how to set up a couple of VerneMQ
---

# Running VerneMQ using Docker

## Start a VerneMQ cluster node

{% hint style="danger" %}
To use the provided docker images the VerneMQ EULA must be accepted. See [Accepting the VerneMQ EULA](accepting-the-vernemq-eula.md) for more information.
{% endhint %}

```text
docker run --name vernemq1 -d gojektech/vernemq
```

Sometimes you need to configure a forwarding for ports \(on a Mac for example\):

```text
docker run -p 1883:1883 --name vernemq1 -d gojektech/vernemq
```

This starts a new node that listens on 1883 for MQTT connections and on 8080 for MQTT over websocket connections. However, at this moment the broker won't be able to authenticate the connecting clients. To allow anonymous clients use the `DOCKER_VERNEMQ_ALLOW_ANONYMOUS=on` environment variable.

```text
docker run -e "DOCKER_VERNEMQ_ALLOW_ANONYMOUS=on" --name vernemq1 -d gojektech/vernemq
```

{% hint style="info" %}
Warning: Setting `allow_anonymous=on` completely disables authentication in the broker and plugin authentication hooks are never called! See more information about the authentication hooks [here](../plugindevelopment/sessionlifecycle.md#auth_on_register-and-auth_on_register_m5).
{% endhint %}

### VerneMQ Configuration

All configuration parameters that are available in `vernemq.conf` can be defined
using the `DOCKER_VERNEMQ` prefix followed by the confguration parameter name.
E.g: `allow_anonymous=on` is `-e "DOCKER_VERNEMQ_ALLOW_ANONYMOUS=on"` or
`allow_register_during_netsplit=on` is
`-e "DOCKER_VERNEMQ_ALLOW_REGISTER_DURING_NETSPLIT=on"`. All available configuration
parameters can be found on https://vernemq.com/docs/configuration/.

#### Logging

VerneMQ store crash and error log files in `/var/log/vernemq/`, and, by default,
doesn't write console log to the disk to avoid filling the container disk space.
However this behaviour can be changed by setting the environment variable `DOCKER_VERNEMQ_LOG__CONSOLE` to `both`
which tells VerneMQ to send logs to stdout and `/var/log/vernemq/console.log`.
For more information please see VerneMQ logging documentation: https://docs.vernemq.com/configuring-vernemq/logging

#### Remarks

Some of our configuration variables contain dots `.`. For example if you want to
adjust the log level of VerneMQ you'd use `-e
"DOCKER_VERNEMQ_LOG.CONSOLE.LEVEL=debug"`. However, some container platforms
such as Kubernetes don't support dots and other special characters in
environment variables. If you are on such a platform you could substitute the
dots with two underscores `__`. The example above would look like `-e
"DOCKER_VERNEMQ_LOG__CONSOLE__LEVEL=debug"`.

There some exceptions on configuration names contains dots. You can see follow examples:

format in vernemq.conf | format in environment variable name
---------------------- | ------------------------------------
`vmq_webhooks.pool_timeout = 60000` | `DOCKER_VERNEMQ_VMQ_WEBHOOKS__POOL_timeout=6000`
`vmq_webhooks.pool_timeout = 60000` | `DOCKER_VERNEMQ_VMQ_WEBHOOKS.pool_timeout=60000`

## Autojoining a VerneMQ cluster

This allows a newly started container to automatically join a VerneMQ cluster. Assuming you started your first node like the example above you could autojoin the cluster \(which currently consists of a single container 'vernemq1'\) like the following:

```text
docker run -e "DOCKER_VERNEMQ_DISCOVERY_NODE=<IP-OF-VERNEMQ1>" --name vernemq2 -d gojektech/vernemq
```

\(Note, you can find the IP of a docker container using `docker inspect <CONTAINER_NAME> | grep \"IPAddress\"`\).

## Checking cluster status

To check if the above containers have successfully clustered you can issue the `vmq-admin` command:

```text
docker exec vernemq1 vmq-admin cluster show
+--------------------+-------+
|        Node        |Running|
+--------------------+-------+
|VerneMQ@172.17.0.151| true  |
|VerneMQ@172.17.0.152| true  |
+--------------------+-------+
```

