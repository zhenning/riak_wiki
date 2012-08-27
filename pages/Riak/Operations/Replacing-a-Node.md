At some point, for various reasons, you might need to replace a node in your cluster (which is different from [[recovering a failed node|Recovering a failed node]]). Here is the recommended way to go about doing that:

1. Tar up your data directory on the node in question.
2. Download and install Riak on the new node you wish to bring into the cluster. Don't start it just yet.
3. Copy the data from the node you're decommissioning to the new machine and untar it. Also make sure to copy the existing ring file to the new node.
4. Run [riak-admin reip](http://wiki.basho.com/Command-Line-Tools.html#reip) on the new node.
5. Stop the old node using [riak stop](http://wiki.basho.com/Command-Line-Tools.html#stop).
6. Start the new machine with [riak start](http://wiki.basho.com/Command-Line-Tools.html#start).

<div class="info"><div class="title">Ring Settling</div>You'll need to make sure that no other ring changes occur between the time when you start the new node and the ring settles with the new IP info.

The ring is considered settled when the new node reports <strong>true</strong> using the <a href="/Command-Line-Tools.html#ringready">riak-admin ringready</a> command.
</div>