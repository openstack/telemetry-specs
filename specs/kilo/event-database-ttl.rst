..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================
Support Time To Live on Event Database
======================================

https://blueprints.launchpad.net/ceilometer/+spec/event-database-ttl


Problem description
===================

Event database grows over time, after we dump data to larger storage system,
the old data in event database should be cleared, but now, there is no such
way to do it.

Proposed change
===============

Add time to live feature on event database, just like what we do on metering
database. A new option event_time_to_live will be added, such as what we do
for metering database.

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

User now can clean event database when they run ceilometer-expirer
and set event_time_to_live options to value that larger than 0.

Performance/Scalability Impacts
-------------------------------

Performance can be improved since event database can keep light.


Other deployer impact
---------------------

None

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  aji-zqfan

Other contributors:
  Contributors who want to help on databases except MongoDB

Ongoing maintainer:
  aji-zqfan

Work Items
----------

1. Implement it on MongoDB
2. Implement it on other database back end


Future lifecycle
================

None


Dependencies
============

None


Testing
=======

Unit test code will be added along with source code.


Documentation Impact
====================

New option will be added, so OS Configuration Document should be update,
and new feature is added, Administrator's Guide Document should be updated
too. But not this spec's job.


References
==========

None
