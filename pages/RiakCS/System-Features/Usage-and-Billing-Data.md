<h2>Access Statistics</h2>
<p>Access stats are tracked on a per-user basis, as rollups for slices of time. They are stored in the same Riak cluster as other Riak CS data, in the <tt>moss.access</tt> bucket.</p>
<p>For information about querying access statistics, please read <a href="https://help.basho.com/entries/21552263-querying-access-statistics">Querying Access Statistics</a>.</p>
<h3>High Level</h3>
<ol>
<li>Riak CS determines who, if anyone, should be billed for each access.</li>
<li>Riak CS send this and some statistical information about the accesses to an aggregation subsystem.</li>
<li>The aggregation subsystem periodically sends its accumulated log to be archived.</li>
<li>The archival subsystem sums all recorded accesses for each user and stores a record for each user for the time slice.</li>
</ol>
<p>Log retrieval is then simply making a request to Riak for all slice objects for a user in a time period.</p>
<p>No access data will be logged unless the user for the access is known.</p>
<h4>Tracked Statisitics</h4>
<p>With nothing more than setting the user for the request, several statistics are logged automatically:</p>
<ul>
<li><tt>Count</tt> -- the number of times this operation was used; each request counts as one (1)</li>
<li><tt>BytesIn</tt> -- the number of bytes that were included in the request body</li>
<li><tt>BytesOut</tt> -- the number of bytes that were sent in the response body</li>
</ul>
<p>For successful requests, each of these stats is logged under the name given. For unsuccessful requests, they are logged under this name with a prefix of either <tt>SystemError</tt>, for requests that end in response codes 500+, or <tt>UserError</tt>, for requests that end in response codes 400-499. For example, if a user tries to download a nonexistent file, it will be logged under <tt>UserErrorCount</tt> with the bytes of the message logged under <tt>UserErrorBytesOut</tt>.</p>
<p>These three metrics are logged for each operation separately. The access logger determines the operation type by comparing the method, resource module, and path to a known table. For example, it knows that a <tt>GET</tt> on the <em>key</em> module with the <tt>acl</tt> query parameter in the path is a <tt>KeyReadACL</tt> operation. A <tt>PUT</tt> to the same resource without the <tt>acl</tt> query parameter is a <tt>KeyWrite</tt> operation. See&nbsp;<a href="https://help.basho.com/entries/21552263-querying-access-statistics">Querying Access Statistics</a>&nbsp;for a list of all operation types.</p>
<h3>Log Accumulation</h3>
<p>As resources finish their processing, the access logger module is called by Webmachine to log the access. This module implements a server that finds all of the access notes in the request's log data and stores them until the current interval ends.</p>
<p>When the current interval ends, the access module transfers ownership of its accumlated data to the archiver module. The logger module then resets for logging the next slice's accesses.</p>
<h4>Interval Duration</h4>
<p>The length of the log flushing interval is configured by the application environment variable <tt>access_log_flush_factor</tt>. The value is expressed as an integer divisor of the <tt>access_archive_period</tt> setting. That is, if <tt>access_log_flush_factor</tt> is 5, and <tt>access_archive_period</tt> is 3600 (== 1 hour) seconds, the log will be flushed every 720 seconds (== 12 minutes), which is 5 times per archive period.</p>
<p>The value of <tt>access_log_flush_factor</tt> must be an integer factor of <tt>access_archive_period</tt>. If the factor does not divide the period evenly, an error will be printed in the log, and the Riak CS node will refuse to start.</p>
<p>The default value for <tt>access_log_flush_factor</tt> is 1 (once per archive period). These settings may be manipulated in the Riak CS <tt>app.config</tt> file, normally located at <tt>/etc/riak-cs/app.config</tt>.</p>
<h4>Log Size Trigger for Archival</h4>
<p>Archival of the access log will also be triggered if the number of records waiting to be archived reaches a certain configured level. When the threshold is reached, all accumulated records are transfered to the archiver, which writes out a sample with <em>now</em> as the end-time. Accumulation is then restarted with <em>now</em> as the start-time, and will continue until either the end of the time interval, or until the log threshold is reached again.</p>
<p>This level is configured by the application environment variable <tt>access_log_flush_size</tt>. Its default value is <tt>1000000</tt> (one million).</p>
<h4>Backlog Caveat</h4>
<p>If the logger finds itself so far behind that it would need to schedule its next archival in the past (that is, after sending a log accumulation for interval N to the archiver, it finds that the end of interval N+1 has already passed), the logger will drop the backlog in its message box by exiting and allowing its supervisor process to restart it. Just before exiting, it will print an error message describing how far behind it was:</p>
<pre>09:56:02.584 [error] Access logger is running 302 seconds behind, skipping 0 log messages to catch up
</pre>
<p>With the default one-hour archive period, this case will only be encountered when the logger is an entire hour behind. This behavior is meant as a safety valve to prevent that hour lag from growing due to memory pressure from the logger processes's message queue.</p>
<h4>Manually Triggering Archival</h4>
<p>When taking a machine out of service, it may be desirable to trigger log archival before the end of the interval. To do so, use the <tt>riak-cs-access</tt> script with the command <tt>flush</tt>. It should be installed on the same path as the <tt>riak-cs</tt> script. For most OS distributions this will be at <tt>/usr/local/sbin</tt>.</p>
<p>By default, the script will wait up to 50 seconds for the logger to acknowledge that it has passed its accumulation to the archiver and another 50 seconds for the archiver to acknowledge that it has finished archiving all accumulations it has received. To wait longer, use the <tt>-w</tt> parameter on the command line with an integer number of 5-second intervals to wait. That is, to wait for 100 seconds for each phase, use:</p>
<pre>riak-cs-access flush -w 20
</pre>
<h3>Archive Retrieval</h3>
<p>When a request is recieved for a user's access stats over some time period, the objects for all intervals in that time period must be retrieved.</p>
<p>It is important to note that the archival process does not attempt a <em>read/modify/write</em> cycle when writing a slice record. The <tt>moss.access</tt> bucket should have the <tt>allow_mult=true</tt> flag set, and so multiple Riak CS nodes writing the same slice record for the same user create siblings. Riak CS attempts to check and set the <tt>allow_mult</tt> bucket property when it starts up, and will print a warning in the log about being "unable to configure" or "unable to verify" bucket settings if it fails.</p>
<p>Siblings should be handled at read time. Sibling resolution should be nothing more than a set union of all records. The HTTP resource serving the statistics expects to provide them on a node-accumulated basis, so it is <strong>important</strong> to set a <strong>unique Erlang node name for each Riak CS node</strong>.</p>
<h2>Storage Statistics</h2>
<p>Storage statistics are also tracked on a per-user basis, as rollups for slices of time. They are stored in the same Riak cluster as other Riak CS data, in the <tt>moss.storage</tt> bucket.</p>
<p>For detailed information about querying storage statistics, please read <a href="https://help.basho.com/entries/21560837-querying-storage-statistics">Querying Storage Statistics</a>.</p>
<h3>High Level</h3>
<ol>
<li>Storage is calculated for all users either:<ol>
<li>on a regular schedule</li>
<li>or when manually triggered with the <tt>riak-cs-storage</tt> script</li>
</ol></li>
<li>Each user's sum is stored in an object named for the timeslice in which the aggregation happened.</li>
<li>Sums are broken down by bucket.</li>
</ol>
<p>Log retrieval is then simply making a request to Riak for all slice objects for a user in a particular time period.</p>
<h4>Prerequisite: Code Paths for MapReduce</h4>
<p>The storage calculation system uses MapReduce to sum the files in a bucket. This means you must tell all of your Riak nodes where to find Riak CS's compiled files before calculating storage.</p>
<p>To ensure this is the case, please follow the direction given <a href="https://help.basho.com/entries/21444026-configuring-a-riak-cs-system-riak">here</a> for configuring Riak properly for a Riak CS system.</p>
<h3>Scheduling and Manual Triggering</h3>
<p>Triggering the storage calculation is a matter of setting up a regular schedule or manually starting the process via the <tt>riak-cs-storage</tt> script.</p>
<h4>Regular Schedules</h4>
<p>If you would like to have an Riak CS node calculate the storage used by every user at the same time (or times) each day, specify a schedule in that node's Riak CS <tt>app.config</tt> file.</p>
<p>In the <tt>riak_moss</tt> section of the file, add an entry for <tt>storage_schedule</tt> like:</p>
<pre>{storage_schedule, "0600"}
</pre>
<p>The time is given as a string of the form <tt>HHMM</tt>, representing the hour and minute GMT to start the calulation process. In this example, the node would start the storage calculation at 6am GMT every day.</p>
<p>To set up multiple times, specify a list in the schedule. For example, to schedule the calculation to happen at both 6am and 6pm, use:</p>
<pre>{storage_schedule, ["0600", "1800"]}
</pre>
<p><strong>Important</strong>: When using multiple times in a storage schedule, they must be scheduled for different archive periods (see details for <tt>storage_archive_period</tt> in the <em>Archival</em> section below). Extra scheduled times in the same archive period are skipped. This is intended to allow more than one Riak CS node to calculate storage statistics concurrently, as they will take notice of users already calculated by other nodes and skip them (see details in the Manual Triggering section about overriding this behavior).</p>
<p>By default no schedule is specified, so the storage calculation is never done automatically.</p>
<h4>Manual Triggering</h4>
<p>If you would rather trigger storage calculations manually, simply use the <tt>batch</tt> command in the <tt>riak-cs-storage</tt> script:</p>
<pre>$ riak-cs-storage batch Batch storage calculation started.
</pre>
<p>If there is already a calculation in progress, or if starting the calculation fails for some other reason, the script will print an error message saying so.</p>
<p>By default, a manually-triggered calculation run will skip users that have already been calculated in the current archive period (see the Archival section below for details about <tt>storage_archive_period</tt>). If you would rather calculate an additional sample for every user in this period, add the <tt>--recalc</tt> (or <tt>-r</tt> for short) option to the commandline:</p>
<pre>$ riak-cs-storage batch -r # force recalculation of every user
</pre>
<h4>Further Control</h4>
<p>In-process batch calculations can also be paused or canceled using the <tt>riak-cs-storage</tt> script.</p>
<p>To pause an in-process batch, use:</p>
<pre>$ riak-cs-storage pause The calculation was paused.
</pre>
<p>To resume a paused batch, use:</p>
<pre>$ riak-cs-storage resume The calculation was resumed.
</pre>
<p>To cancel an in-process batch (whether <em>paused</em> or <em>active</em>), use:</p>
<pre>$ riak-cs-storage cancel The calculation was canceled.
</pre>
<p>You can also retrieve the current state of the daemon by using the <tt>status</tt> command. The first line will indicate whether the daemon is <em>idle</em>, <em>active</em>, or <em>paused</em>, and it will be followed by further details based on progress. For example:</p>
<pre>A storage calculation is in progress 
Schedule: none defined 
Last run started at: 20120316T204135Z 
Current run started at: 20120316T204203Z 
Next run scheduled for: unknown/never 
Elapsed time of current run: 3 
Users completed in current run: 1 
Users left in current run: 4 
</pre>
<h3>Results</h3>
<p>When the node finishes calculating every user's storage, it will print a message to the log noting how long the entire process took:</p>
<pre>08:33:19.282 [info] Finished storage calculation in 1 seconds.
</pre>
<h3>Process</h3>
<p>The calculation process is coordinated by a long-lived finite state machine process that handles both the scheduling (if a schedule is defined) and the running of the process.</p>
<p>When a storage calculation starts, the first step is to obtain a list of known users of the system. Each user's record contains information about the buckets the user owns.</p>
<p>For each bucket that a user owns, a MapReduce query is run. The query's inputs are the list of the keys in the bucket (the input is <tt>BucketName</tt>, so the keys stay on the server). The query then has two phases: a map that produces tuples of the form <tt>{1, ByteSize(File)}</tt> (if <em>active</em>; nothing if <em>inactive</em>), and a reduce that sums those tuples element-wise. The result is one tuple whose first element is the number of files in the bucket, and whose second is the total number of bytes stored in that file.</p>
<p>Only one bucket is calculated at a time to prevent putting too much load on the Riak cluster. Only one user is calculated at a time as well to prevent too large of a temporary list on the Riak CS node.</p>
<p>Once the sum for each of the user's buckets is calculated, a record is written to the <tt>moss.storage</tt> Riak bucket.</p>
<h3>Archival</h3>
<p>Records written to the <tt>moss.storage</tt> bucket are very similar to records written to the <tt>moss.access</tt> bucket used for logging access statistics. The value is a JSON object with one field per bucket. The key is a combination of the user's <tt>key_id</tt> and the timestamp of the timeslice for which the calculation was run.</p>
<p>The period for storage archival is separate from the period for access archival. The storage archival period is configured by the application environment variable <tt>storage_archive_period</tt>. The default is 86400 (one day). This is because storage calculations are expected to be archived much less often than access logs, so specifying fewer possible keys to look up later reduces overhead at reporting time.</p>