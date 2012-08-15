The storage API is compatabile with the Amazon S3 REST API which means that any of the operations listed can be executed using any of the commonly available S3 libraries or tools.

##Service-level operations

* [[GET Service|https://help.basho.com/entries/21653026-get-service]] - Returns a list of all buckets owned by the user who sent the request

##Bucket-level operations

* [[GET Bucket|https://help.basho.com/entries/21635452-get-bucket]] - Returns a list of the objects within a bucket
* [[GET Bucket ACL|https://help.basho.com/entries/21635472-get-bucket-acl]] - Returns the ACL associated with a bucket
* [[PUT Bucket|https://help.basho.com/entries/21643203-put-bucket]] - Creates a new bucket
* [[PUT Bucket ACL|https://help.basho.com/entries/21649877-put-bucket-acl]] - Sets the ACL permissions for a bucket
* [[DELETE Bucket|https://help.basho.com/entries/21658138-delete-bucketDELETE Bucket]] - Deletes a bucket

##Object-level operations

* [[GET Object|https://help.basho.com/entries/21658148-get-object]]- Retrieves an object
* [[GET Object ACL|https://help.basho.com/entries/21626741-get-object-acl]] - Returns the ACLs associated with an object
* [[PUT Object|https://help.basho.com/entries/21653086-put-object]] - Stores an object to a bucket
* [[PUT Object ACL]|https://help.basho.com/entries/21658198-put-object-acl] - Sets the ACLs associated with an object
* [[HEAD Object|https://help.basho.com/entries/21653106-head-object]] - Retrieves object metadata (not the full content of the object)
* [[DELETE Object|https://help.basho.com/entries/21641032-delete-object]]- Deletes an object

##Common headers

* [[Common Request Headers|https://help.basho.com/entries/21635572-common-request-headers]]
* [[Common Response Headers|https://help.basho.com/entries/21658666-common-response-headers]]

