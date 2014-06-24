..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================================
Dedicated database for the alarm definition and history of Ceilometer
=====================================================================

https://blueprints.launchpad.net/ceilometer/+spec/dedicated-alarm-database

Currently if we use MongoDB for storing metering data, we are forced to use
MongoDB for storing events and alarms. Because the API service can use only
one database connection object.

Also metering and alarm are two completely different things, but the db code
are currently in the same place.

The blueprint proposes to allow to have a different database for alarms
definition and alarms history.

Problem description
===================

The usecase is I want to store my metering data into MongoDB and my alarm
definition and history into MySQL

Proposed change
===============

The idea is to move all alarms db stuffs from ceilometer/storage to
ceilometer/alarm/storage

ceilometer.storage.get_connection_from_conf get a new argument 'purpose',
that can be set to metering or alarm.

This will allow to use a different namespace to load a driver that depends
of the purpose.

New entry points will be added to have a different set of driver for alarm
and metering.

Ceilometer API will use two connection objects, one for metering and one for
alarm depending on the used API endpoint requested.

Alternatives
------------

None

Data model impact
-----------------

None

In case of SQLAlchemy, migration scripts will continue to be stored in a
common place to ensure easy migration.
And if a dedicated database is used, we accepted having unused and empty
tables.

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

None

Other deployer impact
---------------------

The deployer can now optionally define a dedicated database for the alarming
of ceilometer by adding::

    [database]
    connection = mongodb://localhost/ceilometer
    alarm_connection = mysql://localhost/ceilometer

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  sileht

Ongoing maintainer:
  sileht

Work Items
----------

* Splits the alarm db API from metering db API
* Remove unuseful drivers for alarm API


Future lifecycle
================

None

Dependencies
============

None

Testing
=======

The is code refactoring with a new optional configuration option.
This configuration will be tested in new unit tests.


Documentation Impact
====================

None

References
==========

None
