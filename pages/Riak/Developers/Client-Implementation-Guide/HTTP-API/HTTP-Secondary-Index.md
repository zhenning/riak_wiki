Query a secondary index on a particular bucket for matching keys.

<div class="note"><p>As of Riak 1.0, only the LevelDB backend supports secondary
indexes.</p></div>

## Request

```bash
GET /buckets/bucket/index/[index]/[key]            # Equality query
GET /buckets/bucket/index/[index]/[start]/[end]    # Range query
```

* `index` - the name of the index, including its type (either `bin` or `int`
* `key` - the value to match in an equality query
* `start` - the lower bound in a range query
* `end` - the upper bound in a range query

## Response

Normal status codes:

* `200 OK`

Typical error codes:

* `400 Bad Request` - Invalid query parameters

Important headers:

* `Content-Type` - `application/json`

The response will be a JSON object containing an array named "keys" that
containing all of the matching keys in the index.

## Example: Equality

```bash
$ curl -v http://127.0.0.1:8098/buckets/test/index/testers_bin/joe
* About to connect() to 127.0.0.1 port 8098 (#0)
*   Trying 127.0.0.1... connected
* Connected to 127.0.0.1 (127.0.0.1) port 8098 (#0)
> GET /buckets/test/index/testers_bin/joe HTTP/1.1
> User-Agent: curl/7.21.4 (universal-apple-darwin11.0) libcurl/7.21.4 OpenSSL/0.9.8r zlib/1.2.5
> Host: 127.0.0.1:8098
> Accept: */*
> 
< HTTP/1.1 200 OK
< Vary: Accept-Encoding
< Server: MochiWeb/1.1 WebMachine/1.9.0 (participate in the frantic)
< Date: Mon, 26 Sep 2011 19:53:27 GMT
< Content-Type: application/json
< Content-Length: 19
< 
* Connection #0 to host 127.0.0.1 left intact
* Closing connection #0
{"keys":["test1","test2","test3"]}
```
## Example: Range

```bash
$ curl -v http://127.0.0.1:8098/buckets/test/index/testers_bin/jane/joe
* About to connect() to 127.0.0.1 port 8098 (#0)
*   Trying 127.0.0.1... connected
* Connected to 127.0.0.1 (127.0.0.1) port 8098 (#0)
> GET /buckets/test/index/testers_bin/jane/joe HTTP/1.1
> User-Agent: curl/7.21.4 (universal-apple-darwin11.0) libcurl/7.21.4 OpenSSL/0.9.8r zlib/1.2.5
> Host: 127.0.0.1:8098
> Accept: */*
> 
< HTTP/1.1 200 OK
< Vary: Accept-Encoding
< Server: MochiWeb/1.1 WebMachine/1.9.0 (participate in the frantic)
< Date: Mon, 26 Sep 2011 19:53:27 GMT
< Content-Type: application/json
< Content-Length: 19
< 
* Connection #0 to host 127.0.0.1 left intact
* Closing connection #0
{"keys":["test1","test2","test3","test4","test5"]}
```