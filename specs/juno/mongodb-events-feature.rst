..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Enable event feature on MongoDB
===============================

https://blueprints.launchpad.net/ceilometer/+spec/mongodb-events-feature

Events feature was implemented on SQLAlchemy and HBase backends. And now it is
necessary to extend it on MongoDB and DB2.

Problem description
===================

The usecase is to store events in database (MongoDB or DB2) and to get
events (event's types, traits, trait's types) from this database.

The main idea is to implement event feature in way like it was done on
other drivers (SQLAlchemy and HBase).

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

Add new methods in ceilometer.storage.pymongo_base.py for storing and getting
events from database. Basing on events implementation that have been already
done we should create next methods:

1. record_event -- method for storing collected events into MongoDB and DB2

2. get_events -- method for getting events from MongoDB and DB2 with specified
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

We will need to add new collection into MongoDB and DB2 for events

Collection:

- events
    - _id: uuid of event (Event.message_id)
    - event_type: description of event's type
    - timestamp: time stamp of event generation
    - traits: [array of {trait_name: string, trait_type: integer,
              trait_value: data}]

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

* Create new method for event feature implementation on MongoDB and DB2
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

