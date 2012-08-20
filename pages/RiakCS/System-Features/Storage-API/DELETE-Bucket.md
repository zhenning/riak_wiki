<div class="info"><div class="title">Riak CS Only</div>This documentation applies only to Riak Cloud Storage, a commercial extension to <a href="http://wiki.basho.com/Riak.html">Riak</a>. To talk to us about using Riak CS, <a href="http://info.basho.com/Wiki_Contact_RiakCS.html" target="_blank">let us know</a>.</div>

# DELETE Bucket
The `DELETE Bucket` operation deletes the bucket specified in the URI.

<div class="note"><div clas="title">Note</div>All objects in the bucket must be deleted before you can delete the bucket.</div>

## Requests

### Request Syntax

```
DELETE / HTTP/1.1
Host: bucketname.data.basho.com
Date: date
Authorization: signature_value
```

## Responses

DELETE Bucket uses only common response headers and doesn't return any response elements.

## Examples

### Sample Request

The DELETE Bucket operation deletes the bucket name "projects".

```
DELETE / HTTP/1.1
Host: projects.data.basho.com
Date: Wed, 06 Jun 2012 20:47:15 +0000
Authorization: AWS QMUG3D7KP5OQZRDSQWB6:4Pb+A0YT4FhZYeqMdDhYls9f9AM=
```

### Sample Response

```
HTTP/1.1 204 No Content
Date: Wed, 06 Jun 2012 20:47:15 +0000
Connection: close
Server: MochiWeb/1.1 WebMachine/1.9.0 (someone had painted it blue)
```
