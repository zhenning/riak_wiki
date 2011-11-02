This script performs operations not related to node-liveness, including node
membership, backup, and basic status reporting.  The node must be running for
most of these commands to work.


```bash
Usage: riak-admin { join | leave | backup | restore | test | status |
                    reip | js_reload | wait-for-service | ringready |
                    transfers | force-remove | down | cluster_info | 
                    member_status | ring_status | vnode-status }
```


## join

Joins the running node to another running node so that they participate in the
same cluster.

* &lt;node&gt; is the other node to connect to.


```bash
riak-admin join <node>
```


## leave

Causes the node to leave the cluster it participates in.


```bash
riak-admin leave
```


## backup

Backs up the data from the node or entire cluster into a file.

* &lt;node&gt; is the node from which to perform the backup.
* &lt;cookie&gt; is the Erlang cookie/shared secret used to connect to the node.
This is "riak" in the [[default configuration|Configuration Files#\-setcookie]].
* &lt;filename&gt; is the file where the backup will be stored. _This should be
the full path to the file._ 
* [node|all] specifies whether the data on this node or the entire cluster will
be backed up, respectively.


```bash
riak-admin backup <node> <cookie> <filename> [[node|all]]
```


## restore

Restores data to the node or cluster from a previous backup.

* &lt;node&gt; is the node which will perform the restore.
* &lt;cookie&gt; is the Erlang cookie/shared secret used to connect to the node.
This is "riak" in the [[default configuration|Configuration Files]].
* &lt;filename&gt; is the file where the backup is stored. _This should be the
full path to the file._


```bash
riak-admin restore <node> <cookie> <filename>
```


## test

Runs a test of a few standard Riak operations against the running node.


```bash
riak-admin test
```


## status

Prints status information, including performance statistics, system health
information, and version numbers. The statistics-aggregator must be enabled in
the [[configuration|Configuration Files#riak_kv_stat]] for this to work.


```bash
riak-admin status
```


## reip

Renames a node.  The current ring state will be backed up in the process. **The
node must NOT be running for this to work.**


```bash
riak-admin reip <old nodename> <new nodename>
```


## js_reload

Forces the embedded Javascript virtual machines to be restarted. This is useful
when deploying new custom built-in [[MapReduce]] functions. (_This needs to be
run on all nodes in the cluster_.)


```bash
riak-admin js_reload
```


## wait-for-service

Waits on a specific watchable service to be available (typically _riak_kv_).
This is useful when (re-)starting a node while the cluster is under load. Use
"services" to see what services are available on a running node.


```bash
riak-admin wait-for-service <service> <nodename>
```


## services

Lists available services on the node (e.g. *riak_kv*).


```bash
riak-admin services
```


## ringready

Checks whether all nodes in the cluster agree on the ring state. Prints "FALSE"
if the nodes do not agree. This is useful after changing cluster membership to
make sure that ring state has settled.


```bash
riak-admin ringready
```


## transfers

Identifies nodes that are awaiting transfer of one or more partitions. This
usually occurs when partition ownership has changed (after adding or removing a
node) or after node recovery.


```bash
riak-admin transfers
```

## force-remove

Removes the specified node from the cluster.

```bash
riak-admin force-remove <node>
```

## down

Marks a node as down so that ring transitions can be performed before the node 
is brought back online.


```bash
riak-admin down <node>
```

## cluster_info

Output system information from a riak cluster. This command will collect
information from all nodes or a subset of nodes and output the data to a single
text file.

The following information is collected:

 * Current time and date
 * VM statistics
 * erlang:memory() summary
 * Top 50 process memory hogs
 * Registered process names
 * Registered process name via regs()
 * Non-zero mailbox sizes
 * Ports
 * Applications
 * Timer status
 * ETS summary
 * Nodes summary
 * net_kernel summary
 * inet_db summary
 * Alarm summary
 * Global summary
 * erlang:system_info() summary
 * Loaded modules
 * Riak Core config files
 * Riak Core vnode modules
 * Riak Core ring
 * Riak Core latest ring file
 * Riak Core active partitions
 * Riak KV status
 * Riak KV ringready
 * Riak KV transfers

```bash
riak-admin cluster_info <output file> [<node list>]
```

Examples:

```bash
# Output information from all nodes to /tmp/cluster_info.txt
riak-admin cluster_info /tmp/cluster_info.txt
```

```bash
# Output information from the current node
riak-admin cluster_info /tmp/cluster_info.txt local
```

```bash
# Output information from a subset of nodes
riak-admin cluster_info /tmp/cluster_info.txt riak@192.168.1.10
riak@192.168.1.11
```

## member_status

Prints the current status of all cluster members.

```bash
riak-admin member_status
```

## ring_status

Ouputs the current claimant, its status, ringready, pending ownership handoofs, 
and a list of unreachable nodes.

```bash
riak-admin ring_status
```

## vnode-status

Ouputs the status of all vnodes the are running on the local node.

```bash
riak-admin vnode-status
```
