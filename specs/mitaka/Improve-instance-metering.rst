..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================
Improve Nova instance metering
==============================

https://blueprints.launchpad.net/ceilometer/+spec/improve-nova-instance-metering

Now we have a great load on Nova API because ceilometer-polling with "compute"
namespace will polling on Nova API periodically. This proposal aims to reduce
the load by abandoning the current way of metrics collection with periodically
polling.

NOTE: The ceilometer-polling service with "compute" namespace is previously
named as ceilometer-agent-compute.

Problem description
===================

Consider the following example. There is an environment with 500 compute nodes
and every compute node deployed with ceilometer-polling with "compute"
namespace (this is a real deployment scale of our company). the polling
interval is set to 60s. For every polling period, ceilometer-polling will
invoke "instance_get_all_by_host()" method (and maybe need querying image and
flavor also) to query the instances of this host from Nova API. That means
the Nova API will receive 500 requests meanwhile every minutes. It would
massively increase the workload on Nova API.

In the metadata caching proposal[1], the instance info from Nova is be cached
in ceilometer-polling by using 'change-since' query filter of Nova server-list
API. That has significantly decreased the amount of items of every Nova query.
But that cannot help to reduce the query frequency.

To illustrate this, We assume that we can deploy 100 instances on one host at
most. with proposal[1], that may reduce 100 instances to 20 instances returned
in every Nova list request, it's not a significant improvement. But with this
proposed change it may will reduce the times of Nova API request from 500
requests/min to 1 requests/min, it would be a significant improvement and that
is the goal of this proposal.


Proposed change
===============

This proposal try to solve the problem described above with changes:

* The ceilometer-polling agent with "compute" namespace runs on every compute
  nodes. Currently, the agent need to get the instances list of this host by
  calling "instance_get_all_by_host" method of novaclient, then traverse this
  instance list to get metrics of instance(e.g. cpu, disk.read.bytes,
  network.incoming.bytes) by inspector, which will invoke the interfaces of
  virt layer to get this metric data. Then ceilometer-polling will assemble the
  metric data from virt layer and instance info from Nova API as sample
  objects.

  Differently, this change want to add a global cache mechanism to cache the
  resource metadata(instance info from Nova db). ceilometer-polling will use
  "instance_get_all" to query all the instance from Nova, and then cache them
  to the global cache. Also, ceilometer-polling will record the resource
  caching time to the cache system, and add an option named
  "interval_to_update_resource" to indicate the max interval of resource cache
  updating. If the time from last updating is larger than this option value,
  the resource cache need to be updated again.

  The "change-since" will also used as a query filter of "instance_get_all" to
  massively reduce the amount of every query and only update the changed
  instance to the resource cache.

* In every ceilometer-polling of every compute node, unlike previous, agent
  will try to query instances list of this host from the global resource cache
  if the cached resources are not out of date, otherwise, query all the
  instances from Nova API to update the cached instance resources and get the
  instances of this host. The rest of the process will be same as before. The
  above "out of date" means the time from last cached resource updating is
  larger than the "interval_to_update_resource" option value.

 * It allows users to enable/disable this global resource cache mechanism,
   when disable this mechanism, the process is same as before,
   ceilometer-polling won't use the global cache but use a local cache
   implemented by proposal[1].


Alternatives
------------

I. Another choice of resource metadata updating solution

Another suggested solution for this goal is using instance events by
notifications instead of instance metrics by polling to collect instance
resource metadata.

Using events to update resource metadata in Ceilometer or Gnocchi will be more
efficient because the resource metadata may only need to be update when
something about resource happened. but I have some concerns about that
approach:

* The resource metadata updating will depend on events collection, what if some
  events didn't be collected ?
* We need to ensure all the resource update actions in other projects of
  OpenStack have corresponding events published, that need other resources
  providers to ensure.
* Do all the events include enough info for ceilometer? we may need the
  resource metadata do some filter query. e.g. use query filtered by instance
  metadata do auto-scaling.

II. Another choice of where to use global cache

Instead of the global cache directly used by compute pollsters of
ceilometer-polling service, another choice is using the global cache in
notification-agent service. Instances info will be polled and cached to the
global cache, metric data will be polled by ceilometer-polling and sent to
notification-agent, in notification-agent, the instances info will be queried
from the global cache, and then assemble the instance info and metric data to
sample objects, then push them to pipeline.

With this proposal, the resource metadata won't be taken from
ceilometer-polling to notification-agent, only the lightweight metric data will
be pushed to notification-agent. In a large scale deployment, this will also
significantly reduce the load of message bus.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Security impact
---------------

None.

Pipeline impact
---------------

None.

Other end user impact
---------------------

None.

Performance/Scalability Impacts
-------------------------------

Need to deploy an external global cache system, more details please see[2].

Other deployer impact
---------------------

None.

Developer impact
----------------

None.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  liusheng

Ongoing maintainer:
  liusheng

Work Items
----------

* Implement a global resource caching mechanism to cache instance info.

* Change the instance metric discover to use the cached resource.


Future lifecycle
================

None

Dependencies
============

None


Testing
=======

- The current tests will need to modify to adapt these change.
- Unit tests should be add to cover these change.

Documentation Impact
====================

Documentation should be updated to add description for this.

References
==========

[1] https://review.openstack.org/#/c/185084/

[2] https://review.openstack.org/#/c/250778/
