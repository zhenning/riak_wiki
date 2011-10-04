# Memory

<div id="toc"></div>

## Overview

The Memory Backend storage engine uses in-memory tables to store all data.
This data is never persisted to disk or any other storage.  The Memory storage
engine is best used for testing Riak clusters and is never recommended for use
in production systems.

## Installing the Memory Backend

Riak ships with the Memory Backend included within the distribution there is no
separate installation required.

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
