# Searching/Accessing Data

Riak is a Key/Value NoSQL data store that is highly optimized for the storage and retrieval of enormous quantities of unstructured data objects (documents, images, session data, etc.). This page gives a high-level overview of the various methods for accessing data in the Riak NoSQL data store.

## Object/Key Operations
Data in Riak is organized into buckets, keys, and objects (values). A bucket is a logical separation of entities with configurable replication, pre/post-commit hooks, and conflict resolution strategies. Objects are a unit of data storage in Riak and are comprised of a bucket, key, value, vector clock, and a set of metadata. A unique key identifies each object and has the following form: hash(bucket_name + key_name). The same key name can exist in multiple buckets.

## Search
[[Riak Search|Riak-Search]] is a distributed, full-text search engine that is built on Riak Core and included as part of Riak open source. Search provides the most advanced query capability next to MapReduce, but is far more concise, easier to use, and in most cases puts far less load on the cluster.

## Secondary Indexes
[[Secondary Indexing|Secondary-Indexes]]  (2i) in Riak gives developers the ability, at write time, to tag an object stored in Riak with one or more values (key/value metadata), which can then be queried.
Since the KV data is completely opaque to 2i, the user must tell 2i exactly what attribute to index on and what its index value should be. This is different from Search, which parses the data and builds indexes based on a schema.

## MapReduce
[[MapReduce]] is the primary method for non-primary-key-based querying in Riak.  MapReduce is a programming model which spreads the processing of a query across many systems to take advantage of the parallel processing power of distributed systems. Developers can use MapReduce for things like filtering documents by tags, counting words in documents, and extracting links to related data.


