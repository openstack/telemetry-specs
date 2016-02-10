..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================================================
Adds disk IOPS metrics implementation in the Hyper-V Inspector
==============================================================

https://blueprints.launchpad.net/ceilometer/+spec/hyper-v-disk-iops-metrics


High latency between I/O requests can be signs of issues. Collecting disk
latency metrics can help us detect those issues. Hyper-V Inspector can collect
those metrics.

Problem description
===================

Currently, when inspecting disk metrics, the Hyper-V Inspector cannot post
any values for read_requests or write_requests, since this kind of metrics is
not supported by Hyper-V. Instead, the Hyper-V Inspector can collect disk IOPS
metrics, which will give users a useful alternative.

Proposed change
===============

Add DiskIOPSPollster and PerDeviceDiskIOPSPollster pollsters, which create
samples having the properties:
* name: 'disk.iops'
* type: 'gauge'
* unit: 'count/second'

Create the named tuple DiskIOPSStats, containing the field 'iops_count'.

Add the method 'inspect_disk_iops' in the Inspector and implement it in the
HyperVInspector. The method will return a DiskIOPSStats object.

The metric value will be fetched from Hyper-V VMs, located in the
Msvm_AggregationMetricValue object (further referred to as metric object)
associated with the VMs. The metric object's 'MetricDefinitionId' must be
equal to the 'Id' of Msvm_AggregationMetricDefinition object having the
Caption 'Average Normalized Disk Throughput'.

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

Users will have to add the meter 'disk.iops' in the disk_source source in
pipeline.yaml instead of 'disk.read.requests' and 'disk.write.requests'.

Having 'disk.read.requests' and 'disk.write.requests' in pipeline.yaml has no
effect, the HyperVInspector will return zeroes for those metrics since Hyper-V
cannot collect those metrics, hence the need for 'disk.iops' metrics.

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

None

Other deployer impact
---------------------

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
  <cbelu@cloudbasesolutions.com>

Work Items
----------

* Adds the DiskIOPSPollster and PerDeviceDiskIOPSPollster pollsters
* Adds the 'iops_count' in DiskStats.
* Adds the 'DiskIOPSStats' named tuple, having the field 'iops_count'.
* Adds the 'inspect_disk_iops' method in the Inspector and implement it in
  the HyperVInspector. This method will return a DiskIOPSStats object.
* Adds related unit tests.
* Updates ceilometer measurements document.

Future lifecycle
================

Once this feature is enabled, it needs tests and bug fixing in the next
2 releases to avoid regression.

Dependencies
============

None

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

None
