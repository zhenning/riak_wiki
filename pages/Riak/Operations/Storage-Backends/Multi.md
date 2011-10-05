# Multiple Backends

<div id="toc"></div>

## Overview

Riak allows you to run multiple backends within a single Riak instance.  This
is very useful for two very different use cases; 1. You may want to use
different backends for different buckets or 2. You may need to use the same
storage engine in different ways for different buckets The Multi backend allows
you to configure more than one backend at the same time on a single cluster.

## Installing Multi-Backend Support

Riak ships with Multi backend support included within the distribution so there
is no separate installation required.

## Configuring Multiple Backends

Modify the default behavior by adding these settings in your
[app.config](Configuration Files).  The `multi_backend` configuration must be
within the `riak_kv` section of the `app.config`.

```erlang
%% Riak KV config
{riak_kv, [
          ...
          %% Use the Multi Backend
          {storage_backend, riak_kv_multi_backend},
          ...
]}
```

Then later anywhere in the `app.config` (but you'll likely want this in the
section with other backend-related information) add a section to configure the
multiple backends.

```erlang
%% Use bitcask by default
{multi_backend_default, <<"bitcask">>},
{multi_backend, [
    %% Here's where you set the individual multiplexed backends
    {<<"bitcask_mult">>,  riak_kv_bitcask_backend, [
                     %% bitcask configuration
                     {config1, ConfigValue1},
                     {config2, ConfigValue2}
    ]},
    {<<"eleveldb_mult">>, riak_kv_eleveldb_backend, [
                     %% eleveldb configuration
                     {config1, ConfigValue1},
                     {config2, ConfigValue2}
    ]},
    {<<"second_eleveldb_mult">>,  riak_kv_eleveldb_backend, [
                     %% eleveldb with a different configuration
                     {config1, ConfigValue1},
                     {config2, ConfigValue2}
    ]},
    {<<"memory_mult">>,   riak_kv_memory_backend, [
                     %% memory configuration
                     {config1, ConfigValue1},
                     {config2, ConfigValue2}
    ]}
]},
```

Once configured start the Riak cluster.  Riak will use the
`multi_backend_default` for all new bucket storage unless you configure a
bucket to use one of the alternate storage engines.  This can be done using
either the Erlang console or the HTTP interface, both methods simply change the
bucket properties.  Here are two examples:

  * Using the Erlang console
    You can connect to a live node using the Erlang console and directly set
    the bucket properties.
```erlang
$ riak attach
...
1> riak_core_bucket:set_bucket(<<"MY_BUCKET">>, [{backend, second_bitcask_mult}])
```


  * Using the HTTP REST API
    You can also connect to Riak using the HTTP API and change the bucket
    properties.
```bash
$ curl -X PUT -H "Content-Type: application/json" -d '{"props":{"backend":"memory_mult"}}' http://riaknode:8098/riak/transient_example_bucketname
```

Once you've changed a bucket's storage engine on a node you'll need to restart
the node for that change to take effect.

<div class="note"><div class="title">Secondary Indicies (2i) with the Multi storage backend</div> You cannot use the multi backend with buckets configured for secondary indicies in the 1.0.0 release of Riak.  All buckets involved with 2i must use the eLevelDB directly.</div>

## FAQ

  * [How can I run Bitcask and Innostore backends on the same cluster?](https://help.basho.com/entries/20186031-how-can-i-run-bitcask-and-innostore-on-the-same-cluster)
