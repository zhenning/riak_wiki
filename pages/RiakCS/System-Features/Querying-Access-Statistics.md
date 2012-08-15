<p>Access statistics are tracked on a per-user basis, as rollups for slices of time. Querying these statistics is done via the <tt>/usage/$USER_KEY_ID</tt> resource.</p>
<p>For information about how access statistics are logged, please read <a href="https://help.basho.com/entries/21560857-usage-and-billing-data">Usage and Billing Data</a>.</p>
<p>The following sections discuss accessing the access statistics using bare HTTP requests. Query parameters are used to specify the types and date ranges of information to include. For information on using <tt>s3cmd</tt> (or other tools) to fetch statistics as S3 objects, skip to the <a href="#the-magic-usage-bucket">The Magic&nbsp;<tt>usage</tt>&nbsp;Bucket</a>&nbsp;section.</p>
<h2>Choosing the Result Format</h2>
<p>Results are available as either JSON or XML. Request the appropriate one by using the HTTP <tt>Accept</tt> header (with either <tt>application/json</tt> or <tt>application/xml</tt>, respectively).</p>
<h2>Specifying the User</h2>
<p>Access statistics are provided on a per-user basis. Specify which user's statistics you want by providing that user's <tt>key_id</tt> in the URL. For example, to get access statistics for the user key <tt>8NK4FH2SGKJJM8JIP2GU</tt>, use the URL <tt>/usage/8NK4FH2SGKJJM8JIP2GU</tt>. <em>Note</em>: the new user id generator should not include non-URL-safe characters, but if it does, those characters will need to be escaped in this url.</p>
<p>A&nbsp;<tt>404</tt> code with an error message body will be returned if the user does not exist. For example, there is no <tt>ASDF</tt> user in my cluster, so fetching <tt>http://localhost:8080/usage/ASDF</tt> produces:</p>
<p>JSON:</p>
<pre>HTTP/1.1 404 Object Not Found 

{"Error":{"Message":"Unknown user"}} 
</pre>
<p>XML (reformatted for easy reading):</p>
<pre>HTTP/1.1 404 Object Not Found 

&lt;?xml version="1.0" encoding="UTF-8"?&gt; 
&lt;Error&gt;&lt;Message&gt;Unknown user&lt;/Message&gt;&lt;/Error&gt; 
</pre>
<h2>Enable Access Results</h2>
<p>The usage HTTP resource provides both access and storage statistics. Since each of these queries can be taxing in its own right, they are both omitted from the result by default:</p>
<pre>curl http://localhost:8080/usage/8NK4FH2SGKJJM8JIP2GU
</pre>
<p>JSON:</p>
<pre>{"Access":"not_requested","Storage":"not_requested"} 
</pre>
<p>XML (reformatted for easy reading):</p>
<pre> &lt;?xml version="1.0" encoding="UTF-8"?&gt; 
&lt;Usage&gt; 
   &lt;Access&gt;not_requested&lt;/Access&gt;
   &lt;Storage&gt;not_requested&lt;/Storage&gt; 
&lt;/Usage&gt; 
</pre>
<p>To request that access results be included, pass the query parameter <tt>a</tt> to the resource (any true-ish value will work, including just the bare <tt>a</tt>, <tt>t</tt>, <tt>true</tt>, <tt>1</tt>, <tt>y</tt>, and <tt>yes</tt>):</p>
<pre>curl http://localhost:8080/usage/8NK4FH2SGKJJM8JIP2GU?a
</pre>
<p>JSON (reformatted for easy reading):</p>
<pre>{"Access":[{"Errors":[]}], "Storage":"not_requested"}
</pre>
<p>XML (reformatted for easy reading):</p>
<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt; 
&lt;Usage&gt; 
   &lt;Access&gt;&lt;Errors/&gt;&lt;/Access&gt; 
   &lt;Storage&gt;not_requested&lt;/Storage&gt; 
&lt;/Usage&gt; 
</pre>
<p>There are no statistics included in this report because the default time span is <em>now</em>, which is not available in the archives.</p>
<h2>Specifying the Time Span to Report</h2>
<p>Request the time span you want data for by passing <tt>s</tt> (start) and <tt>e</tt> (end) query parameters to the resource. The slices for which data will be returned are all of those between <tt>s</tt> and <tt>e</tt>, as well as the slice including <tt>s</tt> and the slice including <tt>e</tt>.</p>
<p>For example, for slices <tt>A</tt>-<tt>I</tt>:</p>
<pre>   A     B     C     D     E     F     G     H     I 
|-----|-----|-----|-----|-----|-----|-----|-----|-----| 
              s                  e 
