..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Cassandra based storage driver
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/cassandra-driver

Cassandra is a distributed data-store that is AP in the CAP trade-off.
As such it is very well suited to metric storage.  This project adds
support for storing data in Cassandra, just like we currently support
HBase, SQL, MongoDB et al.

Problem description
===================

Cassandra is well-suited to storing metrics at large scale; by adding
support for it to Ceilometer, it should be able to scale to bigger data sizes
than is possible with non-distributed systems, and to higher throughput
than is possible with a consistent (in the CAP sense) system.

In addition, Cassandra can operate across multiple datacenters, so we
may be able to enable geo-replication fault-tolerance.

Proposed change
===============

This change will add Cassandra drivers to Ceilometer for samples, events and
alarms.

Alternatives
------------

HBase is a good scalable alternative to Cassandra.  In CAP, HBase is more C,
and Cassandra is AP, so they are at different points in the design space.

Cassandra is also easy to configure and maintain compared
to other distributed datastore options.

I do not expect everyone to use the Cassandra driver, but I expect some
people will want to, and we can offer them that option without interfering
with the desires of operators that do not configure the Cassandra driver.

Data model impact
-----------------

The Cassandra driver uses a different data schema on disk, optimized for
the unique requirements ('wide rows are better') and the consistency
characteristics ('last write wins') of Cassandra.

However, the external data model should not change due to the effort.

Migration from existing datastores will not be supported.

REST API impact
---------------

None

Security impact
---------------

The Cassandra backend should be appropriately protected, just like any
other datastore.

Pipeline impact
---------------

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

Cassandra is believed to be more scalable than some other options,
but there should otherwise be no impact.  There will not impact on users that
do not choose to configure the Cassandra backend.

Other deployer impact
---------------------

The driver will initially support at least the same options as the HBase
driver.

A new storage URL schema, 'cassandra' will be added that will enable
a cassandra backend, as with all the other drivers.

Users must opt-in to this option, so merging it is low-risk.

We are adding a dependency on the cassandra-driver for python (Apache 2).
This has been accepted into the openstack-requirements project.  It would
only be required if cassandra backends are enabled.

Developer impact
----------------

Another storage driver will require maintenance.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  justin-fathomdb

Other contributors:
  None

Ongoing maintainer:
  justin-fathomdb

Work Items
----------

Initial implementation has already been submitted in Gerrit.

A lot of the code is actually code not necessarily specific to Ceilometer: in
particular a Cassandra 'ORM' type layer, and an adapter for eventlet.  Based on
discussions during the code review process, this isn't a great long-term
solution, though it might be that the code has to be in Ceilometer initially.
We will try to get the eventlet adapter into the upstream Cassandra driver.  We
will start the ORM layer where we need it in the Ceilometer code, but likely
move it into Oslo as it is used in multiple OpenStack projects.

Hopefully we can add Cassandra support to Gnocchi soon; that will likely be the
second OpenStack project using the Cassandra 'ORM' layer, which would support
putting it into Oslo.


Future lifecycle
================

To be maintained by justin-fathomdb.


Dependencies
============

None


Testing
=======

A mock Cassandra implementation has been added, so the unit tests are more
like integration tests.

We can also try to get Cassandra added to devstack, which would then support
creating a new testing gate-job; for even better continuous testing.


Documentation Impact
====================

We should just document the cassandra:// endpoint.  It creates its own schemas etc.


References
==========

Interesting blog post which talks about storing metrics in Cassandra:
http://www.datastax.com/dev/blog/metric-collection-and-storage-with-cassandra

Cassandra is often abbreviated C*

