..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================================================
Add Support to include Alarm level as part of notifications
===========================================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-alarm-level

The goal of this blueprint is to expose an alarm priority field that can be used
to set the alarm importance level.

Problem description
===================

A detailed description of the problem:

* Alarms in ceilometer have no way to identify the criticality of an alarm.
  All we know today is if alarm has been triggered or insufficient data. This
  might be ok in general.

* But from auditing point of view, cloud administrators would like to know if
  an alarm is triggered and if so how critical is it. A hardware failure would
  be a critical alarm vs an occasional spike in cpu level could be medium or
  low. Differentiating this level would be very useful for audit.

Proposed change
===============

Expose a new field called alarm_priority as part of the alarm base object. This
will be exposed via the alarm notifications so the alarms can be filtered by
priority.



Alternatives
------------
None

Data model impact
-----------------

Need to update the Alarm model to include a new field called priority


REST API impact
---------------

We will probably want to expose requests to get alarms by priority.

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
  pkilambi

Ongoing maintainer:
  pkilambi

Work Items
----------

The specific work items include:

  * Update the model layer to include the necessary fields
  * Update the notifier module to expose the priority field
  * Update the rpc and service modules under alarm
  * Add support for alarm level in the python-ceilometerclient
  * Update unit tests.


Future lifecycle
================

None

Dependencies
============

None


Testing
=======

Unit and integration Tests will be added/updated to cover the necessary scenarios
for alarms

Documentation Impact
====================

We might need to update the example json in alarm api docs.


References
==========

Initial implementation is here https://review.openstack.org/#/c/142849/
