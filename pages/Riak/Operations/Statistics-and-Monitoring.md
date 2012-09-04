# Statistics and Monitoring

## Statistics from Riak
Riak provides data related to current operating status which includes statistics in the form of counters and histograms. These statistics are made available via HTTP and the [`/stats`](https://wiki.basho.com/HTTP-Status.html) endpoint, or through the [`riak-admin status`](http://wiki.basho.com/Inspecting-a-Node.html#riak-admin-status) command.

Some of the most commonly monitored and gathered statistics are presented here along with numerous solutions for monitoring and gathering statistics which our customers and community report successfully using for Riak cluster environments. You can learn more about the specific Riak statistics provided in the [[Inspecting a Node]] documentation.

### Counters
Riak provides counters which default to tracking values for one minute or for the runtime duration of the node.

#### Gets and Puts
Counters representing the number of node and vnode GET/PUT operations are provided for both one minute and total since node startup. These counters are commonly graphed over time for trend analysis, capacity planning, and so forth.

* `node_gets`: Number of GETs coordinated by this node, including GETs to non-local vnodes on this node within the last minute
* `node_gets_total`: Number of GETs coordinated by this node since startup, including GETs to non-local vnodes
* `node_puts`: Number of PUTs coordinated by this node, including PUTs to non-local vnodes on this node within the last minut
* `node_puts_total`: Number of PUTs coordinated by this node since startup, including PUTs to non-local vnodes
* `vnode_gets`: Number of GET operations coordinated by vnodes on this node within the last minute
* `vnode_gets_total`: Number of GETs coordinated by local vnodes since node startup
* `vnode_puts_total`: Number of PUTS coordinated by local vnodes since node startup

#### Read Repairs
Counters representing the number of read repair operations are provided for both one minute and total since node startup. These counters are commonly graphed and monitored for abnormally high totals, which can be indicative of an issue.

* `read_repairs`: Number of read repair operations this this node has coordinated in the last minute
* `read_repairs_total`: Number of read repair operations this this node has coordinated since node was started

#### Coordinated Redirection
Counters representing the number of coordinated node redirection operations are provided in total since node startup.

* `coord_redirs_total`: Number of requests this node has redirected to other nodes for coordination since startup
### Histograms
Riak provides histograms with a default window of 60 seconds for mean, median, 95th percentile, 99th percentile, and 100th percentile on a range of operations such as the following:

#### Finite State Machine Time
Riak exposes finite state machine (FSM) time counters (`node_get_fsm_time_*` & `node_put_fsm_time_*`) represent the amount of time in microseconds required to traverse the GET or PUT FSM code, offering a picture of general node health.

#### GET FSM Object Size
The GET FSM Object Size (`node_get_fsm_objsize_*`) is a window on the sizes of objects flowing through this node's GET_FSM. The size of an object is obtained by summing the length of the bucket name, key, the serialized vector clock, the value, and the serialized metadata of each sibling.

#### GET FSM Siblings
The GET FSM Sibling statistics (`node_get_fsm_siblings_*`) provide one minute histograms representing the number of siblings encountered by this node on the occasion of a GET request.

## Statistics and Monitoring Tools
There are many open source, self-hosted, and service based solutions for the aggregation and analysis of statistics and log data for the purposes of monitoring, alerting, and trend analysis on a Riak cluster. Some solutions provide Riak specific modules or plugins as noted.

The following are solutions which customers and community members have reported success with when used for monitoring the operational status of their Riak clusters. Community and open source projects are presented along with commercial and hosted services.

### Community and Open Source Tools

#### Riaknostic
[Riaknostic](http://riaknostic.basho.com) is a growing suite of diagnostic checks that can be run against your Riak node to discover common problems and recommend how to resolve them. These checks are derived from the experience of the Basho Client Services Team as well as numerous public discussions on the mailing list, IRC room, and other online media.

Riaknostic integrates into the `riak-admin` command, providing a a `diag` subcommand, and is a great first step in the process diagnosing and troubleshooting issues on Riak nodes.

#### Riak Control
[Riak Control](http://wiki.basho.com/Riak-Control.html) is Basho's REST-driven user-interface for managing Riak clusters. It is designed to give you quick insight into the health of your cluster and allow for easy management of nodes.

While Riak Control does not currently offer specific monitoring and statistics aggregation or analysis functionality, it does offer features which provide immediate insight into overall cluster health, node status, and handoff operations.

#### collectd
[collectd](http://collectd.org) gathers statistics about the system it is running on and stores them. The statistics can then be used typically through generation into graphs, to find current performance bottlenecks, predict system load, and analyze trends.

#### Ganglia
[Ganglia](http://ganglia.info) is a monitoring system specifically designed for large, high-performance groups of computers, such as clusters and grids. Customers and community members using Riak have reported success in using Ganglia to monitor Riak clusters.

A [Riak Ganglia module](https://github.com/jnewland/gmond_python_modules/tree/master/riak/) for collecting statistics from the Riak HTTP `/stats` endpoint is also available.

#### Nagios
[Nagios](http://www.nagios.org) is a monitoring and alerting solution that can provide information on the status Riak cluster nodes, in addition to various types of alerting when particular events occur. Nagios also offers logging and reporting of events, and can be used for identifying trends and capacity planning.

A collection of [reusable Riak specific scripts](https://github.com/basho/riak_nagios) are available to the community for use with Nagios.

#### Riemann
[Riemann](http://aphyr.github.com/riemann/) uses a powerful stream processing language to aggregate events from client agents running on Riak nodes, and can help track trends or report on events as they occur. Statistics can be gathered from your nodes and forwarded to a solution such as Graphite for producing related graphs.

A [Riemann Tools](https://github.com/aphyr/riemann.git) project consisting of small programs for sending data to Riemann provides a module specifically designed to read Riak statistics.

#### OpenTSDB
[OpenTSDB](http://opentsdb.net) is a distributed, scalable Time Series Database (TSDB) used to store, index, and serve metrics from various sources. It can collect data a large scale and also graph these metrics on the fly.

A [Riak collector for OpenTSDB](https://github.com/stumbleupon/tcollector/blob/master/collectors/0/riak.py) is available as part of the [tcollector framework](https://github.com/stumbleupon/tcollector).

### Commercial  and Hosted Service Tools
The following are some commercial tools which Basho customers have reported successfully using for statistics gathering and monitoring within their Riak clusters.

#### Circonus
[Circonus](http://circonus.com) provides organization wide monitoring, trend analysis, alerting, notifications, and dashboards. It can been used to provide trend analysis and help with troubleshooting and capacity planning in a Riak cluster environment.

<!--
Need more information on this one...
#### Scout
[Scout](https://scoutapp.com)
-->

#### Splunk
[Splunk](http://www.splunk.com) is available as downloadable software or as a service, and provides tools for visualization of machine generated data, such as log files and it can be connected to Riak's HTTP statistics `/stats` endpoint.

Splunk can be used to aggregate all Riak cluster node operational log files, including operating system and Riak specific logs and Riak statistics data. These data are then available for real time graphing, search, and other visualization ideal for troubleshooting complex issues and spotting trends.

## Summary
Riak exposes numerous forms of vital statistic information which can be aggregated, monitored, analyzed, graphed, and reported on in a variety of ways using numerous open source and commercial solutions.

If you use a solution not listed here with Riak and would like to include it (or would otherwise like to update the information on this page), feel free to fork the wiki, add it in the appropriate section, and send a pull request to the [Riak wiki project](https://github.com/basho/riak_wiki).

## References

* [Basho Wiki](http://wiki.basho.com)
* [Inspecting a Node](http://wiki.basho.com/Inspecting-a-Node.html)
* [Riaknostic](http://riaknostic.basho.com)
* [Riak Control](http://wiki.basho.com/Riak-Control.html)
* [collectd](http://collectd.org)
* [Ganglia](http://ganglia.info)
* [Nagio](http://www.nagios.org)
* [Riemann](http://aphyr.github.com/riemann/)
* [Riemann Github](https://github.com/aphyr/riemann)
* [OpenTSDB](http://opentsdb.net)
* [tcollector project](https://github.com/stumbleupon/tcollector)
* [tcollector Riak module](https://github.com/stumbleupon/tcollector/blob/master/collectors/0/riak.py)
* [Folsom Backed Stats Riak 1.2](http://basho.com/blog/technical/2012/07/02/folsom-backed-stats-riak-1-2/)
* [Circonus](http://circonus.com)
* [Splunk](http://www.splunk.com)
* [Riak Wiki Github](https://github.com/basho/riak_wiki)
