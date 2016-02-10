..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============
Alarm quotas
=============

https://blueprints.launchpad.net/ceilometer/+spec/quotas-on-alarms

Currently, it is possible to create an unlimited number of alarms in
Ceilometer. It would be useful to be able to specify a maximum number of
alarms allowed per user or project in order to have some control and
limit on the computing power required by Ceilometer.


Problem description
===================

Presently, due to alarm evaluation process, the large number of alarms
affects the performance of Ceilometer, but improvements to reduce this
issue are underway with blueprint [1]_.

Even if performance wasn't an issue, in order to prevent abuse from end users,
the alarm quota mechanism must be implemented.


Proposed change
===============

The proposed change will allow to express maximum number of alarms that
can be set by user or project. These limits, if enforced,
could be specified in Ceilometer's configuration file (alarm section)::

    [alarm]
    user_alarm_quota =
    project_alarm_quota =

When a user adds an alarm, the following checks would occur:

    1. Has the user filled his alarms quota ? If yes, the query is rejected.
    2. Is the user's project's alarms quota filled ? If yes,
       the query is rejected.
    3. Add the alarm.

Quotas don't have to be set at every level. For example,
setting a limit only at the project level will allow users to create as many
alarms as they want as long as the sum of alarms for their specific project
is less than the limit.

In order to provide backward compatibility, the default value for alarm
quotas will be set to None.

Alternatives
------------

An alternative solution, storing project/user quota in Keystone for
all Openstack resources, was proposed during Havana development cycle [2]_.
This solution was since abandoned in favor of letting each component manage
quotas on their own.

An alternative solution to implement quotas in Ceilometer would be
to use the same mechanism that other Openstack components are using. This
will imply the addition of an OS-QUOTAS API extension in Ceilometer,
that will be used by the cloud operator for project/user specific quota
assignments. These specific project/user quotas must be stored in Ceilometer
backend, implying several backend specific quota storage implementations.

This alternative solution is more complex and could be implemented in future
release cycles.


Data model impact
-----------------

None

REST API impact
---------------

None

No API method is either added or changed. Nevertheless, the new error http
response code (HTTP 403) will be returned when the alarm quotas are exceeded.

Security impact
---------------

None

Pipeline impact
---------------

None

Other end user impact
---------------------

The impact of this feature for Heat Alarm and AutoScaling resources
will be evaluated and fixed.

Performance/Scalability Impacts
-------------------------------

None

Other deployer impact
---------------------

The deployer can now define alarm quota in configuration file (alarm
section)::

    [alarm]
    quota_user_alarm =
    quota_project_alarm =

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  arezmerita

Other contributors:
  mhu-s


Work Items
----------

* Implement the alarm quota mechanism
* Test the feature in unit tests

Future lifecycle
================

As mentioned in alternatives section, API extension for dynamic quota
management can be implemented in future release cycles.


Dependencies
============

None

Testing
=======

Tempest tests will be added to tests this feature

Documentation Impact
====================

Ceilometer installation documentation will be updated


References
==========

.. [1] https://review.openstack.org/#/c/95418/
.. [2] https://review.openstack.org/#/c/40568/
