..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================
Rolling Upgrades
================

https://blueprints.launchpad.net/ceilometer/+spec/rolling-upgrades

As Ceilometer matures through each iteration and adds new features, users
will be required to upgrade their existing environments while minimising
the potential downtime required.


Problem description
===================

Ceilometer currently provides 4 discrete services: polling agents, notification
agents, collector service, and an api service. Each of them provide their
own functionality and the flow of data and purpose of each component can often
be lost on operators when upgrading services.

To ensure a smooth upgrade experience, we need to properly describe the upgrade
path of the components.


Proposed change
===============

Fortunately, due to the unilateral design of Ceilometer where work is done and
handed off without worry, upgrading is actually a simple procedure and only
requires proper ordering of upgrades.

Using the simple mantra of 'never remove, never alter, only add', we can ensure
a new schema change is understandable by both old and new consumers.

There are two upgrade paths to handle -- both require no code change:

1. Full upgrade of services

  1. The database is upgraded using the above mantra.
  2. The collector must be first taken offline. The new collector, that knows
     how to interpret the new payload, can then be started. It will
     disregard any historical attributes.
  3. The notification agent can then be taken offline and upgraded with the
     same conditions.
  4. The polling agents can be taken offline and upgraded. In this path, you'll
     want to take down agents on all hosts before starting. After starting
     first agent, you should verify that data is again being polled.
  5. The api service can be taken offline and upgraded at any point.

2. Partial upgrade of services

  1. The database is upgraded using the above mantra.
  2. The new collector services can be started alongside the old collectors and
     must know how to interpret the new payload. It will disregard any
     historical attributes.
  3. The new notification agent can be started alongside the old agent if no
     workload_partioning is enabled OR if it has the same pipeline
     configuration. If not, the old agents must be loaded with the same
     pipeline configuration first to ensure the notification agents all work
     against same pipeline sets.
  4. The new polling agent can be started alongside the old agent only if
     no new pollsters were added. If not, new polling agents must start only
     in its own partitioning group and poll only the new pollsters. After
     all old agents are upgraded, the polling agents can be changed to poll
     both new pollsters AND the old ones.
  5. API service management is handled by WSGI so there is only ever one
     version of API service running

.. note::

   Upgrade ordering does not matter in partial upgrade path. The only
   requirement is that the database be upgraded first. It is advisable to
   upgrade following the same ordering as currently described: database,
   collector, notification agent, polling agent, api.

Regarding new models, they will be pushed into their own unique queue similar
to how Events and Samples are processed on completely separate queues today.

The above procedures will be added to OpenStack documentation.

Alternatives
------------

1. oslo.versionedobjects - this seems like overkill and will add processing
   overhead to each and every sample
2. versioned queues - this does not have o.vo overhead but will require more
   memory to handle additional queues.
3. versioned payloads - this may simplify payload as we don't need to carry
   historical fields but requires agents to understand each unique version

Data model impact
-----------------

Depending on the amount of changes, this may increase the size of the model
as we need to capture old attributes. We can choose to define a drop period
if this becomes an issue where we stop support attributes from EOL builds.

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

Same amount of processing but **potentially** larger payloads.

Other deployer impact
---------------------

For those not consuming data from API, they will need to be aware of
model changes if they want the latest and greatest.

Developer impact
----------------

None.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  gordc

Ongoing maintainer:
  everyone

Work Items
----------

* Add the above conditions to docs
* Add testing support


Future lifecycle
================

If service requirements change, the above assumptions may not be enough.


Dependencies
============

None.


Testing
=======

* multi-node grenade testing
* migration testing - https://review.openstack.org/#/c/234686/


Documentation Impact
====================

This proposal is nothing but documentation.


References
==========

[1] https://etherpad.openstack.org/p/mitaka-telemetry-upgrades
