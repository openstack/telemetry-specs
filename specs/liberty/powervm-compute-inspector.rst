..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=========================
PowerVM Compute Inspector
=========================

https://blueprints.launchpad.net/ceilometer/+spec/powervm-inspector

PowerVM is a hypervisor for the POWER processor architecture.  Community
drivers to support PowerVM in an OpenStack environment for Nova and
Neutron (ML2 agent) are in active development.  Telemetry is a key component
for operators within an OpenStack environment.  This blueprint proposes that
an inspector for PowerVM be included in Ceilometer.

Problem description
===================

OpenStack drivers delivering support for the PowerVM hypervisor are being
developed for Nova and Neutron (an ML2 agent).  Operators require more
support than Nova and Neutron integration - they need telemetry information.
PowerVM provides a variety of performance monitoring interfaces for both
virtual machine and system monitoring metrics.

This proposal is for the PowerVM Driver Team to add a PowerVM compute
inspector to the Ceilometer project.  This will allow operators receive and
store Telemetry data for PowerVM in the same fashion as other hypervisors.

Proposed change
===============

We propose that a new python package be created:
ceilometer/compute/virt/powervm.

This package will contain a compute inspector (similar to those for hyperv,
libvirt, vmware, and xen) that gathers the statistics.

The collection of these statistics will be driven through the pypowervm API.
This is an open source project that allows python programs to interact with the
PowerVM hypervisor.

The module within pypowervm that will be used is the LPAR (logical partition,
equivalent to a virtual machine) statistics package.  This will provide the
interfaces used to collect the statistics.

The metrics reported for a given instance that will be included are:

* CPU Statistics
* CPU Utilization
* vNIC Statistics
* vNIC Utilization
* Disk Statistics
* Disk Rates
* Disk IOPs

Metrics that will not be included are:

* Disk Latency
* Disk Info


Alternatives
------------

An alternative would be to keep the inspector in a separate StackForge project.
This was not seen as viable due to how the inspectors are derived.  Ceilometer
depends on the inspectors residing in the main ceilometer.compute.virt project.

This approach could be supported if the get_hypervisor_inspector method was
extended to support external projects.

Data model impact
-----------------

None.

REST API impact
---------------

None.

Security impact
---------------

None.

Pipeline impact
---------------

None.

Other end user impact
---------------------

The inspector should be running on the PowerVM host system in order to collect
the information.  This is in line with the Nova compute driver and the
Neutron ML2 agent.

Performance/Scalability Impacts
-------------------------------

None.


Other deployer impact
---------------------

The hypervisor_inspector will have a new option available called 'powervm'.
No database impacts would occur as the standard statistics would be supported.

There will be a Chef blueprint that is being developed and worked with that
team to support the deployment of the PowerVM drivers/agents.  The changes
there would support the use of the Ceilometer inspector for PowerVM.

Developer impact
----------------

None.  Unit tests would be developed for the new inspector but are contained
within the given package.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  thorst

Other contributors:
  adreznec, efried

Ongoing maintainer:
  thorst, efried

Work Items
----------

* Deliver the package structure for the powervm inspector.
* Develop the CPU Statistics method and unit tests.
* Develop the CPU Utilization method and unit tests.
* Develop the vNIC Statistics method and unit tests.
* Develop the vNIC Utilization method and unit tests.
* Develop the Disk Statistics method and unit tests.
* Develop the Disk Rates method and unit tests.
* Develop the Disk IOPs method and unit tests.


Future lifecycle
================

The PowerVM Drivers team that resides within IBM will continue maintenance and
support of the inspector for future releases.  The PowerVM Drivers team is
outlined here:  https://launchpad.net/~powervm-drivers

The maintenance and support that will be provided is, at a minimum:

* Changes in the PowerVM inspector due to base library changes
* Ongoing testing and validation of the PowerVM inspector
* Bug fixes identified within the PowerVM inspector


Dependencies
============

* pypowervm: https://github.com/pypowervm/pypowervm


Testing
=======

The existing tempest tests will be run against the PowerVM inspector. The
tempest tests are hypervisor-agnostic, allowing the existing tempest tests to
be run against the PowerVM polling code without changes.

The PowerVM Drivers team will directly run the tempest tests against a
PowerVM system (using the Nova PowerVM Driver and Neutron PowerVM ML2 Agent).


Documentation Impact
====================

The configuration reference should include an update for the PowerVM inspector:
http://docs.openstack.org/kilo/config-reference/content/ch_configuring-openstack-telemetry.html


References
==========

Python library for PowerVM:

* pypowervm: https://github.com/pypowervm/pypowervm

Corresponding OpenStack Projects:

* nova-powervm: https://launchpad.net/nova-powervm
* neutron-powervm: https://launchpad.net/neutron-powervm

Note: These are StackForge projects while we iterate with the core teams to
bring them into the core.  Neutron will stay a StackForge project given the
decomposition goals of Neutron itself.

PowerVM as a Hypervisor:

* Hypervisor Landing Page: http://www-03.ibm.com/systems/power/software/virtualization/
* Hypervisor Documentation: http://www.redbooks.ibm.com/abstracts/sg247940.html
* Hypervisor Best Practices: http://www.redbooks.ibm.com/abstracts/sg248062.html
