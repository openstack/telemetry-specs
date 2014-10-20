..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================================
Adds memory utilization meter to libvirt inspector
==================================================

https://blueprints.launchpad.net/ceilometer/+spec/libvirt-memory-utilization-inspector

Memory usage statistics is not implemented in libvirt inspector. We can get
memory stats of the instance from libvirt API 'virDomainMemoryStats' in order
to add memory usage meter to libvirt inspector.

Problem description
===================

Memory usage of instance is very important data in telemetry, but now it is not
implemented in libvirt inspector. This spec adds memory usage statistics to
libvirt, so the user can get the data on the performance of the instance.

Proposed change
===============

Implements the method 'inspect_memory_usage' of LibvirtInspector, fetches the
memory stats data from libvirt API 'virDomainMemoryStats', used memory is
calculated by the available and unused memory. The libvirt API
'virDomainMemoryStats' may raise an exception if the method is not supported by
libvirt, refer to 'Dependencies' section, catches the exception and translates
that into an empty data of memory stats.

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

User need to prepare suitable balloon driver in image, particularly for Windows
guests, most modern Linuxes have it built in. Booting instance will be
successful without image balloon driver, just can't get guest memory usage
meter.

Performance/Scalability Impacts
-------------------------------

None

Other deployer impact
---------------------

None. By default, the memory statistical feature is enabled in Nova, refer to
[1], we just fetch and collect the data from libvirt API.

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <kiwik-chenrui>

Work Items
----------

* Implements the method 'inspect_memory_usage' of LibvirtInspector.
* Adds relate unit tests.
* Updates ceilometer measurements document.


Future lifecycle
================

Once this feature enabled, need test and bug fixing in next 2 releases to avoid
regression.


Dependencies
============

* libvirt 1.1.1+
* qemu 1.5+
* guest driver that supports memory balloon stats


Testing
=======

Unit tests are sufficient since only data fetching need test.


Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/developer/ceilometer/measurements.html


References
==========

* [1] https://blueprints.launchpad.net/nova/+spec/enabled-qemu-memballoon-stats
* [2] http://libvirt.org/html/libvirt-libvirt.html#virDomainMemoryStats

