# Monitoring and Metrics
Riak CS provides operational statistics which can be useful for monitoring through the Folsom statistics library, and initial probes for analysis of the running system with [[DTrace|http://dtrace.org/blogs/about/]].

## Operational Statistics
Much like Riak, Riak CS exposes statistics on critical operations which are commonly used for monitoring, alerting, and trend analysis. These statistics can be accessed through HTTP requests to the following resource:

```
/riak-cs/stats
```

<!--
The results will include one-minute counters for the following statistics:

* **block_get**: TODO - brief description for this statistic
* **block_put**: TODO - brief description for this statistic
* **block_delete**: TODO - brief description for this statistic
* **service_get_buckets**: TODO - brief description for this statistic
* **bucket_list_keys**: TODO - brief description for this statistic
* **bucket_create**: TODO - brief description for this statistic
* **bucket_delete**: TODO - brief description for this statistic
* **bucket_get_acl**: TODO - brief description for this statistic
* **bucket_put_acl**: TODO - brief description for this statistic
* **object_get**: TODO - brief description for this statistic
* **object_put**: TODO - brief description for this statistic
* **object_head**: TODO - brief description for this statistic
* **object_delete**: TODO - brief description for this statistic
* **object_get_acl**: TODO - brief description for this statistic
* **object_put_acl**: TODO - brief description for this statistic

-->

## DTrace Probes
Riak CS is built with some probes for use with [[DTrace|http://dtrace.org/blogs/about/]] to inspect certain operations in the live system, which can be helpful to diagnose issues.

### Usage Examples
The following are examples of using DTrace for inspecting various components of a running Riak CS installation.

**Trace user object requests**:

    dtrace -qn 'erlang*:::user_trace* /arg2 == 703/ {printf("pid %s: mod %s op %s: user %s bucket/file %s\n", copyinstr(arg0), copyinstr(arg6), copyinstr(arg7), copyinstr(arg8), copyinstr(arg9));}'

**Trace webmachine resource execution**:

    dtrace -qn 'erlang*:::user_trace* /arg2 == 705/ {printf("pid %s: %s:%s\n", copyinstr(arg0), copyinstr(arg6), copyinstr(arg7));}'

<div class="info"><div class="title">DTrace Support</div> Work on packaging of Riak CS for SmartOS and other operating systems with DTrace support is ongoing with a goal to provide enhanced ability to diagnose low level issues in instances of Riak CS running on such operating systems.</div>
