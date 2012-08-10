<div class="info">This document only applies to Riak 1.2+.</div>

Riak Search indexes currently have no form of anti-entropy (such as read-repair). Furthermore, for performance and load balancing reasons, Search reads from 1 random node. This means that when a replica loss has occurred, inconsistent results may be returned.

## Running a Repair

If a replica loss has occurred, you need to run the repair command. Note that this is a short-term fix until we add anti-entropy into Riak search. This command repairs objects from a node's adjacent partitions on the ring, consequently fixing the search index.

This is done as efficiently as possible by generating a hash range for all the buckets and thus avoiding a preflist calculation for each key. Only a hash of each key is done, its range determined from a bucket->range map, and then the hash is checked against the range.

This code will force all keys in each partition on a node to be reread, thus rebuilding the search index properly.

<ol>
<li>From a cluster node with Riak installed, attach to the Riak console:

```bash
$ riak attach
```

You may have to hit enter again to get an console prompt.
</li>
<li>Get a list of partitions owned by the node that needs repaired:

```erlang
> {ok, Ring} = riak_core_ring_manager:get_my_ring().
```

You will get a lot of output with Ring record information. You can safely ignore it.
</li>
<li>Then run the following code to get a list of partitions. Replace 'dev1@127.0.0.1' with the name of the node you need to repair.

```erlang
> Partitions = [P || {P, 'dev1@127.0.0.1'} <- riak_core_ring:all_owners(Ring)].
```

_Details Note: The above is an [Erlang list comprehension](http://www.erlang.org/doc/programming_examples/list_comprehensions.html), that loops over each `{Partition, Node}` tuple in the Ring, and extracts only the partitions that match the given node name, as a list._

</li>
<li>Execute repair on all the partitions. Executing them all at once like this will cause a lot of `{shutdown,max_concurrency}` spam but it's not anything to worry about. That is just the transfers mechanism enforcing an upper limit on the number of concurrent transactions.

```erlang
> [riak_search_vnode:repair(P) || P <- Partitions].
```
</li>
<li>When you're done, press `ctrl-D` to disconnect the console. DO NOT RUN q() which will cause the running Riak node to quit. Note that `ctrl-D` merely disconnects the console from the service, it does not stop the code from running.
</li>
</ol>

## Checking in on a Repair

The above Repair command can be slow, so if you reattach to the console, you can run the repair_status function. You can use the `Partitions` variable defined above to get the status of every partition.

```erlang
> [{P, riak_search_vnode:repair_status(P)} || P <- Partitions].
```

When you're done, press `ctrl-D` to disconnect the console.

## Killing a Repair

Sometimes you want to stop a repair from running, for whatever reason.

If you happen to be attached on the node you wish to kill repair, just run:

```erlang
> riak_core_vnode_manager:kill_repairs(killed_by_user).
```

Log entries will reflect that repairs were killed manually, something akin to this:

```
2012-08-10 10:14:50.529 [warning] <0.154.0>@riak_core_vnode_manager:handle_cast:395 Killing all repairs: killed_by_user
```

If you are not attached to the node whose repair you are killing, you'll have to instead execute the call remotely, using Erlang's RPC module.

```erlang
> rpc:call('dev1@127.0.0.1', riak_core_vnode_manager, kill_repairs, [killed_by_user]).
```
When you're done, press `ctrl-D` to disconnect the console.
