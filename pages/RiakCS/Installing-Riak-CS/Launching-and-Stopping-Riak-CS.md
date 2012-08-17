# Launching and Stopping Riak
To launch Riak CS in the background, enter:

```bash
sudo riak-cs start 
```

To run Riak CS with an interactive Erlang console:

```bash
sudo riak-cs console
```

When Riak CS is running, the Riak CS process appears in the process list. To check for the Riak CS process, enter:

```bash
ps -ef|grep riak-cs
```

To stop Riak CS, enter:

```bash
sudo riak-cs stop
```

You can use the command:

```bash
sudo riak-cs attach
```

to attach and obtain an interactive console to a running instance of Riak CS.
