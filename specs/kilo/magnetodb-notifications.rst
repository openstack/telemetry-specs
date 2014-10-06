..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================
MagnetoDB Notifications
=======================

https://blueprints.launchpad.net/ceilometer/+spec/support-magnetodb

This spec proposes to add MagnetoDB metering to Ceilometer by catching
notifications emitted by MagnetoDB. For example, when a table is created a
notification called table.create.start is emitted and when the table creation
is finished a notification called table.create.end is emitted.

Problem description
===================

The notifications sent by MagnetoDB needs to be fetched from message bus
and then transformed into samples and then stored in the database. To
achieve this a notification handler is needed.

Proposed change
===============

A new notification plugin for MagnetoDB to transform the notifications
into samples using existing ``NotificationBase`` implementation as a model.

`Example of a notification and corresponding sample`_

List of new samples:

* :Resource ID: id
  :Name: table.create
  :Type: gauge
  :Volume: 1
  :Unit: table
  :Timestamp: time

* :Resource ID: id
  :Name: table.delete
  :Type: gauge
  :Volume: 1
  :Unit: table
  :Timestamp: time

* :Resource ID: id
  :Name: index.size
  :Type: gauge
  :Volume: 2
  :Unit: index
  :Timestamp: time

Alternatives
------------

None.

Data model impact
-----------------

None.

REST API impact
---------------

There will be additional valid values in the query parameters but no changes
to API endpoints.

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

No new impacts. As we are only storing table creation and deletion
notification, the impact would be negligible.

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
  ajayaa

Other contributors:
  None.

Work Items
----------

* Establish expected data.

* Create tests of transformation of notifications to samples.

* Create notification plugin to consume notifications.

* Create tests of notifications across fake bus.

* Create sample query tests.

Future lifecycle
================

In future new types of notifications are expected from the MagnetoDB.
These will need to be handled either by additional notification
plugins or (hopefully) generic notification handling. The MagnetoDB team
will be responsible for collaborating with the ceilometer team to ensure these
are handled smoothly.

Dependencies
============

None.

Testing
=======

Unittests.

Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section:
   http://docs.openstack.org/developer/ceilometer/measurements.html

.. _Example of a notification and corresponding sample:
   https://gist.github.com/ajayaa/3e4617a832afd9f229c6

References
==========

None.
