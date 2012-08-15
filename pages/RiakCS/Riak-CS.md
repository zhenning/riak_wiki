# Welcome to the Riak CS Wiki

![Riak Logo](images/riaklogo.png)


Riak CS is multi‐tenant cloud storage software for public and private clouds. Built on Basho's distributed database Riak, Riak CS is designed to provide simple, available, distributed cloud storage at any scale. Riak CS is S3‐API compatible and supports per‐tenant reporting for billing and metering use cases.

<!--<div id ="new_nav">
	<ul id="top_list">
		<li><a href="Installation.html">Download and Install</a></li>
		<li><a href="The-Riak-Fast-Track.html">Riak Fast Track</a></li>
		<li><a href="Community.html">Community</a></li>
		<li><a href="Client-Libraries.html">Client Libraries</a></li>
	</ul>
</div>-->

Notable Features of Riak CS
---------------------------

<table style="width: 100%; border-spacing: 0px;">
<tbody>
<tr align="left" valign="top">
<td style="padding: 15px; margin: 15px; border-width: 1px 0 1px 0; border-style: solid;">Amazon S3‐API Compatibility</td>
<td style="padding: 15px; margin: 15px; border-width: 1px 0 1px 0; border-style: solid;">
<p>Riak CS has a built‐in S3 interface with support for S3&nbsp;ACL so you can use existing S3 tools and frameworks&nbsp;or import and extract Amazon data. The HTTP REST&nbsp;API supports service, bucket and object‐level&nbsp;operations to easily store and retrieve data.&nbsp;</p>
</td>
</tr>
<tr align="left" valign="top">
<td style="padding: 15px; margin: 15px; border-width: 0 0 1px 0; border-style: solid;">Per‐Tenant Visibility</td>
<td style="padding: 15px; margin: 15px; border-width: 0 0 1px 0; border-style: solid;">
<p>With the Riak CS Reporting API, you get per‐tenant&nbsp;usage data and statistics on network I/O. This&nbsp;reporting functionality supports use cases including&nbsp;accounting, subscription, chargebacks, plugins with&nbsp;billing systems or efficient multi‐department&nbsp;utilization.</p>
</td>
</tr>
<tr align="left" valign="top">
<td style="padding: 15px; margin: 15px; border-width: 0 0 1px 0; border-style: solid;">
<p>Supports Objects of Arbitrary&nbsp;Content Type Up to 5GB, Plus&nbsp;Metadata</p>
</td>
<td style="padding: 15px; margin: 15px; border-width: 0 0 1px 0; border-style: solid;">
<p>Store images, text, video, documents, database&nbsp;backups, software binaries and other content up to&nbsp;5GB as a single, easily retrievable object.&nbsp;&nbsp;Riak CS also&nbsp;supports standard Amazon metadata headers.</p>
</td>
</tr>
<tr align="left" valign="top">
<td style="padding: 15px; margin: 15px; border-width: 0 0 1px 0; border-style: solid;">
<p>Pluggable&nbsp;Authentication / Authorization for&nbsp;Integration with Existing&nbsp;Infrastructure</p>
</td>
<td style="padding: 15px; margin: 15px; border-width: 0 0 1px 0; border-style: solid;">
<p>Riak CS provides an extensible authentication system,&nbsp;enabling integration with existing directory services&nbsp;(LDAP, ActiveDirectory, NIS, PAM).</p>
</td>
</tr>
</tbody>
</table>
<blockquote>
<p>Note: Not all Riak EDS features are currently compatible with Riak CS,&nbsp;including multi-datacenter replication.</p>
</blockquote>