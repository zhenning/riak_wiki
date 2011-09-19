To avoid downtime of a Riak cluster we suggest performing upgrades in a rolling
fashion. This process involves stopping, upgrading, and restarting one node at a
time. This process is known to work as of Riak 0.12 (i.e. upgrading 0.12 to
0.13).

<div class="note">If you are upgrading from Riak Search, please first read
[[Upgrading from Riak Search]]
</div>


<div id="toc"></div>

## Note on Upgrading to Riak 1.0

Rolling upgrades should work when moving from Riak 0.13 or later to Riak 1.0
following the OS specific instructions below, but there are a few considerations
to keep in mind when doing so. Riak 1.0 has new features that add additional
steps to the rolling upgrade procedure, specifically Riak Pipe—the new data
processing library backing MapReduce and the updated backend API supporting
asynchronous keylisting. If these features are not explicitly enabled after
upgrading, the legacy variant of the feature will be used instead. These feature
can only be enabled once *all* nodes in the cluster have been upgraded to 1.0.
To enable these new features, add the following to the `riak_kv` section on the
`app.config` file:

```erlang
{legacy_keylisting, false},
{mapred_system, pipe}
```

For the new settings to take effect, you can either restart all of the nodes in
the cluster or use `riak attach` on each node and execute the following
commands:

```erlang
> application:set_env(riak_kv, legacy_keylisting, false).
> application:set_env(riak_kv, mapred_system, pipe).
```

Once you have completed this procedure, you will have a Riak 1.0 cluster taking
full advantage of the improved MapReduce and keylisting capabilities.

## Debian/Ubuntu

The following example demonstrates upgrading a Riak node that has been installed
with the Debian packages provided by Basho.

1\. Stop Riak

```bash
riak stop
```


2\. Backup Riak etc and data directories

```bash
sudo tar -czf riak_backup.tar.gz /var/lib/riak /etc/riak
```


3\. Upgrade Riak

```bash
sudo dpkg -i <riak_package_name>.deb
```


4\. Restart Riak

```bash
riak start
```


5\. Verify Riak is running the new version

```bash
riak-admin status
```


6\. Wait for the riak_kv service to start

```bash
riak-admin wait-for-service riak_kv <target_node>
```


* &lt;target_node&gt; is the node which you have just upgraded (e.g.
riak@192.168.1.11)

7\. Wait for any hinted handoff transfers to complete

```bash
riak-admin transfers
```


* While the node was offline other nodes may have accepted writes on it's
behalf. This data is transferred to the node when it becomes available.

8\. Repeat the process for the remaining nodes in the cluster


## RHEL/Centos

The following example demonstrates upgrading a Riak node that has been installed
with the RHEL/Centos packages provided by Basho.

1\. Stop Riak

```bash
riak stop
```


2\. Backup Riak etc and data directories

```bash
sudo tar -czf riak_backup.tar.gz /var/lib/riak /etc/riak
```


3\. Upgrade Riak

```bash
sudo rpm -Uvh <riak_package_name>.rpm
```


4\. Restart Riak

```bash
riak start
```


5\. Verify Riak is running the new version

```bash
riak-admin status
```


6\. Wait for the riak_kv service to start

```bash
riak-admin wait-for-service riak_kv <target_node>
```


* &lt;target_node&gt; is the node which you have just upgraded (e.g.
riak@192.168.1.11)

7\. Wait for any hinted handoff transfers to complete

```bash
riak-admin transfers
```


* While the node was offline other nodes may have accepted writes on it's
behalf. This data is transferred to the node when it becomes available.

8\. Repeat the process for the remaining nodes in the cluster


## Solaris/OpenSolaris

The following example demonstrates upgrading a Riak node that has been installed
with the Solaris/OpenSolaris packages provided by Basho.

1\. Stop Riak

```bash
riak stop
```



<div class="note">If you are using the service management facility (SMF) to
manage Riak you will have to stop Riak via "svcadm" instead of using "riak
stop":
<br /><br />
```bash
sudo svcadm disable riak
```
</div>


2\. Backup Riak etc and data directories

```bash
sudo gtar -czf riak_backup.tar.gz /opt/riak/data /opt/riak/etc
```


3\. Uninstall Riak

```bash
sudo pkgrm BASHOriak
```


4\. Install the new version of Riak

```bash
sudo pkgadd -d <riak_package_name>.pkg
```



<div class="note">If you are upgrading from Riak 0.12 you will have to restore
the etc directory from the backups made in step 2. The 0.12 package removes the
etc files when uninstalled.</div>


4\. Restart Riak

```bash
riak start
```



<div class="note">If you are using the SMF you should start Riak via "svcadm":
<br /><br />
```bash
sudo svcadm enable riak
```
</div>


5\. Verify Riak is running the new version

```bash
riak-admin status
```


6\. Wait for the riak_kv service to start

```bash
riak-admin wait-for-service riak_kv <target_node>
```


* &lt;target_node&gt; is the node which you have just upgraded (e.g.
riak@192.168.1.11)

7\. Wait for any hinted handoff transfers to complete

```bash
riak-admin transfers
```


* While the node was offline other nodes may have accepted writes on it's
behalf. This data is transferred to the node when it becomes available.

8\. Repeat the process for the remaining nodes in the cluster