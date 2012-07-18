<div class="info"><div class="title">Riak Enterprise Only</div>This documentation only applies to the Enterprise version of Riak.</div>
<<<<<<< HEAD

## The riak-repl Command
Replication is controlled by the `riak-repl` command. Usage:

**add-listener**
Adds a listener (primary) to the given node, IP address and port.

  * *Syntax:* `riak-repl add-listener <nodename> <listen_ip> <port>`
  * *Example:* `riak-repl add-listener riak@10.0.1.156 10.0.1.156 9010`

**del-listener**
Removes and shuts down a listener (primary) on the given node, IP address and port.

  * *Syntax:* `riak-repl del-listener <nodename> <listen_ip> <port>`
  * *Example:* `riak-repl del-listener riak@10.0.1.156 10.0.1.156 9010`

**add-site**
Adds a site (secondary) to the local node, connecting to the specified listener.

  * *Syntax:* `riak-repl add-site <ipaddr> <portnum> <sitename>`
  * *Example:* `riak-repl add-site 10.0.1.156 9010 newyork`

**del-site**
Removes a site (secondary) from the local node by name.

  * *Syntax:* `riak-repl del-site <sitename>`
  * *Example:* `riak-repl del-site newyork`

**status**
Gets status information about replication. Reports counts on how much data has been transmitted, transfer rates, message queue lengths of clients and servers, number of fullsyncs, and connection status. This command only displays useful information on the leader node.

  * *Syntax:* `riak-repl status`

**start-fullsync**
Manually initiates full synchronization with connected sites.

  * *Syntax:* `riak-repl start-fullsync`

**cancel-fullsync**
Cancels any full synchronizations in progress. If a partition is in progress, synchronization will stop after that partition completes. During cancellation, riak-repl status will show 'cancelled' in the status.

  * *Syntax:* `riak-repl cancel-fullsync`

**pause-fullsync**
Pauses any full synchronizations in progress. If a partition is in progress, synchronization will pause after that partition completes. While paused, riak-repl status will show 'paused' in the status information. Fullsync may be cancelled while paused.

  * *Syntax:* `riak-repl pause-fullsync`

**resume-fullsync**
Resumes any full synchronizations that were paused. If fullsync was running at the time of the pause, the next partition will be synchronized. If not, it will wait until the next start-fullsync command/ fullsync_interval.

  * *Syntax:* `riak-repl resume-fullsync`
=======
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
