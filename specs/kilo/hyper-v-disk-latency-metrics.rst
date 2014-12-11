..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================================
Adds disk latency metrics implementation in the Hyper-V Inspector
=================================================================

https://blueprints.launchpad.net/ceilometer/+spec/hyper-v-disk-latency-metrics

High latency between I/O requests can be signs of issues. Collecting disk
latency metrics can help us detect those issues. Hyper-V Inspector can collect
those metrics.

Problem description
===================

Disk latency metrics are important in telemetry and useful when determining
instance's performance. This spec is about these stats and their implementation
in the Hyper-V Inspector.

Proposed change
===============

Add DiskLatencyPollster and PerDeviceDiskLatencyPollster pollsters, which
create samples having the properties:
* name: 'disk.latency'
* type: 'gauge'
* unit: 'ms'

Add the method 'inspect_disk_latency' in Inspector and implementing it in the
HyperVInspector, fetching disk latency stats data from Hyper-V VMs, located
in the Msvm_AggregationMetricValue objects (further referred to as
'metrics objects') associated with the VMs.
The metrics objects 'MetricDefinitionId' must be the equal to the 'Id' of
Msvm_AggregationMetricDefinition object having the Caption
'Average Disk Latency'.

Hyper-V disk metrics were introduced in Windows / Hyper-V Server 2012 R2
(kernel version 6.3). They are not supported in the previous versions.

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

Users will have to add meter 'disk.latency' in the disk_source source in
pipeline.yaml
By default, the disk usage metrics collection is enabled in Nova, we just need
to collect the data from the Hyper-V API.

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <cbelu>

Work Items
----------

* Adds the DiskLatencyPollster and PerDeviceDiskLatencyPollster pollsters
* Adds the method 'inspector_disk_latency' in Inspector.
* Implements the method 'inspect_disk_latency' in HyperVInspector.
* Adds related unit tests.
* Updates ceilometer measurements document.

Future lifecycle
================

Once this feature is enabled, it needs tests and bug fixing in the next
2 releases to avoid regression.

Dependencies
============

* Windows / Hyper-V Server 2012 R2 (kernel version 6.3)
* wmi 1.4.9+

Testing
=======

Unit tests are needed to test the new pollsters and the implementation on
the Hyper-V side.

Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/developer/ceilometer/measurements.html

References
==========

* [1] http://msdn.microsoft.com/en-us/library/cc768535%28v=bts.10%29.aspx
