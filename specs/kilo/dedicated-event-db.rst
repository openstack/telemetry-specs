..
   This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Dedicated Event Database
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/dedicated-event-db

Ceilometer provides the ability for metering, alarming, and eventing. As
Ceilometer tracks more and more data, scalability issues will arise.

Problem description
===================

Currently, metering and event data coexists on the same database. While
related, there's a logical separation in the metering and event models
where the data in each model has its own unique data and structure; the
metering model can be best described as a time series while the event model is
closer to an entity attribute model. As the models are different in what they
capture, it makes sense that deployers may choose to use different storage
drivers to store each data set.

Proposed change
===============

Similar to the work done to split alarming into its own database, this
blueprint is to allow for event related data to be stored in its own database.

Alternatives
------------

Continue on as status quo.

Data model impact
-----------------

None, the model remains the same.

REST API impact
---------------

None, the api will remain the same. The only change is the api will now
read from events database when event api is called.

Security impact
---------------

Policies and permissions will remain unchanged. The API will continue to
function as before and give the same amount of access to data. To that effect,
users will require an admin role to have access to data.


Pipeline impact
---------------

None.

Other end user impact
---------------------

The end user will now be able to store event data in a different database from
metering data. Alternatively, they can continue on as how Ceilometer currently
functions and store event and metering data in the same database.

Performance/Scalability Impacts
-------------------------------

This will probably improve scalability as it allows deployers to split event
and metering data so a single database is not flooded by queries against
disconnected data sets. There also remains the ability to write to the same
db so the currently design/performance can continue as well.

Other deployer impact
---------------------

A config option to specify an event database is added::

   [database]
   metering_connection=hbase://
   alarm_connection=sqlite://
   *event_connection=mongodb://*

Developer impact
----------------

Future event related work should be done in event submodule.

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

* Move event models to event submodule
* Add support for event_connection option and create framework for drivers
  in event submodule
* Move over event related code for each driver from storage module to event
  submodule

Future lifecycle
================

There is common code shared between alarm, event, and metering backends which
can be refactored.

Dependencies
============

None

Testing
=======

Existing testing should be sufficient. The only additional tests required are
test the ability to define separate backends for each data set (alarm, event,
metering) and to ensure the capabilities of each backend return the correct set
of capabilities.

Documentation Impact
====================

We need to update docs to highlight how to enable event specific backend.

References
==========

dedicated alarm database blueprint: https://blueprints.launchpad.net/ceilometer/+spec/dedicated-alarm-database
