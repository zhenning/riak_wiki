# Welcome to the Riak Wiki

![Riak Logo](images/riaklogo.png)


Riak is the most powerful open-source, distributed database you'll ever put into production. 
<div id ="new_nav">
	<ul id="top_list">	
		<li><a href="Installation.html">Download and Install</a></li>
		<li><a href="The-Riak-Fast-Track.html">Riak Fast Track</a></li>	
		<li><a href="Community.html">Community</a></li>
		<li><a href="Client-Libraries.html">Client Libraries</a></li>
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
		 <td><h2>1000s of Users</h2>Comcast, Yammer, Voxer, Boeing, BestBuy, SEOMoz, Joyent, Kiip, DotCloud, Formspring, GitHub, and the Danish Government <a href="http://basho.com/company/production-users/">are just a few</a> of the thousands of startups and enterprises that have deployed Riak.</td>
			 <td><h2>Fault Tolerant</h2>Decide how many replicas of the data you want (start at 3). If nodes go down, requests are routed transparently to other nodes. Riak uses proven architectural principles like hinted handoff and read repair so <a href="/Replication.html">you can always write and read data</a>, even in failure conditions. </td>
        <td><h2>Complex Queries</h2>In addition to key/value access to your data, Riak has built-in support for MapReduce, Full Text Search, and Secondary Indexes, giving developers <a href="/MapReduce-Search-2i-Comparison.html">various ways to store and query their data</a>. 
		</td>
    </tr>
</table>
