# Welcome to the Riak Wiki

![Riak Logo](images/riaklogo.png)


Riak is the most powerful open-source, distributed database you'll ever put into production. Riak scales predictably and easily and simplifies development by giving users the ability to quickly prototype, test, and deploy their applications.

A truly fault-tolerant system, Riak has no single point of failure. No machine is special or central in Riak, so developers and operations professionals can decide exactly how fault-tolerant they want and need their applications to be.

<div id ="new_nav">
	<ul id="top_list">
		<li><a href="#">Why Riak</a></li>
		<li><a href="#downloads">Download and Install</a></li>		
		<li><a href="#community">Community</a></li>
		<li><a href="#client_libraries">Client Libraries</a></li>
		<li><a href="#contribute">Contribute</a></li>
	</ul>
</div>	


<table id="new">
	<tr>
        <td><h2>Scalable</h2>Riak is built so you can add more capacity as your app or platform grows. When you add new machines, Riak <a href="/Concepts.html#Distribution">distributes data automatically around the cluster</a> with no downtime and a near-linear increase in performance and throughput.</td>
		<td><h2>Simple Ops</h2>Riak is the most boring database you’ll ever run in production. No sharding required, just <a href="/Adding-and-Removing-Nodes.html">horizontal scaling</a> and straight-forward capacity planning. The same operational tasks apply to small clusters and large clusters. More machines does not mean more ops.</td>
		 <td><h2>Masterless</h2>A Riak cluster is masterless. <a href="/Concepts.html#No-master-node">No node is special</a> and any node can handle requests for any other node in the cluster. All requests to Riak happen concurrently and developers don't have to spend time worrying about read/write locks or single points of failure.</td>
    </tr>	
	<tr>
		<td><h2>Language Support</h2>Basho and the Riak Community <a href="/Client-Libraries.html">maintain libraries for most major languages</a> including Java, Node.js, Python, Ruby, PHP, C/C++, and many more. All our client code, like the core Riak code, is available on GitHub and is awaiting your contributions.</td>
			<td><h2>Powerful Community</h2>The <a href="/Community.html">Riak community</a> is composed of smart, passionate hackers who are contributing code, support, and evangelism every day. We have hundreds of companies and individuals testing, breaking, running, and building Riak with us.</td>
			<td><h2>Flexible APIs</h2> 
				Riak is equipped with fully-featured HTTP and Protocol Buffers APIs, with support for both of these transports in all of our supported client libraries. You can also write your own client layer to <a href="/Client-Implementation-Guide.html">wrap our APIs</a> if your use case or preference calls for it.</td>
    </tr>
	<tr>
		 <td><h2>1000s of Deployments</h2>Riak users include Comcast, Yammer, Voxer, Boeing, SEOMoz, Joyent, Kiip.me, DotCloud, Formspring, Boeing, and the Danish Government, just to name a few of the thousands of startups and enterprises that trust Riak.</td>
			 <td><h2>Fault Tolerant</h2>Decide how many replicas of the data you want (start at 3). If nodes go down, requests are routed transparently to other nodes. Riak uses proven architectural principles like hinted handoff and read repair so <a href="/Replication.html">you can always write and read data</a>, even in failure conditions. </td>
        <td><h2>Complex Queries</h2>In addition to key/value access to your data, Riak has built-in support for MapReduce, Full Text Search, and Secondary Indexes, giving developers <a href="/MapReduce-Search-2i-Comparison.html">various ways to store and query their data</a>. 
		</td>
    </tr>
</table>

<a name ="downloads"><h2>Download and Install</a>
	
<p>Basho provides Riak packages for a wide variety of platforms and architectures. Select your preferred environment for instructions on how to get up and running.</p>  

