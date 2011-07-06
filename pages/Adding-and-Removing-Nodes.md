This page describes the process of adding and removing nodes to and from a Riak
cluster. We'll look at how you join nodes together into a cluster, and what
happens when you add or remove nodes.

<div id="toc"/></div>

## Preconditions

For most operations you need to access configuration files, mainly app.config
and vm.args, which are located, depending on whether you installed from source
or using binary package, in `etc/` in your release directory (built from
source), or in `/etc/riak/` when installed using a binary package,
`/opt/riak/etc` when you installed the Riak binary packages on Solaris.

## Creating the First Node

After installing Riak on a system, using either the binary packages or from
sources, there's some initial configuration steps you need to take that depend
on your networking infrastructure and security measures.

Assuming your node is not running (if it is, stop it using `riak stop` or
`/etc/init.d/riak stop`)

## Primary Steps

## Change the Node Name

The node name is an important setting for the Erlang VM, especially when you
want to build a cluster of nodes, as the node name identifies both the Erlang
application and the host name on the network. All nodes in the Riak cluster need
these node names to communicate and coordinate with each other.

The node name is configured in `vm.args`. Change the following line, which
defaults to 127.0.0.1 (a.k.a. localhost):

```bash
-name riak@127.0.0.1
```

Change it to something that corrosponds to either the IP address or a resolveable
host name for this particular node, like so:

```bash
-name riak@192.168.1.10
```

## Change the HTTP and Protocol Buffers binding address

By default, Riak's HTTP service is bound to the local interface, i.e. 127.0.0.1,
therefore unable to serve requests from the outside network. The relevant
setting is configured in `app.config`, find the line in the riak_core section
that reads:

```bash
{http, [ {"127.0.0.1", 8098 } ]},
```

Either change it to use an IP address that corresponds to one of the server's
network interfaces, or 0.0.0.0 to allow access from all interfaces and networks,
e.g.:

```bash
{http, [ {"0.0.0.0", 8098 } ]},
```

Same configuration should be changed for the Protocol Buffers interface if you
intend on using it, do the same as above for the line in the `riak\_kv` section
that reads:

```bash
{pb_ip,   "127.0.0.1" },
```

## Start the Node

Just like the configuration steps, this step will be the same for every node in
your cluster. Before a node can join an existing cluster it needs to be started.


## What Happens When You Start a Node?

When you start a node it looks for a cluster description, the ring file, in its
data directory. If none exists it creates a new ring description based on the
initially configured `ring\_creation\_size`. The node is then ready to serve
requests.

## Add a Node to an Existing Cluster

When the node is running, it can be added to an existing cluster. Pick a random
node in your existing cluster and send a join request from the new node. The
example shown below uses the IP 192.168.2.2 as the so called seed node, the node
that seeds the existing cluster data to the new node.

```bash
riak-admin join riak@192.168.2.2
```

## What Happens When a New Node Joins a Cluster?

The process of joining a cluster has several steps. When you send the join
request from a new node, it will ask send the seed node to send its cluster
state. When the new node receives the cluster state, it discards its own,
overwriting it completely with the state it just received and starts claiming
partitions until the number of partitions in the cluster reaches as even a
distribution as possible, taking into account the N value to guarantee a good
physical distribution of partitions in the cluster.

While claiming partitions the new node keeps updating the cluster state until an
even distribution is reached. Claiming a partition means that the new node is
now a primary replica for the particular partition.

When the node has recalculated a new cluster state, it gossips the state to a
random node in the cluster, thus making its own claims known to the other nodes.

After it ensured that all the vnodes, i.e. partitions,  it's responsible are
running, partition handoff starts, transferring the data from existing nodes to
the new one. The handoff is initiated by the existing nodes, as the vnodes
running on them realize that they're not a primary replica for a particular
partition anymore, therefore transferring all their data to the new primary
replica on the node that just joined.

This process happens asynchronously, as the gossip is updated across the cluster
over the next couple of minutes. Remember that after claiming its partitions the
new node only gossips the new cluster state to a random node in the cluster,
which then in turn gossips the state to the other nodes, so it can take up to a
minute until the handoff starts.

During the handoff, the new cluster state is already gossiped throughout the
cluster, so there currently are periods where handoff is still active, but the
new node is already expected to serve requests. Basho is working on improving
this situation, but in general the application interacting with Riak is expected
to deal with situations where not all replicas may have the data yet. See our
page on [[Eventual Consistency]] for more details on these scenarios.

Ryan Zezeski wrote a [great
introduction](https://github.com/rzezeski/try-try-try/tree/master/2011/riak-core-the-vnode)
of what happens during a vnode's lifecycle, including an overview of the
different states of handoff.

## Removing a Node From a Cluster

A node can be removed from the cluster in two ways. One assumes that a node is
decommissioned because its added capacity is not needed anymore or because it's
explicitly replaced with a new one. The second is relevant for failure
scenarios, where a node has crashed and is irrecoverable, so it must be removed
from the cluster from another node.

The command to remove a node is `riak-admin leave`. This command must be
executed on the node that's supposed to be removed from the cluster.

The other command is `riak-admin remove <node>`, where `<node>` is an Erlang
node name as specified in the node's vm.args file, e.g. `riak-admin remove
riak@192.168.2.1`. This command can be run from any other node in the cluster.

Under the hood, both commands do the same thing, running `riak-admin leave` just
selects the current node for you automatically.

## What Happens When You Remove a Node?

Removing a node is basically the process of joining a node in reverse. Instead
of claiming partitions, the node to be removed determines a new cluster state,
taking out all the partitions it currently owns, re-distributing them evenly
across the remaining nodes.

The new state is sent to all nodes in the cluster, not just a random one, so
every node in the cluster knows immediately that the node left. Then it sets
the cluster state on the leaving node, causing handoff to occur, which again is
initialized by vnodes realizing they're not the primary replicas anymore,
transferring the data to the new owners.

When all data is handed off, the Erlang VM process eventually exits.
