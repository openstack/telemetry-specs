..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================
Enable event feature on HBase
=============================

https://blueprints.launchpad.net/ceilometer/+spec/hbase-events-feature

Currently events feature is implemented only on SQLAlchemy backend, so choosing
another database events are not supported.

Problem description
===================

The usecase is to store events in HBase and to get events (event's types,
traits, trait's types) from this database.

The main idea is to implement event feature in way like it was done on
SQLAlchemy driver.

Usecases:

- to get all events that were created in a specified period of time

- to get events by event's type

- to get events by event's message id

- to get events by trait's description, type, and value

- to get all event's types that were stored

- to get all trait types belonging to specific event type

- to get all traits that could be found in specific event's type, and
  optionally with specific trait type.

Proposed change
===============

Add new methods in ceilometer.storage.impl_hbase.py for storing and getting
events from database. Basing on events implementation for SQL database we
should create next methods:

1. record_event -- method for storing collected events into HBase

2. get_events -- method for getting events from HBase with specified
   filters:
   - period of event's generation time
   - event's type
   - event's message id
   - trait's description, type, and value (here should be taken into account
   possible operations on values, for ex: <lt, le, eq, ne, ge, gt>)

3. get_event_types -- method for getting all event's types that were stored in
   database.

4. get_trait_types -- method returns all trait types for all stored event
   types or only for one event's type if it is specified.

5. get_traits -- method returns all traits for certain event type, or traits
   for specified trait's type if type is defined.

Alternatives
------------

None

Data model impact
-----------------

We will need to add new collection into HBase for events

Collection:

- events
      - row_key: timestamp of event's generation + uuid of event
                 in format: "%s+%s" % (ts, Event.message_id)
                 If we store events with such row_key they are sorted by
                 timestamp in the database.

      -Column Families:
          f: contains the following qualifiers:
              -event_type: description of event's type
              -timestamp: time stamp of event generation
              -all traits for this event in format
              "%s+%s" % (trait_name, trait_type)

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

None

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  * idegtiarov

Ongoing maintainer:
  * idegtiarov

Work Items
----------

* Create new method for event feature implementation on HBase
* Test method in unit tests

Future lifecycle
================

None

Dependencies
============

None

Testing
=======

This code will be tested in unit tests for event feature

Documentation Impact
====================

None

References
==========

None

