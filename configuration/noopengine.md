---
description: Configure how VerneMQ implements message persistence
---

# No op engine

```text
generic_message_store_engine=vmq_storage_engine_no_op
```
This implementation of message store does not persist messages to the disk.

## Tradeoffs

When a node in a VerneMQ cluster restarts, it tries to load from the disk, all it's messages before it was shutdown. We have seen that this process is very CPU intensive and time-taking, and it may so happen that the node may not be able to join the cluster.

This config allows the node to restart and join the cluster because it does not have to build it's messages from the disk.
The drawback is that all the messages that were not synced with other nodes before the node got killed will be lost.
