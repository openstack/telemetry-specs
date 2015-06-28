..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Mandatory API Query Limits
==========================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/ceilometer/+spec/mandatory-api-limit

Currently, Ceilometer's api allows users to query for as much or as little
data as they desire. They are also allowed to query for everything no matter
the size of results.


Problem description
===================

Allowing the ability to have return the entire world of data is not a realistic
use case. While it is easier in theory to query for everything, in reality,
this is not viable as it puts a lot of strain on CPU and memory to manage all
the data required. In addition, these queries take a significant amount
of time to calculate and will always time out even if enough resources are
available to the system.

As Ceilometer intends to remove unused pagination, another form of query
restriction is required.

Proposed change
===============

Mandatory limits must be enforced on queries, whether it be a limit value.
If neither value is provided, a default restriction of 100 will be
applied. This behaves similarly to other dbs such as ElasticSearch which limit
the number of results returned if no restrictions are provided.

Users are still allowed to query the entire set of data if they choose to, but
instead of implicitly doing so as it currently functions, users will need to
explicitly do so.

These restrictions will apply to both events and meters api.


Alternatives
------------

Nothing, and let users blame Ceilometer for insane queries.

Data model impact
-----------------

None

REST API impact
---------------

All GET queries will require a limit value. If none is given, a default of 100
will be applied.

Security impact
---------------

None

Pipeline impact
---------------

None

Other end user impact
---------------------

A limit value is required to get data as expected. If none is provided, the
returned results may be truncated. A warning will be logged whenever the default
limit is applied.

Performance/Scalability Impacts
-------------------------------

This will limit the load created by unintentional large queries.

Other deployer impact
---------------------

A new option will be defined to allow operators control of what the default
return limit is.

Developer impact
----------------

Unit tests need to be written to consider this limit.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  gordc

Other contributors:
  None

Ongoing maintainer:
  None

Work Items
----------

- Add option to control default limit value
- Apply limit check to meter queries and fix UT
- Apply limit check to event queries and fix UT
- Update docs

Future lifecycle
================

We can derive a query link which returns next set of data based on last
timestamp value of returned set.

Dependencies
============

None

Testing
=======

Unit test to test default limits are applied.

Documentation Impact
====================

This may be a breaking change to some -- Ceilometer will probably be already
broken to them. That said, this will need to be publicised.

References
==========

