---
description: New message passing model for VerneMQ
---

This new model uses redis as its queue for passing the messages between the broker nodes. Each broker node has a set of workers which constantly polls redis to fetch messages from its own queue, called main queue.



## Redis shards connect options

Specify the list of comma separated connect options list [[{host, "localhost"}, {port, 1234}]] of redis shards used for message passing.

```text
msg_queue_redis_shards_connect_options = [[{host,"127.0.0.1"},{port,6379},{database,1}]]
```

## Main queue workers per redis shard

Specify the number of worker processes per redis shard that will poll their main queues in message passing redis shard for new messages. 

```text
main_queue_workers_per_redis_shard = 1
```

## Main queue worker sleep interval

Specify the interval (in milliseconds) a worker process waits before making another poll request.

```text
redis_queue_sleep_interval = 0
```
