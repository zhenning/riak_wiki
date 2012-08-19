<div class="info"><div class="title">Riak CS Only</div>This documentation applies only to Riak Cloud Storage, a commercial extension to <a href="http://wiki.basho.com/Riak.html">Riak</a>. To talk to us about using Riak CS, <a href="http://info.basho.com/Wiki_Contact_RiakCS.html" target="_blank">let us know</a>.</div>

# Configuration Overview
In a Riak CS storage system, three components work in conjunction, so you must configure each component to work with the others:

* Riak - The database system that acts as the backend storage.
* Riak CS - The cloud storage layer over Riak which exposes the storage and  billing APIs, storing files and metadata in Riak, and streaming them back to  users.
* Stanchion - Manages requests involving globally unique system entities, such as  buckets and users sent to a Riak instance, for example, to create users or to create or delete buckets.

In addition, you must also configure the S3 client you use to communicate with your Riak CS system.

You should plan on having one Riak node for every Riak CS node in your system. Riak and Riak CS nodes can be run on separate physical machines, but in many cases it is preferable to run one Riak and one Riak CS node on the same physical machine. Assuming the single physical machine has sufficient capacity to meet the needs of both a Riak and a Riak CS node you will typically see better performance due to reduced network latency.

If your system consists of several nodes, configuration primarily represents setting up the communication between components. Other settings, such as where log files are stored, are set to default values and need to be changed only if you want to use non-default values.

## Configuration of System Components

* [[Configuring Riak|Configuring-Riak.html]]
* [[Configuring Riak CS|Configuring-Riak-CS.html]]
* [[Configuring Stanchion|Configuring-Stanchion.html]]
* [[Configuring an S3 client|Configuring-an-S3-Client.html]]