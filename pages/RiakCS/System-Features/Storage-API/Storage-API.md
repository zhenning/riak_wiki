<div class="info"><div class="title">Riak CS Only</div>This documentation applies only to Riak Cloud Storage, a commercial extension to <a href="http://wiki.basho.com/Riak.html">Riak</a>. To talk to us about using Riak CS, <a href="http://info.basho.com/Wiki_Contact_RiakCS.html" target="_blank">let us know</a>.</div>

# Storage API
The storage API is compatabile with the Amazon S3 REST API which means that any of the operations listed can be executed using any of the commonly available S3 libraries or tools.

## Service-level Operations

* [[GET Service|GET-Service.html]] - Returns a list of all buckets owned by the user who sent the request

## Bucket-level Operations

* [[GET Bucket|GET-Bucket]] - Returns a list of the objects within a bucket
* [[GET Bucket ACL|GET-Bucket-ACL]] - Returns the ACL associated with a bucket
* [[PUT Bucket|PUT-Bucket]] - Creates a new bucket
* [[PUT Bucket ACL|PUT-Bucket-ACL]] - Sets the ACL permissions for a bucket
* [[DELETE Bucket|DELETE-Bucket]] - Deletes a bucket

## Object-level Operations

* [[GET Object|/GET-Object]]- Retrieves an object
* [[GET Object ACL|GET-Object-ACL]] - Returns the ACLs associated with an object
* [[PUT Object|PUT-Object]] - Stores an object to a bucket
* [[PUT Object ACL|PUT-Object-ACL] - Sets the ACLs associated with an object
* [[HEAD Object|HEAD-Object]] - Retrieves object metadata (not the full content of the object)
* [[DELETE Object|DELETE-Object]]- Deletes an object

## Common Headers

* [[Common Request Headers|Common-Request-Headers]]
* [[Common Response Headers|Common-Response-Headers.html]]
