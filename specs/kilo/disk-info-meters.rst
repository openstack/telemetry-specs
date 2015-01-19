..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================================================
Adds Disk Capacity, Allocation and Usage metrics implementation in Libvirt
==========================================================================

https://blueprints.launchpad.net/ceilometer/+spec/disk-info-meters

Problem description
===================

Currently, Libvirt Inspector does not collect disk capacity, usage and
allocation metrics. This feature can be added for Libvirt to monitor disk
metrics : capacity, usage and allocation. It will help to manage disk
resources for addition of Virtual Machinesusing same physical disk.

Proposed change
===============

Implement the method 'inspect_disk_info' in Inspector for Libvirt. This method
will return the disk capacity,usage and allocation per device. The values
correspond to values monitored by domblkinfo command of Libvirt.

Add CapacityPollster which create sample :

* name='disk.capacity'
* type=sample.TYPE_GAUGE
* unit='B'

Add PerDeviceCapacityPollster which create sample :

* name='disk.device.capacity'
* type=sample.TYPE_GAUGE
* unit='B'

Add AllocationPollster which create sample :

* name='disk.allocation'
* type=sample.TYPE_GAUGE
* unit='B'

Add PerDeviceAllocationPollster which create sample :

* name='disk.device.allocation'
* type=sample.TYPE_GAUGE
* unit='B'

Add PhysicalPollster which create sample :

* name='disk.usage'
* type=sample.TYPE_GAUGE
* unit='B'

Add PerDevicePhysicalPollster which create sample :

* name='disk.device.usage'
* type=sample.TYPE_GAUGE
* unit='B'


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

In pipeline.yaml, if user is not having * in meters_sink
then new meters disk.capacity, disk.allocation, disk.usage,
disk.device.capacity, disk.device.usage and disk.device.allocation
needs to be added

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ahmed-nooras-saba


Work Items
----------

* Add _DiskInfoPollsterBase base class for reading values and aggregating
  for all devices
* Add CapacityPollster , AllocationPollster , PhysicalPollster for disk
  metrics at instance level
* Add PerDeviceCapacityPollster , PerDeviceAllocationPollster and
  PerDevicePhysicalPollster for disk metrics at device level of an instance
* Add inspect_disk_info to Libvirt inspectors
* Add new metrics definition in measurements.rst


Future lifecycle
================

This feature will need testing for next two releases.


Dependencies
============

libvirt 0.8.1 and above.


Testing
=======

Unit tests are required to test all the new pollsters at Libvirt.
The new meters should be discoverable when listing ceilometer meters.
Example : ceilometer meter-list


Documentation Impact
====================

Added the following metrics in ceilometer/doc/source/measurements.rst and
"measurement section" of
http://docs.openstack.org/developer/ceilometer/measurements.html needs to
be updated.

====================== = = ======= = = ===================================
====================== = = ======= = = ===================================
disk.capacity          g B inst ID n 1  Capacity of disk in B
disk.allocation        g B inst ID n 1  Allocation of disk in B
disk.usage             g B inst ID n 1  Usage of disk in B
disk.device.capacity   g B disk ID n 1  Capacity per device of disk in B
disk.device.allocation g B disk ID n 1  Allocation per device of disk in B
disk.device.usage      g B disk ID n 1 Usage per device of disk in B
====================== = = ======= = = ===================================


References
==========

http://osdir.com/ml/libvir-list/2010-04/msg01300.html
