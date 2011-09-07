A key feature of Riak KV is the pluggable storage backends. These allow the ability to choose a low-level storage engine that suits specific operational needs. For example, if one needs maximum throughput coupled with data persistence and has a bounded keyspace, Bitcask is a good choice. If one want to store a large number of keys then LevelDB is recommended.

Currently, five backends are supported:

- [[Bitcask]]
- LevelDB
- [[Innostore]]
- Cache
- Multi