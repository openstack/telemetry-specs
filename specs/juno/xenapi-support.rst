..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
XenAPI support
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/xenapi-support

Currently ceilometer can support libvirt, hyperv and vmware hypervisor.
And now it is necessary to add xenapi inspector for XenServer/Xen Cloud
Platform.

Problem description
===================

The usecase is to use xenapi to inspect the XenServer hypervisor.

Proposed change
===============

Create xenapi directory in ceilometer/compute/virt, and implement
xenapi inspector to support all existing meters.

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

None

Performance/Scalability Impacts
-------------------------------

None

Other deployer impact
---------------------

The deployer can now optionally define connection information to
XenServer/Xen Cloud Platform by adding::

    [xenapi]
    connection_url =
    connection_username =
    connection_password =

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  * qiaowei-ren

Ongoing maintainer:
  * qiaowei-ren

Work Items
----------

* Implement xenapi inspector for XenServer/Xen Cloud Platform.
* Test method in unit tests

Future lifecycle
================

None

Dependencies
============

None

Testing
=======

Unit Tests will be added to cover the necessary inspector calls.

Documentation Impact
====================

The Measurement docs need to be updated to reflect xenapi support.

References
==========

XenAPI Documentation: http://docs.vmd.citrix.com/XenServer/6.2.0/1.0/en_gb/api/

