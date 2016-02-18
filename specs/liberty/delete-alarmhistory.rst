..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Add the function of deleting alarm history
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/delete-alarmhistory

Currently the ceilometer-expirer doesn't delete expired AlarmChanges.
Remained AlarmChanges would be cause of wasted disk-space and slow response.
This blueprint adds the function of deleting alarm history.

Problem description
===================

Currently the ceilometer-expirer doesn't delete expired AlarmChanges.
So we need to add the function of deleting alarm history.

For doing so, a time-to-live (TTL) value needs to be specified.
Currently the metering_time_to_live (aka time_to_live) value is used
for deleting metering data. However, alarm history expirations shouldn't
be the same frequency as sample expiration.

Therefore, separate TTL value needs to be introduced for alarm history.


Proposed change
===============

Add the function of deleting alarm history. This functionality will be
put into the ceilometer-expirer.

New TTL, alarm_history_time_to_live, is added as a config option to database
section.


Alternatives
------------

It is possible to use the same time_to_live value for both sample and
alarm history.

However their scale might be completely different, so the expiration
frequency shouldn't be the same.

Therefore, we will have separate TTL.


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

None

Performance/Scalability Impacts
-------------------------------

None

Other deployer impact
---------------------

A new option should be set if deployer wants to enable this feature.

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------


Primary assignee:
  * honjo-rikimaru-c6
  * ZhiQiang Fan

Other contributors:
  None

Ongoing maintainer:
  * ZhiQiang Fan

Work Items
----------

* "alarm_history_time_to_live" is added as a config option.
* support mongodb back end
* support sqlalchemy back end


Future lifecycle
================

None

Dependencies
============

None

Testing
=======

This code will be tested in unit tests for expire.

Documentation Impact
====================

None

References
==========

[1] https://blueprints.launchpad.net/ceilometer/+spec/delete-alarmhistory

[2] bug report that addresse this issue
https://bugs.launchpad.net/ceilometer/+bug/1289141

[3] ongoing patch for [2]
https://review.openstack.org/#/c/87869/

