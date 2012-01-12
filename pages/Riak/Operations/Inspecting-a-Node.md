`riak-admin status` provides data for examining a node's current running condition. It's output is broken down into six general categories below.

<div id="toc"/></div>

## One-minute
One-minute Counters are data points delineating the number of times a particular activity has occurred within the last minute on this particular node.

Sample One-minute Counters:

<table>
	<tr>
		<td>node_puts</td><td>Number of PUTs coordinated by this node, including PUTs to non-local vnodes</td>
	</tr>
	<tr>
		<td>node_gets</td><td>Number of GETs coordinated by this node, including GETs to non-local vnodes</td>
	</tr>
	<tr>
		<td>vnode_puts</td><td>Number of PUTs coordinated by vnodes local to this node</td>
	</tr>
	<tr>
		<td>vnode gets</td><td>Number of GETs coordinated by vnodes local to this node</td>
	</tr>
	<tr>
		<td>read_repairs</td><td>Number of Read Repairs this node has coordinated</td>
	</tr>
</table>


## FSM_Time
FSM_Time Counters represent the amount of time in microseconds required to traverse the GET or PUT Finite State Machine code, offering a picture of general node health. From your application's perspective, FSM_Time effectively represents experienced latency. Mean, Median, and 95th-, 99th-, and 100th-percentile (Max) counters are displayed. These are one-minute stats.

Sample FSM_Time Counters:

<table>
	<tr>
		<td>node_get_fsm_time_mean</td><td>Mean time between when a GET request is received by this node and when a reply is sent to the client</td>
	</tr>
	<tr>
		<td>node_put_fsm_time_mean</td><td>Mean time between when a PUT request is received by this node and when a reply is sent to the client</td>
	</tr>
</table>


## GET_FSM_Siblings
GET_FSM_Sibling Stats offer a count of the number of siblings encountered by this node 
on the occasion of a GET request. These are one-minute stats.

Sample GET_FSM_Sibling Counters:

<table>
	<tr><td>node_get_fsm_siblings_mean</td><td>Mean number of siblings encountered of all GETs by this node within the last minute</td>
	</tr>	
</table>


## GET_FSM_Objsize
GET_FSM_Objsize is a window on the sizes of objects flowing through this node's GET_FSM. The size of an object is obtained by summing the length of the bucket name, key, the serialized vector clock, the value, and the serialized metadata of each sibling. GET_FSM_Objsize and GET_FSM_Siblings are inextricably linked. These are one-minute stats.

Sample GET_FSM_Objsize Counters:

<table>
	<tr>
		<td>node_get_fsm_objsize_mean</td><td>Mean size of an object on this node within the last minute</td>
	</tr>	
</table>


## Totals
Total Counters are data points that represent the total number of times a particular activity has occurred since this node was started.

Sample Total Counters:

<table>
	<tr>
		<td>vnode_gets_total</td><td>Number of GETs coordinated by local vnodes since node startup</td>
	</tr>
	<tr>
		<td>vnode_puts_total</td><td>Number of PUTS coordinated by local vnodes since node startup</td>
	</tr>
	<tr>
		<td>node_gets_total</td><td>Number of GETs coordinated by this node since startup, including GETs to non-local vnodes</td>
	</tr>
	<tr>
		<td>node_puts_total</td><td>Number of PUTs coordinated by this node since startup, including PUTs to non-local vnodes</td>
	</tr>
	<tr>
		<td>read_repairs_total</td><td>Number of Read Repairs this node has coordinated since startup</td>
	</tr>
	<tr>
		<td>coord_redirs_total</td><td>Number of requests this node has redirected to other nodes for coordination since startup</td>
	</tr>
</table>


## Miscellaneous Information
Miscellaneous Information stats are data points that provide details particular to this node.

Sample Miscellaneous Information Stats:

<table>
	<tr>
		<td>nodename</td><td>The name this node uses to identify itself to the ring</td>
	</tr>
	<tr>
		<td>ring_num_partitions</td><td>The configured number of partitions in the ring</td>
	</tr>
	<tr>
		<td>connected_nodes</td><td>A list of the nodes that this node is aware of at this time</td>
	</tr>
</table>


