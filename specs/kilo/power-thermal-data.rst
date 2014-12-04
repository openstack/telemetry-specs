..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================
More Power and Thermal Data
===========================

https://blueprints.launchpad.net/ceilometer/+spec/power-thermal-data

Add more power and thermal data besides IPMI sensor data and Node Manager basic
data. These data from platform hardware are independent of OS/driver, and
valuable to show running status of node.


Problem description
===================

We already have IPMI sensor data and Node Manager basic data, but they don't
provide enough info of overall picture for server in data center. Some extra
data can be added, like CUPS(Compute Usage Per Second), which indicates
CPU/IO/Memory utilization, and Volumetric Airflow, which indicates current
amount of air that goes through server. These data plus previous basic
power/thermal data, can be used as input for Nova scheduling.


Proposed change
===============

Add capability to get new data in NodeManager class. Add new pollsters to get
CUPS and airflow data.

Add following new metrics:

 ======================================   ========         =============   =========
              Name                         Type             Unit           Origin
 ======================================   ========         =============   =========
 hardware.ipmi.airflow                        g              CFM             p
 hardware.ipmi.cups.core                      g              %               p
 hardware.ipmi.cups.io                        g              %               p
 hardware.ipmi.cups.mem                       g              %               p
 ======================================   ========         =============   =========

  * g = gauge, n = notification , p = pollster, CFM = Cubic Feet per Minute

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

These new data should be exposed via the Horizon metering dashboard.

Performance/Scalability Impacts
-------------------------------

Fetching some new metrics would not cause obvious perf drop


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
  edwin-zhai

Other contributors:
  lianhao-lu


Work Items
----------

* Add raw IPMI command to get new data in NodeManager class

* Implement 2 new pollster: CUPS and airflow

* Add unit test coverage

* Update related docs


Future lifecycle
================

Once this feature enabled, need test and bug fixing in next 2 releases to avoid
regression


Dependencies
============

This feature depends on IPMI/NM capable servers


Testing
=======

This feature with previous IPMI sensor data and NM basic data need 3rd party
testing system to verify its function. This testing system development is in
progress.


Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/developer/ceilometer/measurements.html


References
==========
None
