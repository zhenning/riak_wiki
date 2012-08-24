<div class="info"><div class="title">Riak CS Only</div>This documentation applies only to Riak Cloud Storage, a commercial extension to <a href="http://wiki.basho.com/Riak.html">Riak</a>. To talk to us about using Riak CS, <a href="http://info.basho.com/Wiki_Contact_RiakCS.html" target="_blank">let us know</a>.</div>

# Storage API
The storage API is compatabile with the Amazon S3 REST API which means that any of the operations listed can be executed using any of the commonly available S3 libraries or tools.

## Service-level Operations

* [[GET Service|RiakCS GET Service]] - Returns a list of all buckets owned by the user who sent the request

## Bucket-level Operations

* [[GET Bucket|RiakCS GET Bucket]] - Returns a list of the objects within a bucket
* [[GET Bucket ACL|RiakCS GET Bucket ACL]] - Returns the ACL associated with a bucket
* [[PUT Bucket|RiakCS PUT Bucket]] - Creates a new bucket
* [[PUT Bucket ACL|RiakCS PUT Bucket ACL]] - Sets the ACL permissions for a bucket
* [[DELETE Bucket|RiakCS DELETE Bucket]] - Deletes a bucket

## Object-level Operations

* [[GET Object|RiakCS GET Object]]- Retrieves an object
* [[GET Object ACL|RiakCS GET Object ACL]] - Returns the ACLs associated with an object
* [[PUT Object|RiakCS PUT Object]] - Stores an object to a bucket
* [[PUT Object ACL|RiakCS PUT Object ACL]] - Sets the ACLs associated with an object
* [[HEAD Object|RiakCS HEAD Object]] - Retrieves object metadata (not the full content of the object)
* [[DELETE Object|RiakCS DELETE Object]]- Deletes an object

## Common Headers

* [[Common RiakCS Request Headers]]
* [[Common RiakCS Response Headers]]
