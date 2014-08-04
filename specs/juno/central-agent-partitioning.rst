..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================================================
Horizontal scaling and work-load partitioning of the Central Agent
==================================================================

https://blueprints.launchpad.net/ceilometer/+spec/central-agent-partitioning

The central agent's job is polling resources for information, transforming that
information into samples and passing the samples on to the Collector Agent.

This specification proposes an implementation of coordination between multiple
Central Agents, which could then dynamically distribute the workload between
them, providing scalability and high availability.

Problem description
===================

Currently, each Central Agent retrieves a set of resources and polls all of
them. If we have multiple Central Agents running, they all poll the same set
of resources, which prevents us from scaling out horizontally.

At the start of each polling interval, each of the pollsters retrieves a list
of resources to poll from its Discovery plugin (configured in the pipeline
or a default one). This makes the discovery process a great place to implement
the coordination and partitioning logic, while the pollsters themselves can
remain in blissful ignorance of anything going on.

Proposed change
===============

The basic idea is to use the *tooz* [1]_ library for group membership and
hashing to assign resources to active Central Agents.

* Discovery plugins on different Central Agents that discover the same set of
  resources would join the same group in *tooz*.
* For each polling interval, after independently discovering the global set of
  resources that need to be polled by one or other of the central agents,
  they would get a list of all group members (Discovery plugins in other
  Central Agents that have the same set of resources).
* They would use hashing to determine the set of resources assigned to the
  Discovery's Central Agent.

**Determining the resources we're responsible for**

We have a list of resources and get a list of active agents from *tooz*. We
then get our assigned resources as follows::

    our_key = sorted(agents).index(our_agent_uuid)
    our_resources = []

    for resource in resources:
        key = hash(resource) mod len(agents)
        if key == our_key:
            our_resources.append(resource)

    # or more pythonic
    our_resources = [r for r in resources if hash(r) mod len(agents) == our_key]

In essence we hash the resources to <number of Central Agents> of buckets and
only poll the resources that fall into our bucket. A good hash function
[3]_ ensures that the resources are evenly distributed to the active Central
Agents.

**Grouping**

The pollster's Discovery plugin (be it a Compute Discovery, Hardware Discovery,
etc.) provides the scope its resources are a part of.

For example, if a Discovery plugin isn't constrained to a subset of
resources, as is the case for most Discovery plugins, then it should simply
join the global group of unconstrained Discovery plugins.

If, on the other hand, the resources that the Discovery plugin can discover
are constrained, like in the case of Compute Discovery, then the group name
should reflect their scope. An example of this would be
'compute-<hostname>-discovery'. This way only the pollsters that are polling
the same host will share their workload between them.


**What happens when we start another agent (or stop an existing one)?**

*tooz* allows us to register a callback that is called when a member joins or
leaves the group. It keeps track of member liveness using a heartbeat
mechanism.

When a member joins or leaves the group, this is what happens to:

* **The agents already running**
  If they are currently in the middle of polling, they complete their full
  polling cycle and only then they re-balance their hash buckets.

* **The agent we just started**
  Joins a group first, but then waits for one polling interval to ensure all the
  other agents have updated their hash buckets as well, then starts polling.

* **The agent that is stopped/crashes mid-cycle**
  The resources that the stopped agent hasn't polled yet will not be polled in
  this cycle, but they will be polled from the next one on.


**Generalizing the implementation for re-use**

The need for coordinated assignment of "things" (resources, alarms, ...) to
agents is not unique to the Central Agent. Currently, the Alarm Evaluator could
make use of it as well to have multiple Alarm Evaluators running, each
evaluating their share of alarms.

This functionality could be captured in a *PartitionCoordinator* class, which
agents could use like::

    partition_coordinator = PartitionCoordinator(group='alarm')
    partition_coordinator.start()

    every evaluation_interval:
        all_alarms = get_all_alarms()
        my_alarms = partition_coordinator.get_my_subset(all_alarms)
        for a in my_alarms:
            evaluate(a)

.. note::
   The actual change-over of the alarm partitioning coordination to the
   proposed approach will be tracked in a separate blueprint.

or in the case of the central agent::

    partition_coordinator = PartitionCoordinator(group='central_agent')
    partition_coordinator.start()

    every polling_interval:
        all_resources = discover_resources()
        my_resource = partition_coordinator.get_my_subset(all_resources)
        for r in my_resources:
            poll(r)


Alternatives
------------

* **Fabio Gianetti's approach** [2]_.

  Fabio's approach uses source<->agent assignments in the database for
  figuring out what to poll and a heartbeat in combination with additional
  agents listening for that heartbeat for failure detection.

  In contrast, this proposal uses *tooz* for failure detection (via heartbeats
  as well). Additionally, the resource allocation is more dynamic since the
  resources are assigned to agents evenly at any point in time. It is also
  more lightweight since we don't need to keep an explicit resource<->agent
  mapping in the database, but use hashing instead.

* **Locking**

  Another approach would be to use distributed locking provided by *tooz*.
  Before a pollster would poll a resource, it'd need to acquire its lock.
  Pollsters contend for the locks and whoever gets the lock, polls the
  resource.

  The downside of this approach is the overhead of distributed locking.
  Acquiring a distributed lock incurs a cost (time, network traffic). When using
  distributed locks for resource contention, this cost is incurred per-resource.
  Whereas in the approach with group membership, the coordination cost is
  incurred only when a member joins/leaves the group, the frequency of which is
  negligible compared to the amount of resources.

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

None

Pipeline impact
---------------

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

Positive


Other deployer impact
---------------------

If deployers want to use multiple central agents, they will need to deploy
one of the tooz backends (ZooKeeper, memcached, possibly just an AMQP broker
soon)

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  nejc-saje

Other contributors:
  chdent

Ongoing maintainer:
  ceilometer team

Work Items
----------



Future lifecycle
================


Dependencies
============

* tooz
* one of the backends for tooz (ZooKeeper, memcached, possibly just
  oslo.messaging)


Testing
=======

The implementation should be tested with unit tests.


Documentation Impact
====================

Operator's manual should explain the process and properties of running multiple
Central Agents.

References
==========

.. [1] https://github.com/stackforge/tooz
.. [2] https://review.openstack.org/#/c/101282/5
.. [3] http://en.wikipedia.org/wiki/Hash_function#Properties
