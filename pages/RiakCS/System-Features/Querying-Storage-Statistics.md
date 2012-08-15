<p>Storage statistics are tracked on a per-user basis, as rollups for slices of time. Querying these statistics is done via the <tt>/usage/$USER_KEY_ID</tt> resource.</p>
<p><em>Note</em>: Storage statistics are not calculated by default. Please read <a href="https://help.basho.com/entries/21560857-usage-and-billing-data">Usage and Billing</a> for details about how to enable storage calculation archival.</p>
<p>The basics of querying storage statistics, including the URL used and the parameters for specifying the time slice, are the same as they are for <a href="https://help.basho.com/entries/21552263-querying-access-statistics">Querying Access Statistics</a>. Please refer to the descriptions there for more details.</p>
<h2>Enable Storage Results</h2>
<p>The usage HTTP resource provides both access and storage statistics. Since each of these queries can be taxing in its own right, they are both omitted from the result by default:</p>
<pre>curl http://localhost:8080/usage/8NK4FH2SGKJJM8JIP2GU 
</pre>
<p>JSON:</p>
<pre>{"Access":"not_requested","Storage":"not_requested"} 
</pre>
<p>XML (reformatted for easy reading):</p>
<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt; 
&lt;Usage&gt; 
   &lt;Access&gt;not_requested&lt;/Access&gt; 
   &lt;Storage&gt;not_requested&lt;/Storage&gt; 
&lt;/Usage&gt; 
</pre>
<p>To request that storage results be included, pass the query parameter <tt>b</tt> to the resource (any true-ish value will work, including just the bare <tt>b</tt>, <tt>t</tt>, <tt>true</tt>, <tt>1</tt>, <tt>y</tt>, and <tt>yes</tt>):</p>
<pre>curl http://localhost:8080/usage/8NK4FH2SGKJJM8JIP2GU?b
</pre>
<p>JSON (reformatted for easy reading):</p>
<pre>{"Access":"not_requested", 
 "Storage":[{"Errors":[]}]}
</pre>
<p>XML (reformatted for easy reading):</p>
<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt; 
&lt;Usage&gt; 
   &lt;Access&gt;not_requested&lt;/Access&gt; 
   &lt;Storage&gt;&lt;Errors/&gt;&lt;/Storage&gt; 
&lt;/Usage&gt;
</pre>
<p>There are no statistics included in this report because the default time span is <em>now</em>, which is not available in the archives.</p>
<h3>S3 Object-style Access</h3>
<p>As described in <a href="https://help.basho.com/entries/21552263-querying-access-statistics">Querying Access Statistics</a>, these statistics are also available as S3 objects. To add storage statistics to the result, add the character <tt>b</tt> to the <tt>Options</tt> portion of the object's path. For example, the following command would produce storage statistics in XML format:</p>
<pre>s3cmd get s3://usage/8NK4FH2SGKJJM8JIP2GU/bx/20120315T140000Z/20120315T160000Z
</pre>
<p>You may also pass both <tt>b</tt> and <tt>a</tt> as <tt>Options</tt> to fetch both types of stats, as in:</p>
<pre>s3cmd get s3://usage/8NK4FH2SGKJJM8JIP2GU/abx/20120315T140000Z/20120315T160000Z
</pre>
<h2>Interpreting the Results</h2>
<p>The result of the storage query is one or more "samples" for each time slice in which storage was calculated for the user. The sample will have a start time and end time describing what span the sample covers.</p>
<p>The other entries of each sample are the buckets the user owned during the sampled time. Bucket statistics are provided as rollups including each of the following fields:</p>
<ul>
<li><tt>Objects</tt> -- the number of active (not deleted, and not incompletely uploaded) files in the bucket</li>
<li><tt>Bytes</tt> -- the total of number of bytes stored in the files of the bucket</li>
</ul>
<p>For example, a user that owns two buckets, <tt>foo</tt> and <tt>bar</tt>, where <tt>foo</tt> contains one 32MB file, and <tt>bar</tt> contains 4 32MB files, would have a sample similar to the following.</p>
<p>JSON (reformatted for easy reading):</p>
<pre>{"Access":"not_requested", 
 "Storage":[{"StartTime":"20120316T123318Z", 
             "EndTime":"20120316T123319Z", 
             "foo":{"Objects":1,"Bytes":32505856}, 
             "bar":{"Objects":4,"Bytes":130023424}}, 
            {"Errors":[]}]} 
</pre>
<p>XML (reformatted for easy reading):</p>
<pre>&lt;?xml version="1.0" encoding="UTF-8"?&gt; 
&lt;Usage&gt; 
   &lt;Access&gt;not_requested&lt;/Access&gt; 
   &lt;Storage&gt; 
      &lt;Sample StartTime="20120316T123318Z" EndTime="20120316T123319Z"&gt; 
         &lt;Bucket name="hooray"&gt; 
            &lt;Objects&gt;1&lt;/Objects&gt; 
            &lt;Bytes&gt;32505856&lt;/Bytes&gt; 
         &lt;/Bucket&gt; 
         &lt;Bucket name="foo6"&gt; 
            &lt;Objects&gt;4&lt;/Objects&gt; 
            &lt;Bytes&gt;130023424&lt;/Bytes&gt; 
         &lt;/Bucket&gt; 
      &lt;/Sample&gt; 
      &lt;Errors/&gt; 
   &lt;/Storage&gt; 
&lt;/Usage&gt;
</pre>