# Introduction
Most interactions you'll have with Riak will be setting or retrieving objects. You can read more about basic operations through the Riak HTTP API, Erlang interface, protobufs API, or many client libraries in our [[Developers section|Developers]]. This section focuses on strategies for searching and aggregating data in Riak: Search, secondary indexes and MapReduce. 

Below is an overview of each functionality. Drill deeper to see features, when to use and when not to use, how it works, examples and documentation. Not sure where to start? Check out our [[comparison page|MapReduce-Search-2i-Comparison]] for a high-level guide of what to use when. 

## Search
[[Riak Search|Riak-Search]] is a distributed, full-text search engine that ships with Riak. It indexes Riak KV objects as they're written using a precommit hook. Based on the object’s mime type and the schema you’ve set for its bucket, the hook will automatically extract and analyze your data and build indexes from it. Search has a robust, easy-to-use query language with scoring and ranking for most relevant results. 


## Secondary Indexes
[[Secondary Indexing|Secondary-Indexes]] (2i) in Riak gives developers the ability to tag an object stored in Riak with one or more values (key/value metadata), which can then be queried to return a list of matching keys. Indexes can be queried by exact match or range. 2i also has anti entropy and does not require a schema.

## MapReduce
[[MapReduce]] is the main method for non-primary-key-based querying in Riak. MapReduce is a programming model which spreads the processing of a query across many systems to take advantage of the parallel processing power of distributed systems. Riak MapReduce is intended for batch processing and has support for Javascript and Erlang. 

