..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================================
Coordination of Notification Agents
===================================

https://blueprints.launchpad.net/ceilometer/+spec/notification-coordination


Problem description
===================

Currently, the notification agent can function in active/active HA where each
notification agent grabs notifications from the message queue in a round robin
type rotation (it checks for message(s), grabs message(s)). The issue that
arises is that if we want to implement a pipeline to process events, we cannot
guarantee what event each agent worker will get and because of that, we cannot
enable transformers which aggregate/collate some relationship across similar
events.

This also fixes an existing bug where if multiple notification agents are
enabled, the pipeline transformers may not work as expected because it will
only act on the samples it sees on current worker.

Proposed change
===============

Similar to how we implemented coordination between the central agents, this bp
is to proposed a way to coordinate which event gets passed to which
notification agent.

The proposed solution is to implement additional processing steps as follows::

  1. Existing listener in notification agent continues as is, each agent will
     listen to the same queue and grab messages as they arrive without any
     coordination.
  2. After the agent has converted messages into samples and events, the result
     will be republished onto a ceilometer internal queue. The sample and event
     will be published to (multiple) topics corresponding to known pipeline
     sinks. For example, the pipeline has two sinks, one that processes all
     meters, the other only processes compute meters. In this case, all agents
     will publish to a queue for 'all-meters' sink, and agents with compute
     meters will publish to 'compute-only' sink.
  3. An additional listener will be added to each agent which will listen
     to new internal queues. This is where the coordination will happen. Each
     agent will listen to a set of targets created in step 2.

The existing notification agent will exists as is as there isn't a need for
additional queueing to handle single worker scenario.

Alternatives
------------

1. The notification agent can be coordinated across existing targets ie. each
   existing projects exchange. This does not allow for cross exchange sinks.
   ie. we cannot have an aggregation that combines events from different
   projects. This might be a non-issue for samples (who needs to aggregate and
   transform a new sample from samples from nova and neutron?). It will
   probably be an issue for events which we may want to coordinate across
   exchanges. This would mean we should have samples listener to coordinate
   across exchanges, and events listener which would do the above.

2. The notification agent can be coordinated across endpoints. This will cause
   issues as we need to have duplicate queues that each agent listens to (to
   ensure agent with endpoint gets the message it needs). It also runs into
   potential loss or duplicate messages because of duplicate queues.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Security impact
---------------

Same as existing security concerns polling agents face -- we need to ensure
that phantom agents can't register and create phantom tasks.

Pipeline impact
---------------

It should actually work properly when multiple workers deployed.
Additional event pipeline will be added in a subsequent bp and patch.

Other end user impact
---------------------

None.

Performance/Scalability Impacts
-------------------------------

This should improve write performance (at least from SQL pov) as we can do
batch inserts.


Other deployer impact
---------------------

Users will need to enable coordination. By default, the notification agent is
expected to continue as currently ie. pull messages if they exists.


Developer impact
----------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  chungg

Ongoing maintainer:
  chungg

Work Items
----------

- add coordination of notification agent
- reuse or add tests for coordination


Future lifecycle
================

First step to adding in event specific pipeline.

Dependencies
============

- tooz
- one of the backends for tooz (ZooKeeper, memcached, redis, etc...)

Testing
=======

- coordination tests exist. We may just need to add it for notification agent
  if the existing agent coordination tests are central agent specific.


Documentation Impact
====================

- add notes on enabling coordination of notification agents.

References
==========

[1] https://github.com/openstack/ceilometer-specs/blob/master/specs/juno/central-agent-partitioning.rst
