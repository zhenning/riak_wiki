# Product Overview
This section provides an overview of the features available in the Riak
CS API.

Riak and Riak CS can both be installed on the same machine. Riak CS can
also be installed on a different machine when the Riak machine is
accessible over a network. Riak CS communicates with Riak via the Riak
Erlang client, which uses Protocol Buffers. When Riak CS runs on
multiple machines, you can set up load balancing between the nodes. Riak
CS can store files up to 5GB in size.

## Storage API (Amazon S3‐compatible)
The Riak CS API is a subset of the Amazon S3 API. As of version 1, the Riak CS API supports the following operations.

### Service‐level operations

* GET Service ‐ Returns a list of all buckets owned by the user who
    sent the request

### Bucket‐level operations

* GET Bucket ‐ Returns a list of the objects within a bucket
* GET Bucket ACL ‐ Returns the ACL associated with a bucket
* PUT Bucket ‐ Creates a new bucket
* PUT Bucket ACL ‐ Sets the ACL permissions for a bucket
* DELETE Bucket ‐ Deletes a bucket

### Object‐level operations

* GET Object ‐ Retrieves an object
* GET Object ACL ‐ Returns the ACLs associated with an object
* PUT Object ‐ Stores an object to a bucket
* PUT Object ACL ‐ Sets the ACLs associated with an object
* HEAD Object ‐ Retrieves object metadata (not the full object content )
* DELETE Object ‐ Deletes an object

### Provisioning API

* CREATE User ‐ Creates a user account

### Reporting API

* GET Usage ‐ Special resource to query storage and access usage data on a per user basis.

Access statistics are available in JSON or XML formats for individual users via the HTTP API. Use the **usage** resource to query access statistics:

    /usage/$USER_KEY_ID

For example, to request the usage statistics for a user, access the usage resource as shown in this example:

    http://<storage‐address>/usage/<user‐key‐id>

Substitute <storage‐address> for the IP address or domain name of your Riak CS storage host, and <user‐key‐id> with the key for a specific user.
