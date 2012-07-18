<div class="info"><div class="title">Riak Enterprise Only</div>This documentation only applies to the Enterprise version of Riak.</div>

The configuration for replication is kept in the riak_repl section of `etc/app.config`.

    {riak_repl, [
                 {fullsync_on_connect, true},
                 {fullsync_interval, 360},
                 % Debian/Centos/RHEL:
                 {data_root, "/var/lib/riak/data/riak_repl"},
                 % Solaris:
                 % {data_root, "/opt/riak/data/riak_repl"},
                 % FreeBSD/SmartOS:
                 % {data_root, "/var/db/riak/riak_repl"},
                 {queue_size, 104857600},
                 {server_max_pending, 5},
                 {client_ack_frequency, 5}
              ]}

<<<<<<< HEAD
**fullsync_on_connect**
=======
**fullsync_on_connect**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
Whether to initiate a fullsync on initial connection from the secondary cluster.

  * *Value:* true | false
  * *Default:* true
  * *Available:* Riak Enterprise (All)

<<<<<<< HEAD
**fullsync_interval**
=======
**fullsync_interval**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
How often to initiate a full synchronization of data, in minutes. This is measured from the completion of one full-sync operation to the initiation of the next. This setting only applies to the primary cluster (listener). To disable full synchronization, use: `disabled`.

  * *Value:* minutes [integer] | disabled
  * *Default:* 360 (6 hours)
  * *Available:* Riak Enterprise (All)

**data_root**
Path (relative or absolute) to the working directory for the replication process.

  * *Value:* path [string]
  * *Default:* data/riak_repl
  * *Available:* Riak Enterprise (All)

<<<<<<< HEAD
**queue_size**
=======
**queue_size**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
The size of the replication queue in bytes before the replication leader will drop requests. If requests are dropped (the information is available in riak-repl status) a full_sync will be required.

  * *Value:* bytes [integer]
  * *Available:* Riak Enterprise (All)

<<<<<<< HEAD
**server_max_pending**
=======
**server_max_pending**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
The maximum number of objects the leader will wait to get an acknowledgement from the remote location before queuing the request.

  * *Value:* number of objects [integer]
  * *Available:* Riak Enterprise (All)

<<<<<<< HEAD
**client_ack_frequency**
=======
**client_ack_frequency**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
The number of requests a remote leader will handle before sending an acknowledgment to the remote cluster.

  * *Value:* number of requests [integer]
  * *Available:* Riak Enterprise (All)

<<<<<<< HEAD
**sndbuf**
=======
**sndbuf**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
The buffer size for the listener (server) socket measured in bytes.

  * *Value:* bytes [integer]
  * *Default:* OS dependent
  * *Available:* Riak Enterprise (1.1+)

<<<<<<< HEAD
**recbuf**
=======
**recbuf**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
The buffer size for the site (client) socket measured in bytes.

  * *Value:* bytes [integer]
  * *Default:* OS dependent
  * *Available:* Riak Enterprise (1.1+)

<<<<<<< HEAD
**vnode_gets**
=======
**vnode_gets**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
If true, repl will do a direct get against the vnode, rather than use a GET finate state machine.

  * *Value:* true | false
  * *Default:* true
  * *Available:* Riak Enterprise (1.1+)

<<<<<<< HEAD
**shuffle_ring**
=======
**shuffle_ring**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
Toggles whether the ring is traversed in-order or shuffled randomly.

  * *Value:* true | false
  * *Default:* true (shuffle enabled)
  * *Available:* Riak Enterprise (1.1+)

<<<<<<< HEAD
**diff_batch_size**
=======
**diff_batch_size**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
Defines how many fullsync objects to send before waiting for an acknowledgement from the client site.

  * *Value:* object count [integer]
  * *Default:* 100
  * *Available:* Riak Enterprise (1.1+)

<<<<<<< HEAD
**max_get_workers**
=======
**max_get_workers**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
The maximum number of get workers spawned for fullsync. Every time a replication difference is found, a GET will be performed to get the actual object to send.

_Note:_ Each get worker spawns 2 processes, one for the work, and one for the get FSM. Be sure you don't run over the maximum number of allowed processes in an Erlang VM (check vm.args for a +P property)

  * *Value:* worker count [integer]
  * *Default:* 100
  * *Available:* Riak Enterprise (1.1+)

<<<<<<< HEAD
**max_put_workers**
=======
**max_put_workers**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
The maximum number of put workers spawned for fullsync. Every time a replication difference is found, a GET will be performed to get the actual object to send.

_Note:_ Each put worker spawns 2 processes, one for the work, and one for the put FSM. Be sure you don't run over the maximum number of allowed processes in an Erlang VM (check vm.args for a +P property)

  * *Value:* worker count [integer]
  * *Default:* 100
  * *Available:* Riak Enterprise (1.1+)

<<<<<<< HEAD
**min_get_workers**
=======
**min_get_workers**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
The minimum number of get workers spawned for fullsync. Every time a replication difference is found, a GET will be performed to get the actual object to send.

_Note:_ Each get worker spawns 2 processes, one for the work, and one for the get FSM.

  * *Value:* worker count [integer]
  * *Default:* 5
  * *Available:* Riak Enterprise (1.1+)

<<<<<<< HEAD
**min_put_workers**
=======
**min_put_workers**  
>>>>>>> f0bf532094bc429dbdbfb7bf91c005dee40bebd5
The minimum number of put workers spawned for fullsync. Every time a replication difference is found, a GET will be performed to get the actual object to send.

_Note:_ Each put worker spawns 2 processes, one for the work, and one for the put FSM.

  * *Value:* worker count [integer]
  * *Default:* 5
  * *Available:* Riak Enterprise (1.1+)
