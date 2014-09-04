..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================
Merge Compute and Central agents
================================

https://blueprints.launchpad.net/ceilometer/+spec/merge-central-and-compute-agents

Central agent was previously extracted from compute agent code to support not
only compute metrics to be polled, but also to collect measurements from other
services and resources. Actually the main difference between these two agents
was in the way they discover and poll the resources. After the discovery was
unified for the all pollsters, we actually are having the logic duplication,
having two different agents, although they both discover and poll *some*
resources and collect then *some* metrics.

It looks logical to merge the code of agents back. They even might have exactly
the same pipeline being used, but differ only in the console script parameter
*--polling-namespace* to have the opportunity still to separate them in the
real deployment. Also that might be possible to define exact list of pollsters
to be used via *--pollster-list* script parameter here.

Problem description
===================

We have logic mismatch with two separated agents, although they might be
part of one piece of code, configured via pipeline.

Proposed change
===============

We need to merge carefully code of compute and central agents, using unified
mechanism we already have to process discovery and polling processes.

Also we need to save the opportunity of setting up agent to poll only compute
or only meters used to be polled by central agent. Basically all-in-one
installation will be satisfied with agent polling all available meters, but
production installations need this separation. This will be achieved by adding
additional per-agent CLI script parameter:
*--polling-namespace={compute|central|compute,central}*
(with adding here *ipmi* when this agent will be added here as well). In this
case running of polling agent will look like following:

    ceilometer-polling --polling-namespace central compute --logfile /var/log/ceilometer/polling.log

This parameter will be passed to the agent manager class directly, and will
define what setup.cfg namespace (*ceilometer.poll.central*,
*ceilometer.poll.compute* or both) with possible pollsters to load. This
will emulate current workflow (and probably become deprecated in next cycles).

Also this change will propose new way of how do pollsters to be used are
chosen for polling agent instance - via CLI script parameter
*--pollster-list=<list of pollster entrypoints or wildcards>*. In this case
polling agent running command will look like this:

    ceilometer-polling --pollster-list image image.size storage.* --logfile /var/log/ceilometer/polling.log

If both of these parameters will be set, we need to use them both (possibly using
logical AND operation to find out final list of pollsters to be used).

Alternatives
------------

None

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

Better RAM usage in case of development OpenStack all-in-one installations.

Other deployer impact
---------------------

* It'll be less services Ceilometer will run on node (if it'll be all-in-one
  installation) and we'll have more unified way of Ceilometer installation.
* Downstream impact on how Ceilometer packages will be created and on the
  deployment tools that use them right now. Currrently we are having two
  *separated* packages for central and compute agents in Ceilometer (both for
  the .rpm and .deb), and these packages have different .service/.upstart
  definitions for launching the corresponding service. The simplest way to
  fix this moment will be to add new combined package and make current
  separated ones to use introduced in this change namespaces like below::

      s/ExecStart=...ceilometer-agent-compute/
        ExecStart=...ceilometer-agent-polling --pollster-namespace=compute/

* We need to update puppet-ceilometer module and add there new manifest
  that will support combined agent.

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  dbelova <dbelova@mirantis.com>

Work Items
----------

* Merge central agent code into the current compute one
  (ceilometer-polling). Leave ceilometer-agent-compute and
  ceilometer-agent-central CLI commands for a deprecation
  cycle.
* Investigate Tempest testing impact and provide new tests if needed.
* Provide Devstack with new agents scheme support (for all-in-one and multinode
  installations).
* Investigate Grenade tests aspect.


Future lifecycle
================

None


Dependencies
============

None


Testing
=======

This change needs to be tested by merged unit tests and via integration tests
(old and possible new ones).


Documentation Impact
====================

We'll need to rewrite our installation guide and common documentation parts
with the information about one agent instead of two.


References
==========

None
