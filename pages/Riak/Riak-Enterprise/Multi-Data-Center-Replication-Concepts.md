<div class="info"><div class="title">Riak Enterprise Only</div>This documentation only applies to the Enterprise version of Riak.</div>

## Overview
This document is a guide to configuring and managing replication between Riak clusters.

## How Replication Works
Riak replication copies all data from a primary cluster to a secondary. Currently it is unidirectional, although you can configure a pair of connections to sync bidirectionally between two clusters. All data is synchronized on initial connection (configurable), followed by streamed updates to the secondary cluster and periodic full syncs.

## Concepts
### Listeners
Replication 'listener' nodes are Riak cluster nodes that will listen on an external IP address and port for the purposes of handling Replication requests. Any node in a Riak cluster can participate as a listener - adding more nodes will increase the fault-tolerance of the replication process in the presence of individual node failures. Listeners are started on the primary cluster. These are also called 'servers'.

### Sites
Replication 'site' nodes are Riak cluster nodes that connect to listeners and initiate Replication requests. Site nodes must be given a listener node to connect to when started. Sites are started on the *secondary cluster*. These are also called 'clients'.

### Leadership
Riak replication uses a leadership-election protocol to determine which node in the cluster will participate in replication. Only one node in each cluster will be responsible for serving as the client or server to any other clusters.

If a client/site connects to a listener that is not the leader, it will be redirected to the listener node that is currently the leader.

### Full-sync
On initial connection, the primary cluster (listener/server) will initiate a full synchronization with the secondary cluster (site/client), which computes hashes of all keys stored in each partition of the primary cluster and sends them to the secondary cluster. The secondary cluster then calculates its own hashes and requests updates for keys that are missing or stale.

New writes on the primary cluster will be streamed to the secondary cluster between synchronization of individual partitions.

Full-syncs are also performed on a periodic basis, configurable in the riak_repl section of app.config, under the fullsync_interval parameter. See the [[Multi Data Center Replication Operations]] document for more details.
