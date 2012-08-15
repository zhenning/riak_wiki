<p>This section provides an overview of the features available in the Riak CS API.</p>
<p>Riak and Riak CS can both be installed on the same machine. Riak CS can also be installed&nbsp;on a different machine when the Riak machine is accessible over a network. Riak CS&nbsp;communicates with Riak via the Riak Erlang client, which uses Protocol Buffers. When&nbsp;Riak CS runs on multiple machines, you can set up load balancing between the nodes.&nbsp;Riak CS can store files up to 5GB in size.</p>
<h2>Storage API (Amazon S3‐compatible)</h2>
<p>The Riak CS API (version 1) is a subset of the Amazon S3 API.</p>
<h3>Service‐level operations</h3>
<ul>
<li>GET Service ‐ Returns a list of all buckets owned by the user who sent the request</li>
</ul>
<h3><span style="line-height: normal;">Bucket‐level operations</span></h3>
<div>
<ul>
<li><span style="background-color: white;">GET Bucket&nbsp;‐&nbsp;Returns a list of the objects within a bucket</span></li>
<li><span style="background-color: white;"></span><span style="background-color: white;">GET Bucket ACL&nbsp;‐&nbsp;Returns the ACL associated with a bucket</span></li>
<li><span style="background-color: white;">PUT Bucket&nbsp;‐&nbsp;Creates a new bucket</span></li>
<li><span style="background-color: white;">PUT Bucket ACL&nbsp;‐&nbsp;Sets the ACL permissions for a bucket</span></li>
<li><span style="background-color: white;">DELETE Bucket&nbsp;‐&nbsp;Deletes a bucket</span></li>
</ul>
<h3>Object‐level operations</h3>
<div>
<ul>
<li><span style="background-color: white;">GET Object&nbsp;‐&nbsp;Retrieves an object</span></li>
<li><span style="background-color: white;"></span><span style="background-color: white;">GET Object ACL&nbsp;‐&nbsp;Returns the ACLs associated with an object</span></li>
<li><span style="background-color: white;">PUT Object&nbsp;‐&nbsp;Stores an object to a bucket</span></li>
<li><span style="background-color: white;">PUT Object ACL&nbsp;‐&nbsp;Sets the ACLs associated with an object</span></li>
<li><span style="background-color: white;">HEAD Object&nbsp;‐&nbsp;Retrieves object metadata (not the full object content )</span></li>
<li><span style="background-color: white;">DELETE Object&nbsp;‐&nbsp;Deletes an object</span></li>
</ul>
<h3>Provisioning API</h3>
</div>
<div>
<ul>
<li><span style="background-color: white;">CREATE User ‐ Creates a user account</span></li>
</ul>
</div>
</div>
<h3>Reporting API</h3>
<div>
<ul>
<li><span style="background-color: white;">GET Usage&nbsp;‐&nbsp;Special resource to query storage and access usage data on a per user&nbsp;</span><span style="background-color: white;">basis.</span></li>
</ul>
<p>Access statistics are available as JSON or XML for individual users via an HTTP API. To query access statistics, use the resource:</p>
<pre>/usage/$USER_KEY_ID</pre>
<p>For example, to request the usage statistics for a user, use the URL:</p>
<pre>http://storage‐address/usage/user‐key‐id</pre>
<p>Substitute 'storage‐address' for the IP address or domain name of your Riak storage and 'user‐key‐id' with the key for the specific user.</p>
</div>