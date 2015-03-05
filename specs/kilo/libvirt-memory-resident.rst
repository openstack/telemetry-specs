..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================================
Adds resident memory size meter to libvirt inspector
====================================================

https://blueprints.launchpad.net/ceilometer/+spec/memory-resident

Memory statistics include just the actual usage inside the Virtual Machine.
The 'memory.resident' metric will add the resident memory size. The
implementation is done in the libvirt inspector. The value will be fetched
using the libvirt API and the 'rss' for dommemstats.

Problem description
===================

Resident memory is important to see the amount of memory consumed by the
Virtual Machine from the Physical Host. This spec adds the resident memory
statistics to the libvirt inspector.

Proposed change
===============

Implements the method 'inspect_memory_resident' of LibvirtInspector, fetches the
rss value for memory from libvirt API 'virDomainMemoryStats'. The libvirt API
'virDomainMemoryStats'.

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

None

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  vivek-nandavanam

Work Items
----------

* Implements the method 'inspect_memory_resident' of LibvirtInspector.
* Updates ceilometer measurements document.


Future lifecycle
================

Once this feature enabled, need test and bug fixing in next 2 releases to avoid
regression.


Dependencies
============

* libvirt 0.7.5


Testing
=======

Unit tests should be sufficient.


Documentation Impact
====================

The added metrics will need to be documented in https://github.com/openstack/openstack-manuals/blob/master/doc/admin-guide-cloud/telemetry/tables/ceilometer-measurements-nova.xml which will get reflected in the cloud admin documenation http://docs.openstack.org/admin-guide-cloud/content/section_telemetry-compute-metrics.html


References
==========

* [1] https://blueprints.launchpad.net/nova/+spec/memory-resident
* [2] http://libvirt.org/html/libvirt-libvirt.html#virDomainMemoryStats

