# Memory

<div id="toc"></div>

## Overview

The Memory Backend storage engine uses in-memory tables to store all data.
This data is never persisted to disk or any other storage.  The Memory storage
engine is best used for testing Riak clusters or for small amounts of transient
state in production systems.

<div class="note"><div class="title">Memory replaces the Cache backend</div>The
Memory backend is designed to offer you the same functionality as the now
obsolete Cache backend found in pre-1.0 versions of Riak.  The configuration
options for Memory match those of Cache and can be used to make the memory
backend behave similarly to the cache backend.</div>

## Installing the Memory Backend

Riak ships with the Memory Backend included within the distribution so there is
no separate installation required.

## Configuring the Memory Backend

Modify the default behavior by adding these settings into the `memory_backend`
section in your [app.config](Configuration Files).

### Max Memory

  The amount of memory in megabytes to limit the backend to.

```erlang
{memory_backend, [
	    ...,
            {max_memory, 4096}, %% 4GB in megabytes
	    ...
]}
```

### TTL

  The time in seconds before an object expires.

```erlang
{memory_backend, [
	    ...,
            {ttl, 86400}, %% 1 Day in seconds
	    ...
]}
```

## Memory Backend Implementation Details

This backend uses the Erlang `ets` tables internally to manage data.
