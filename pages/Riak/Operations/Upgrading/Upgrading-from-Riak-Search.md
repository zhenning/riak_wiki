As of Riak 1.0, Riak Search has been integrated into Riak. A few simple
considerations must be kept in mind when transitioning.

<div id="toc"></div>

## Upgrading from Riak Search 0.14.2

Upgrading from Riak Search 0.14.2 should be a relatively simple process, you can
follow the [[Rolling Upgrades]] instructions, but before starting the newly
upgraded node, you must add the following section to the `app.config`:

```erlang
  {riak_search, [
                 {enabled, true}
                ]},
```

Afterwards, you can start the node and continue with the [[Rolling Upgrades]]
instructions.

## Upgrading from Riak Search <0.14.2

When moving from older versions of Riak Search to Riak 1.0, rolling upgrades are
not possible. The entire cluster must be shut down, each node upgraded to Riak
1.0, the configs updated with the following section:

```erlang
  {riak_search, [
                 {enabled, true}
                ]},
```

and then restarted.