</pre>
<p>Specifying an <tt>s</tt> that falls somewhere in slice <tt>C</tt> and an <tt>e</tt> that falls somewhere in slice <tt>F</tt> means that data for slices <tt>C</tt>, <tt>D</tt>, <tt>E</tt>, and <tt>F</tt> will be returned.</p>
<p>Each should be provided in ISO8601 format (<tt>yyyymmddThhmmssZ</tt>). For example, the following values would request the span between 2:00pm and 4:00pm (GMT) on January 30, 2012:</p>
<pre>http://localhost:8080/usage/8NK4FH2SGKJJM8JIP2GU?a&amp;s=20120315T140000Z&amp;e=20120315T160000Z 
</pre>
<p>JSON:</p>
<pre>{"Access":[ 
   {"Node":"riak_moss@127.0.0.1", 
    "Samples":[{"StartTime":"20120315T150000Z", 
                "EndTime":"20120315T152931Z", 
                "KeyWrite":{"BytesIn":32505856,"Count":1}, 
                "KeyRead":{"BytesOut":32505856,"Count":1}, 
                "BucketRead":{"BytesOut":3633,"Count":5}}]}, 
   {"Errors":[]}], 
"Storage":"not_requested"} 
</pre>
<p>XML:</p>
<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt; 
&lt;Usage&gt;
   &lt;Access&gt; 
      &lt;Node name="riak_moss@127.0.0.1"&gt; 
         &lt;Sample StartTime="20120315T150000Z" EndTime="20120315T152931Z"&gt; 
            &lt;Operation type="KeyWrite"&gt; 
               &lt;BytesIn&gt;32505856&lt;/BytesIn&gt; 
               &lt;Count&gt;1&lt;/Count&gt; 
            &lt;/Operation&gt; 
            &lt;Operation type="KeyRead"&gt; 
               &lt;BytesOut&gt;32505856&lt;/BytesOut&gt; 
               &lt;Count&gt;1&lt;/Count&gt; 
            &lt;/Operation&gt; 
            &lt;Operation type="BucketRead"&gt; 
               &lt;BytesOut&gt;3633&lt;/BytesOut&gt; 
               &lt;Count&gt;5&lt;/Count&gt; 
            &lt;/Operation&gt; 
         &lt;/Sample&gt; 
      &lt;/Node&gt; 
      &lt;Errors/&gt; 
   &lt;/Access&gt; 
   &lt;Storage&gt;not_requested&lt;/Storage&gt; 
