The following article discusses standard configurations and port settings to use
when thinking about how to secure your Riak Cluster.

There are two classes of access control for Riak:

- other Riak nodes in the ring
- clients making use of the Riak ring

For both access groups, the settings you want are in riak/etc/app.config.  The
config directives you care about for client access all end in "_ip" and "_port":
`web_ip`, `web_port`, `pb_ip`, and `pb_port`.  Make note of those and configure
your firewall to incoming TCP access to those ports or IP and port combinations.
 The exceptions to this is the `handoff_ip` and `handoff_port` directives. 
Those are for communication between Riak nodes only.

Riak uses the Erlang distribution mechanism for most inter-node communication. 
Riak identifies other machines in the ring using Erlang identifiers (`<hostname
or IP>`, i.e. `riak@10.9.8.7`).  Erlang resolves these node identifiers to a TCP
port on a given machine via the Erlang Port Mapper daemon(epmd) running on each
machine in a ring.  epmd listens on TCP port 4369 on the wildcard interface. For
inter-node communication, Erlang uses an unpredictable port by default; it binds
to port 0, which means the first available port.

For ease of firewall configuration you can configure Riak to tell the Erlang
interpreter to only use a limited range of ports in riak/etc/app.config.  For
example, to restrict the range of ports that Erlang will use for inter-Erlang
node communication to 6000-7999, add the following lines to riak/etc/app.config:


```bash
{ kernel, [
            {inet_dist_listen_min, 6000},
            {inet_dist_listen_max, 7999}
          ]},
```


This goes in the top level list in app.config, at the same level as all the
other applications (eg. riak_core).


Then just configure your firewall to allow incoming access to TCP ports 6000 to
7999 from whichever network(s) contain your Riak nodes.

**Riak nodes in ring need to be able to communicate freely with one another on
the following ports:**

- epmd's listener: TCP:4369
- handoff_port listener: TCP:8099
- range of ports you configure in app.config


**Riak clients need to be able to contact a at least one machine in a Riak ring
on the following ports:**

- web_port: TCP:8098
- pb_port: TCP:8097

One important note: if you do add the `inet_dist_listen_min` and
`inet_dist_listen_max` entries to riak/etc/app.config, you need to kill off any
running epmd so it it will pick up the new settings.  epmd will continue to run
on a given machine even after all Erlang interpreters have exited.