<div id ="dl_nav">
	<ul>
		<li><a href="/Installing-on-Debian-and-Ubuntu.html">Debian and Ubuntu</a></li>
		<li><a href="/Installing-on-RHEL-and-CentOS.html">RHEL and CentOS</a></li>
		<li><a href="/Installing-on-Mac-OS-X.html">Mac OS X</a></li>
		<li><a href="/Installing-on-SUSE.html">SUSE</a></li>
		<li><a href="/Installing-Riak-from-Source.html">Installing from Source</a></li>
		<li><a href="http://github.com/basho/riak">Riak on GitHub</a></li>		
	</ul>	
</div>	

<h2>The Riak Fast Track</h2>

In addition to downloading and installing Riak to get started, you might want to take some time and go through the [[Riak Fast Track|The-Riak-Fast-Track]]. This is an extensive tutorial that will take you from [[building a three node Riak cluster|http://wiki.basho.com/Building-a-Development-Environment.html]] on your laptop to using [[Riak's HTTP API|http://wiki.basho.com/Basic-Riak-API-Operations.html]] to seeing Riak's [[fault-tolerance in action|http://wiki.basho.com/Tunable-CAP-Controls-in-Riak.html]].


<a name ="community"><h2>Community</a>

<p>The Riak community is an exciting place to live and work. Some of the ways to get involved immediately:</p>

<div id="">
	<ul>
		<li><a href="lists.basho.com/mailman/listinfo/riak-users_lists.basho.com">The Riak Mailing List</a> - This is the primary mailing list for all things Riak. There are thousands of developers discussing here every day.</li>
		<li><a href=""></a>Riak on IRC</a> - The Riak IRC Room on Freenode is filled with hundres of users and core Riak developers. It's the fastest way to get help with a Riak issue or get a question answered.</li>
		<li><a href="http://basho.com/community/">Riak Community Home</a> - An extensive, focused home for the Riak Community featuring events, user groups, videos, and more.</li>
		<li><a href="https://twitter.com/#!/basho">Basho on Twitter</a> Follow the Basho Engineering Team on Twitter</li>
		<li><a href="http://planetriak.org/">Planet Riak</a> - A blog aggregator for all-things Riak</li>
	</ul>

<p>You can find a full list of Community Resources <a href="http://wiki.basho.com/Community.html">here</a>.</p>

<a name ="client_libraries"><h2>Client Libraries</a></h2>

Looking for a specific Riak client library? Basho's [[list of supported clients|Client Libraries]] includes Erlang, Java, PHP, Python and Ruby. Here's a list of what we support currently:

<div id ="">
	<ul>
		<li><a href="/Client-Libraries.html#C-C%2B%2B">C/C++</a></li>
		<li><a href="/Client-Libraries.html#Erlang">Erlang</a></li>
		<li><a href="/Client-Libraries.html#Java">Java</a></li>
		<li><a href="/Client-Libraries.html#%0A--PHP%0A">PHP</a></li>
		<li><a href="/Client-Libraries.html#Python"><a>Python</a></li>
		<li><a href="/Client-Libraries.html#Ruby">Ruby</a></li>
	</ul>	
</div>

<div><p>There's also a ton of [[community contributed code|Community-Developed-Libraries-and-Projects]], including fully-featured Node.js and Perl drivers, and various monitoring tools for things like Munin, Cacti, and Nagios.</p></div>

<a name ="contribute"><h2>Contribute</a>

<p>Basho is actively accepting contributions on all of our code. We host everything on GitHub, and strive to make our repositories and other resources as accessible and easy to use as possible. You can start with <a href="http://github.com/basho/">all of Basho's Open Source code</a> or check out some of these select repos:</p>

<div id="">
	<ul>
	<li><a href="https://github.com/basho/Riak">Riak</a></li> 
	<li><a href="https://github.com/basho/riak_wiki">Riak Wiki</a></li> 
	<li><a href="https://github.com/basho/riak_core">Riak Core</a></li>
	<li><a href="https://github.com/basho/riak_kv">Riak KV</a></li>
	<li><a href="https://github.com/basho/riak-java-client">Riak Java Client</a></li> 
	<li><a href="https://github.com/basho/riak-ruby-client">Riak Ruby Client</a></li>
	</ul>
</div>