&lt;/Usage&gt;
</pre>
<p>The behavior of the resource when the <tt>s</tt> or <tt>e</tt> parameter is omitted may change, but is currently:</p>
<ul>
<li>omitting <tt>e</tt> will cause the resource to return only data for the slice in which <tt>s</tt> falls</li>
<li>omitting <tt>s</tt> will cause the resource to return data for all slices from <tt>e</tt> through the current time</li>
</ul>
<p>Or, more simply, the default <tt>s</tt> is <em>now</em> and the default <tt>e</tt> is equal to <tt>s</tt>.</p>
<h3>Time Span Limit</h3>
<p>To prevent excessive time and memory from being consumed accidentally, the amount of time that may be retrieved in any request is limited.</p>
<p>The limit is configured by the <tt>riak_moss</tt> application environment variable <tt>usage_request_limit</tt>. The value is expressed as an integer number of archive intervals (see <a href="https://help.basho.com/entries/21560857-usage-and-billing-data">Usage and Billing</a> for a description of archive intervals).</p>
<p>The default value is <tt>744</tt>, which is 31 days at the default archive interval of one hour.</p>
<h2 id="the-magic-usage-bucket">The Magic <tt>usage</tt> Bucket</h2>
<p>If you would prefer to use <tt>s3cmd</tt> or another S3 library to fetch access stats, you may do so by referencing objects in the global <tt>usage</tt> bucket. The format for objects in the usage bucket is:</p>
<pre>s3://usage/UserKeyId/Options/StartTime/EndTime
</pre>
<p>Or, if <tt>/</tt> is automatically quoted (<tt>%2f</tt>) by your client, the <tt>.</tt> character may be used (this is also nicer for s3cmd, since it will automatically choose a more useful name for the file it creates):</p>
<pre>s3://usage/UserKeyId.Options.StartTime.EndTime
</pre>
<p>That is, in the usage bucket, this is a sub-bucket named for the user's <tt>key_id</tt> (the <tt>UserKeyId</tt> part of the path).</p>
<p>Inside the user's bucket is a sub-bucket named for the contents and their representation (the <tt>Options</tt> part of the path). This portion should be:</p>
<ul>
<li><tt>aj</tt> to receive access statistics as JSON data</li>
<li><tt>ax</tt> to receive access statistics as XML data</li>
</ul>
<p>The next two portions of the path, <tt>StartTime</tt> and <tt>EndTime</tt> are the start and end times for the window to report, respectively. These take the same ISO8601 format that the <tt>s</tt> and <tt>e</tt> query parameters take in the other request method.</p>
<p>As an example, making the same request as the last example, for JSON-format access statistics between 2:00pm and 4:00pm GMT on January 30, 2012, looks like this:</p>
<pre>s3cmd get s3://usage/8NK4FH2SGKJJM8JIP2GU/aj/20120315T140000Z/20120315T160000Z
</pre>
<p><em>NOTE</em>: All objects in the <tt>usage</tt> bucket are read-only. <tt>PUT</tt> and <tt>DELETE</tt> requests will fail for them.</p>
<p><em>NOTE</em>: Regular users are only allowed to accss the statistics bucket for their own <tt>key_id</tt>. The admin user is allowed to access any stat bucket.</p>
<h2>Interpretting the Results</h2>
<p>Results of the access query are grouped by node. That is, within the access field of the result will be one entry for each Riak CS node that had data for the requested time span.</p>
<p>Each node entry will contain one or more "samples" for each time slice that the user accessed that Riak CS node. The sample will have a start time and end time describing what span the sample covers.</p>
<p>The other entries of each sample are the operations the user performed during the sampled time. Operation statistics are provided as rollups for each operation type. The rollup includes one or more of the following fields:</p>
<ul>
<li><tt>Count</tt> -- the number of times this operation was used successfully</li>
<li><tt>UserErrorCount</tt> -- the number of times this operation was used, but ended in a 400-499 response code</li>
<li><tt>SystemErrorCount</tt> -- the number of times this operation was used, but ended in a 500-599 response code</li>
<li><tt>BytesIn</tt> -- the number of bytes that were included in the request bodies of successful operations</li>
<li><tt>UserErrorBytesIn</tt> -- the number of bytes that were included in the request bodies of operations that ended in 400-499 response codes</li>
<li><tt>SystemErrorBytesIn</tt> -- the number of bytes that were included in the request bodies of operations that ended in 500-599 response codes</li>
<li><tt>BytesOut</tt> -- the number of bytes that were included in the response bodies of successful operations</li>
<li><tt>UserErrorBytesOut</tt> -- the number of bytes that were included in the response bodies of operations that ended in 400-499 response codes</li>
<li><tt>SystemErrorBytesOut</tt> -- the number of bytes that were included in the response bodies of operations that ended in 500-599 response codes</li>
<li><tt>BytesOutIncomplete</tt> -- the number of bytes that were sent in response bodies before the client disconnected, if there was more that could have been sent afterward (i.e. the byte count of partial downloads)</li>
</ul>
<p>It is important to note that accesses are only logged when the webmachine request finishes. This means that, for example, an upload started in one time slice but ended in another will only add to the "bytes in" field for the time slice in which in finished, rather than splitting the statistics between the slices in which they actually happened.</p>
<h3>Operation Types</h3>
<p>The operation types that are currently tracked are:</p>
<ul>
<li>
<p><tt>ListBuckets</tt> -- listing a user's buckets (<tt>GET /</tt>)</p>
</li>
<li>
<p><tt>UsageRead</tt> -- reading a user's usage statistics (<tt>GET /usage/user/*</tt>)</p>
</li>
<li>
<p><tt>BucketRead</tt> -- listing the files in a bucket (<tt>GET /bucket</tt>)</p>
</li>
<li>
<p><tt>BucketStat</tt> -- checking the existence of a bucket (<tt>HEAD /bucket</tt>)</p>
</li>
<li>
<p><tt>BucketCreate</tt> -- creating a bucket (<tt>PUT /bucket</tt>)</p>
</li>
<li>
<p><tt>BucketDelete</tt> -- deleting a bucket (<tt>DELETE /bucket</tt>)</p>
</li>
<li>
<p><tt>BucketUnknown</tt> -- unknown bucket operation (<tt>?? /bucket</tt>)</p>
</li>
<li>
<p><tt>BucketReadACL</tt> -- retrieving the ACL of a bucket (<tt>GET /bucket?acl</tt>)</p>
</li>
<li>
<p><tt>BucketStatACL</tt> -- checking the existence of a bucket (<tt>HEAD /bucket?acl</tt>)</p>
</li>
<li>
<p><tt>BucketWriteACL</tt> -- changing the ACL of a bucket (<tt>PUT /bucket?acl</tt>)</p>
</li>
<li>
<p><tt>BucketUnknownACL</tt> -- unknown bucket ACL operation (<tt>?? /bucket?acl</tt>)</p>
</li>
<li>
<p><tt>KeyRead</tt> -- fetching a object (<tt>GET /bucket/key</tt>)</p>
</li>
<li>
<p><tt>KeyStat</tt> -- checking the existence of a object (<tt>HEAD /bucket/key</tt>)</p>
</li>
<li>
<p><tt>KeyWrite</tt> -- uploading a object (<tt>PUT /bucket/key</tt>)</p>
</li>
<li>
<p><tt>KeyDelete</tt> -- deleting a object (<tt>DELETE /bucket/key</tt>)</p>
</li>
<li>
<p><tt>KeyUnknown</tt> -- unknown object operation (<tt>?? /bucket/key</tt>)</p>
</li>
<li>
<p><tt>KeyReadACL</tt> -- retrieving the ACL of a key (<tt>GET /bucket/key?acl</tt>)</p>
</li>
<li>
<p><tt>KeyStatACL</tt> -- checking the existence of a object (<tt>HEAD /bucket/key?acl</tt>)</p>
</li>
<li>
<p><tt>KeyWriteACL</tt> -- changing the ACL of a object (<tt>PUT /bucket/key?acl</tt>)</p>
</li>
<li>
<p><tt>KeyUnknownACL</tt> - unknown key ACL operation (<tt>?? /bucket/key?acl</tt>)</p>
</li>
<li>
<p><tt>UnknownGET</tt> -- a <tt>GET</tt> was issued on an unrecognized resource; likely means that the <tt>riak_moss_access_logger:operation/1</tt> function is out of date</p>
</li>
<li>
<p><tt>UnknownHEAD</tt> -- see <tt>UnknownGET</tt></p>
</li>
<li>
<p><tt>UnknownPUT</tt> -- see <tt>UnknownGET</tt></p>
</li>
<li>
<p><tt>UnknownPOST</tt> -- see <tt>UnknownGET</tt></p>
</li>
<li>
<p><tt>UnknownDELETE</tt> -- see <tt>UnknownGET</tt></p>
</li>
</ul>
<h3>Lookup Errors</h3>
<p>In addition to the node entries in the access results, there is also an entry for errors that Riak CS encountered while fetching access archives. The errors list is very similar to the samples of a node list: each entry will contain the start and end times of the period, as well as the "reason" the lookup failed.</p>
<p>For example, if the Riak lookups that Riak CS uses end in timeout instead of success, the result including an errors list might look like the following.</p>
<p>JSON:</p>
<pre>{"Access":[ 
   {"Errors":[ 
      {"StartTime":"20120315T160000Z", 
       "EndTime":"20120315T170000Z", 
       "Reason":"timeout"}]}], 
"Storage":"not_requested"} 
</pre>
<p>XML:</p>
<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt; 
&lt;Usage&gt; 
   &lt;Access&gt; 
      &lt;Errors&gt; 
         &lt;Sample StartTime="20120315T160000Z" EndTime="20120315T170000Z"&gt;
            &lt;Reason&gt;timeout&lt;/Reason&gt; 
         &lt;/Sample&gt; 
      &lt;/Errors&gt; 
   &lt;/Access&gt; 
   &lt;Storage&gt;not_requested&lt;/Storage&gt; 
&lt;/Usage&gt;
</pre>