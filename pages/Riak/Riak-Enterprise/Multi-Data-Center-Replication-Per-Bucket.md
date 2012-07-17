<div class="info"><div class="title">Riak Enterprise Only</div>This documentation only applies to the Enterprise version of Riak.</div>

<div class="note"><div class="title">This feature is available in Riak Enterprise 1.1+</div></div>

To enable or disable replication per bucket, you can use the `repl` bucket property.

### Example of Disabling

    curl -v -X PUT -H "Content-Type: application/json" \
    -d '{"props":{"repl":false}}' \
    http://127.0.0.1:8091/riak/my_bucket

### Example of Enabling

    curl -v -X PUT -H "Content-Type: application/json" \
    -d '{"props":{"repl":true}}' \
    http://127.0.0.1:8091/riak/my_bucket
