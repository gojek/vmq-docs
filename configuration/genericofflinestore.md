---
description: Configure offline message store.
---

In the vanilla VerneMQ(1.11.0) on netsplit, the broker state had to be cleared before restoration which led to loss of offline messages stored in leveldb using `vmq_generic_message_store` plugin. Since this kind of persistence is actually not useful so no-op engine was introduced which bypass this persistence step and helped in quick recovery of broker as there was no message state to be loaded in memory.

Using offline message store, only offline messages are persisted for the offline clients. In order to use offline message store, message store plugin needs to be explicitly configured.
```text
message_store_plugin = vmq_generic_offline_msg_store
```

Currently two offline store engines are available, postgres and redis sentinel.

## Offline store engine

Specify the engine to be used for storage.

```text
offline_message_store_engine = vmq_offline_storage_engine_redis
```

## Offline store host

For redis sentinel engine, all the hosts(master and slaves) can be specified in a list format, `["host1", "host2"]`

```text
offline_message_store_opts.host = localhost
```


## Offline store port

Specify the port on which the offline store is listening.

```text
offline_message_store_opts.port = 6379
```

## Offline store database

Specify the name of the database to be used as offline store.
```text
offline_message_store_opts.database = 2
```

## Offline store connect timeout

Specify the connect timeout (in milliseconds).
```text
offline_message_store_opts.connect_timeout = 4000
```

## Offline store query timeout

Specify the query timeout (in milliseconds).
```text
offline_message_store_opts.query_timeout = 2000
```
