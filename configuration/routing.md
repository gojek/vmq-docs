---
description: Configure Routing store.
---

One of the problems that we tried to address was that of routing state not syncing when one of the nodes did was restarted. The subscription state used to build up on other nodes and the restarted node was unable to catch up to the other nodes. We finally achieved this using an external store(redis) for maintaining subscription state. 

You can use the redis based external subscription store using the following configs as mentioned below:


## Default Reg trie

You have to configure the default_reg_view to vmq_reg_redis_trie to start using redis store as the subscription store.

```text
default_reg_view = vmq_reg_redis_trie
```

## Lua directory

By default, we use a set of lua scripts that are loaded into `/etc/lua` inside the `_build/default/rel/vernemq` when a release is created using `make rel`. This value can be overridden if you wish to use a different set of scripts. 

```text
redis_lua_dir = ./etc/lua
```


## Redis database

Redis databases are numbered from 0 to 15 and, by default, you connect to database 0 when you connect to your Redis instance. However, you can change the database you’re using with the select command after you connect. This config specifies which redis_database to use. By default, you always connect to database 0.

```text
redis_database = 0
```

## Redis sentinel endpoints

The address of redis sentinel endpoints.
```text
redis_sentinel_endpoints = [{"vernemq-redis-1", 26379}, {"vernemq-redis-2", 26379}, {"vernemq-redis-3", 26379}]
```

