..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================================
Adds memory usage metrics implementation in the Hyper-V Inspector
=================================================================

https://blueprints.launchpad.net/ceilometer/+spec/hyper-v-memory-metrics

Currently, the Hyper-V Inspector does not collect the memory metrics. This
feature can be added, since Hyper-V can measure the amount of memory the
instances are using. This is especially useful if the instances are configured
to use dynamic memory.

Problem description
===================

Instances' memory usage metrics are very important in telemetry, but it is not
currently implemented in the Hyper-V Inspector. This spec adds memory usage
statistics to Hyper-V, so the user can get vital data on the instance's
performance.

Proposed change
===============

Implements the method 'inspect_memory_usage' of HyperVInspector, fetches the
memory stats data from Hyper-V VMs, located in the Msvm_AggregationMetricValue
objects (further referred to as 'metrics objects') associated with the VMs.
The metrics objects 'MetricDefinitionId' must be the equal to the 'Id' of
Msvm_AggregationMetricDefinition object having the Caption
'Aggregated Average Memory Utilization'.

Hyper-V metrics were introduced in Windows / Hyper-V Server 2012 (kernel
version 6.2). They are not supported in the previous versions.

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

None. By default, the memory metrics collection is enabled in Nova, we just
need to collect the data from the Hyper-V API.

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <itoader>

Work Items
----------

* Implements the method 'inspect_memory_usage' in HyperVInspector.
* Adds related unit tests.
* Updates ceilometer measurements document.

Future lifecycle
================

Once this feature is enabled, it needs tests and bug fixing in the next
2 releases to avoid regression.

Dependencies
============

* Windows / Hyper-V Server 2012 (kernel version 6.2)
* wmi 1.4.9+

Testing
=======

Unit tests are sufficient, since the implementation will only require data
fetching from Hyper-V.

Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/developer/ceilometer/measurements.html

References
==========

* [1] http://msdn.microsoft.com/en-us/library/cc768535%28v=bts.10%29.aspx
