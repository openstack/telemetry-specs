..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.
 
 http://creativecommons.org/licenses/by/3.0/legalcode 
 
===========================================================
Instance Auto Restart with a Heartbeat Meter and Ceilometer
===========================================================

https://blueprints.launchpad.net/ceilometer/+spec/instance-autorestart

Problem description
===================

Ceilometer currently does not support a heartbeat meter.
In order to support an auto restart on an instance shutdown or a host failure a
heartbeat pollster is needed.
This spec deals with adding a new meter to Ceilometer which monitors the
instance health by using a ping command.

Proposed change
===============

Adding a new pollster to Ceilometer which will monitor the instance health by
checking its heartbeat.
This meter will contain a binary value: 1 if the instance is available and if
not it will return 0.

According to the alarm condition an auto restart will be preformed using an
OS::Heat::HARestart resource in heat.

Metric Definition:
 =================================		========	=============	=========
	Name								Type    	Unit			Origin
 =================================		========	=============	=========
	heartbeat							g			counter			p
 =================================		========	=============	=========

Alternatives
------------

The alternative solution is to add a new property inside OS::Nova::Server
(e.g. highly_available) which will tell Heat/Ceilometer to keep the instance
up and available.

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

Each instance that is being monitored should be open to ICMP requests.

Pipeline impact
---------------

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

Using a different Ceilometer alarm for each instance may impact Ceilometer
performance if multiple instances are being monitored.
Adding this feature alone does not affect Ceilometer performance/scalability.

Other deployer impact
---------------------

None

Developer impact
----------------

This feature should have minimal impact on developers for ongoing maintenance.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  * erivni

Other contributors:
  * None

Ongoing maintainer:
  * None


Work Items
----------

* Add new Pollster to Ceilometer.

* Edit setup.cfg file to support the new Pollster.

* Add Unit/Integration test coverage


Future lifecycle
================

None

Dependencies
============

None

Testing
=======

Unit and integration Tests will be added to cover the necessary pollsters calls.

Documentation Impact
====================

The Measurement docs need to be updated to reflect the new meter that is been
added.

References
==========

None
