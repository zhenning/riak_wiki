In this section, we’ll install Riak and build a four node cluster running on your local machine.  For production deployments, Basho [[recommends a minimum of five nodes|http://basho.com/blog/technical/2012/04/27/Why-Your-Riak-Cluster-Should-Have-At-Least-Five-Nodes/]]. For simplicity, this tutorial uses three.

## Dependencies

Building Riak from source requires Erlang R14B03 or later. Basho's pre-packaged Riak binaries, the latest versions of which can be found in our [[Downloads Directory|http://downloads.basho.com/riak/CURRENT/]], embed the Erlang runtime. However, this tutorial is based on a source build, so if you do not have Erlang already installed, see [[Installing Erlang|Installing-Erlang.html]] for instructions on how to do this.

## Download and Install Riak From Source

The below links provide platform-specific instructions for downloading and installing Riak from source. <br>
<div id ="dl_nav">
	<ul>
		<li><a href="/Installing-on-Debian-and-Ubuntu.html#From-source">Debian and Ubuntu</a></li>
		<li><a href="/Installing-on-RHEL-and-CentOS.html#From-source">RHEL and CentOS</a></li>
		<li><a href="/Installing-on-Mac-OS-X.html#From-source">Mac OS X</a></li>
		<li><a href="/Installing-on-SUSE.html">SUSE</a></li>
		<li><a href="/Installing-Riak-from-Source.html">Installing from Source</a> (to be used on an unlisted-operating system)</li>
		<li><a href="http://github.com/basho/riak">Riak on GitHub</a></li>		
	</ul>	
</div>


## Build Riak

So now you have a copy of Riak. Time to build it. Do this by accessing the "riak" directory and running "make all"

```bash
$ cd riak-1.1.2
$ make all
```

As you can see, "make all" is grabbing all the Riak dependencies for you so that you don't have to chase them down. This should take a few moments.

## Use Rebar to Start Up Three Nodes

Now that Riak is built, we are going to use [[Rebar|https://github.com/basho/rebar]], a packaging and build system for Erlang applications, to get four self-contained Riak nodes running on your machine. Tomorrow, when you put Riak into production, Rebar will enable you to ship a pre-built Riak package to your deployment machines. But for now, we will just stick to the three nodes. To start these up, run "make devrel"

```bash
$ make devrel
```

You have just generated a "dev" directory. Let's go into that directory to check out its contents:

```bash
$ cd dev; ls
```

That should give you the following:

```bash
dev1       dev2       dev3       dev4  
```

Each directory starting with "dev" is a complete package containing a Riak node. We now need to start each node. Let's start with "dev1"

```bash
$ dev1/bin/riak start
```

Then do the same for "dev2", "dev3", and "dev4"

```bash
$ dev2/bin/riak start
$ dev3/bin/riak start
$ dev4/bin/riak start
```

## Test to see the running Riak nodes

After you have the nodes up and running, it's time to test them and make sure they are available. You can do this by taking a quick look at your process list. To do this, run:

```bash
$ ps aux | grep beam
```

This should give you details on four running Riak nodes.

## Join the nodes to make a cluster

The next step is to join these three nodes together to form a cluster. You can do this using the Riak Admin tool. Specifically, what we want to do is join "dev2", "dev3", and "dev4" to "dev1":

```bash
$ dev2/bin/riak-admin join dev1@127.0.0.1
$ dev3/bin/riak-admin join dev1@127.0.0.1
$ dev4/bin/riak-admin join dev1@127.0.0.1
```

<div class="info"><div class="title">About riak-admin</div>
riak-admin is Riak's administrative tool. It's used to do any operational tasks
other than starting and stopping node, e.g. to join and leave a cluster, to back
up data, and to manage general cluster operations. For more information on
riak-admin, check our <a
href="http://wiki.basho.com/Command-Line-Tools.html#riak-admin">page dedicated
to the command line tools</a> that come with Riak.
</div>

## Test the cluster and add some data to verify the cluster is working

Great. We now a have a running three node Riak cluster. Let's make sure it's working correctly. For this we can hit Riak's HTTP interface using _curl_. Try this:

```bash
$ curl -H "Accept: text/plain" http://127.0.0.1:8091/stats
```

Once this runs, look for a field in the output named "ring_ownership." This should show you that all three of your nodes are part of your Riak cluster. You might also notice that each node has claimed a set of partitions. The default partition setting for our three node development cluster is 64. This means that two of the three nodes will have claimed 21 partitions, and the third node will handle the last 22.

<div class="info">
The number of partitions in your cluster is set with by the parameter [[ring_creation_size|Configuration Files#ring_creation_size]] in Riak's app.config file.
</div>

If you want, you can add a file to your Riak cluster and test it's working properly. Let's say, for instance, we wanted to add an image and make sure it was accessible. First, copy an image to your directory if you don't already have one:

```bash
$ cp ~/image/location/image_name.jpg .
```

We can then PUT that image into Riak using a curl command:

```bash
$ curl -X PUT HTTP://127.0.0.1:8091/riak/images/1.jpg \
  -H "Content-type: image/jpeg" --data-binary @image_name.jpg
```

You can then verify that image was in fact stored. To do this, simply copy the URL where we PUT the image and paste it into a browser. Your image should appear.

You should now have a running, four node Riak cluster. Congratulations! That didn't take so long, did it?

<div class="note"><div class="title">HTTP interface ports</div>The above configuration sets up nodes with HTTP interfaces listening on ports 8091-3. The default port for nodes to listen on is 8098 and users will need to take note of this when trying to use any of the default other-language client settings.</div>


**What's Next? You now have a three node Riak cluster up and running.* *[[Time to learn about the basic HTTP API operations.|Basic Riak API Operations]]**

<div class="info"><div class="title">Additional Reading</div>* [[Rebar Documentation|https://github.com/basho/rebar/wiki]]</div>
