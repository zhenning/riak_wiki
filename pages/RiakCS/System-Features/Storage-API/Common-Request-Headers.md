<div class="info"><div class="title">Riak CS Only</div>This documentation applies only to Riak Cloud Storage, a commercial extension to <a href="http://wiki.basho.com/Riak.html">Riak</a>. To talk to us about using Riak CS, <a href="http://info.basho.com/Wiki_Contact_RiakCS.html" target="_blank">let us know</a>.</div>

# Common Request Headers
These are the headers that are common to all Riak CS REST requests.

**Authorization** - Information required to request authentication. This header is not required for anonymous requests.

**Content-Length** - Length of message without headers according to RFC 2616. This header is required for PUTs and operations that load XML.

**Content-Type** - The content type of the resource.

**Content-MD5** - The base64-encoded 128-bit MD5 digest of the message without the headers according to RFC 1864. Although this header is optional, the Content-MD5 header can be used to confirm that the data is the same as what was originally sent.

**Date** - The current data and time according to the requester, for example, Fri, 01 Jun 2012 12:00:00 GMT. With the `Authorization` header, you must specify either `x-amz-date` or `Date` header.

**Expect** - When you use `100-continue` in your application, it doesn't send the request body until it receives an acknowledgement. That way, the body of the message isn't sent if the message is rejected based on the headers.

* *Valid Values*: 100-continue

**Host** - For path-style requests, the value is something like `data.basho.com`. For virtual-style requests, the value is something like `bucketname.data.basho.com`.

This header is optional for HTTP/1.0 requests, but is required for HTTP 1.1.

**x-amz-date** - The current data and time according to the requester, for example, Fri, 01 Jun 2012 12:00:00 GMT. With the `Authorization` header, you must specify either `x-amz-date` or `Date` header. If you specify both, the value for this header takes precedence.
