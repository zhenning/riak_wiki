# Configuration Overview
In a Riak CS storage system, three components work in conjunction, so you must configure each component to work with the others:

* Riak - The database system that acts as the backend storage.
* Riak CS - The cloud storage layer over Riak which exposes the storage and  billing APIs, storing files and metadata in Riak, and streaming them back to  users.
* Stanchion - Manages requests involving globally unique system entities, such as  buckets and users sent to a Riak instance, for example, to create users or to create or delete buckets.

In addition, you must also configure the S3 client you use to communicate with your Riak CS system.

If your system consists of several nodes, configuration primarily represents setting up the communication between components. Other settings, such as where log files are stored, are set to default values and need to be changed only if you want to use non-default values.

## Configuration of System Components

* [[Configuring Riak|https://help.basho.com/entries/21444026-configuring-riak-for-a-riak-cs-system]]
* [[Configuring Riak CS|https://help.basho.com/entries/21444166-configuring-riak-cs]]
* [[Configuring Stanchion|https://help.basho.com/entries/21426482-configuring-stanchion]]
* [[Configuring an S3 client|https://help.basho.com/entries/21359386-configuring-an-s3-client]]