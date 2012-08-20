<div class="info"><div class="title">Riak CS Only</div>This documentation applies only to Riak Cloud Storage, a commercial extension to <a href="http://wiki.basho.com/Riak.html">Riak</a>. To talk to us about using Riak CS, <a href="http://info.basho.com/Wiki_Contact_RiakCS.html" target="_blank">let us know</a>.</div>

# DELETE Object
The `DELETE Object` operation removes an object, if one exists.

## Requests

### Request Syntax

```
DELETE /ObjectName HTTP/1.1
Host: bucketname.data.basho.com
Date: date
Content-Length: length
Authorization: signature_value
```

## Examples

### Sample Request

The DELETE Object operation deletes the object, `projects-schedule.jpg`.

```
DELETE /projects-schedule.jpg HTTP/1.1
Host: bucketname.data.basho.com
Date: Wed, 06 Jun 2012 20:47:15 GMT
Authorization: AWS QMUG3D7KP5OQZRDSQWB6:4Pb+A0YT4FhZYeqMdDhYls9f9AM=
```

### Sample Response

```
HTTP/1.1 204 No Content
Date: Wed, 06 Jun 2012 20:47:15 GMT
Connection: close
Server: MochiWeb/1.1 WebMachine/1.9.0 (someone had painted it blue)
```
