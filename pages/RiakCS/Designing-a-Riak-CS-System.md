<div class="info"><div class="title">Riak CS Only</div>This documentation applies only to Riak Cloud Storage, a commercial extension to <a href="http://wiki.basho.com/Riak.html">Riak</a>. To talk to us about using Riak CS, <a href="http://info.basho.com/Wiki_Contact_RiakCS.html" target="_blank">let us know</a>.</div>

# Designing a Riak CS System

## Best Practices

You should plan on having one Riak node for every Riak CS node in your system.

Riak and Riak CS nodes can be run on separate physical machines, but in many cases it is preferable to run one Riak and one Riak CS node on the same physical machine. Assuming the single physical machine has sufficient capacity to meet the needs of both a Riak and a Riak CS node you will typically see better performance due to reduced network latency.