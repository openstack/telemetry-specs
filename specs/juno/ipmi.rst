..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============
IPMI support
============

https://blueprints.launchpad.net/ceilometer/+spec/ipmi-support

Add IPMI function in ceilometer, so that power/thermal data can be fetched
through IPMI capable servers. These data can be exported to other openstack
component for other usage, like policy control.


Problem description
===================

The Intelligent Platform Management Interface (IPMI) is a standardized computer
system interface used by system administrators for out-of-band management of
computer systems and monitoring of their operation. Enabling IPMI in ceilometer
allows the cloud operator to monitor the power and thermal behaviors of servers
and make reasonable operation or policy according to the real-time power/thermal
status of data center.

With IPMI support,the power and thermal information of these servers can be used
to improve overall data center efficiency and maximize overall data center
usage. Data center managers can maximize the rack density with the confidence
that rack power budget will not be exceeded. During a power or thermal
emergency, we can limit server power consumption and send alert to
administrators via the pre-defined policy.

Currently, Ironic project is doing work to emit notifications on the message bus
containing IPMI sensor data, so that Ceilometer is possible to process these
notifications for high level usage. But for nodes that are not managed by
Ironic, IPMI data is still missing. This spec removes dependency on Ironic by
adding IPMI support in Ceilometer.

Proposed change
===============

Add a new IPMI agent, who get the power & thermal data via IPMI protocol in
periodic way and publish these data via pipeline. IPMI agent lives in each IPMI
capable node, and gets configured in deployment.

In this way, data can be fetched locally without credentials via network. Also
agent polling period can be controlled by pipeline file.

Line up metrics with IPMI support in Ironic, so we get same IPMI data regardless
of the underlying method (an agent-per-node directly emitting samples, or the
ceilometer notification agent consuming notifications emitted by Ironic. If both
available, user need deploy one only):

 ======================================   ========         =============   =========
              Name                         Type             Unit           Origin
 ======================================   ========         =============   =========
 hardware.ipmi.fan                            g              RPM             p
 hardware.ipmi.temperature                    g              C               p
 hardware.ipmi.current                        g              W               p
 hardware.ipmi.voltage                        g              V               p
 ======================================   ========         =============   =========

  * g = gauge, n = notification , p = pollster

IPMI is generic protocol, each vendor may add specific metric for their own
hardware. So there are probably more vendor specific metric in implementation.

Aim of this spec is to provide:
1) a standard metrics that exist for most of vendor
2) infrastructure to enable IPMI, that is, basic working unit to send IPMI
commands.

It could include an simple vendor specific implementation based on 2 as example,
so other vendor can do their own work easily.


Alternatives
------------

Instead of deploying IPMI agent for each node, Ceilometer itself hosts one agent
for a bunch of nodes. It poll IPMI data from each node via network in periodic
way. SNMP support in Ceilometer and IPMI in ironic take similar way.

This method avoid deploying agent in each nodes, but it introduce more work
loads in Ceilometer itself, thus bad scalability. Furthermore, IPMI via network
requires credential management.

Data model impact
-----------------

None.

REST API impact
---------------

None

Security impact
---------------

None.

Pipeline impact
---------------

None.

Other end user impact
---------------------

Power/Thermal data should be exposed via the Horizon metering dashboard.

Performance/Scalability Impacts
-------------------------------

By default, IPMI agent is not enabled until operator explicitly turn on it in
config file for IPMI capable servers. So no performance impacts by default.

Other deployer impact
---------------------

* IPMI config option(default off) need to be added

* an IPMI agent need to be added and deployed to every node, along with
  ceilometer config such as the metering secret

* If both Ironic and Ceilometer available for IPMI data, user need deploy one
  only

Developer impact
----------------

None.


Implementation
==============

Assignee(s)
-----------


Primary assignee:
  edwin-zhai

Other contributors:
  lianhao-lu
  shane-wang


Work Items
----------

* Implement an IPMI agent for data fetching deployed in each IPMI capable node.

* Add unit test coverage

* Update related docs

Future lifecycle
================

Once this feature enabled, need test and bug fixing in next 2 releases to avoid
regression


Dependencies
============

This feature depends on IPMI capable servers


Testing
=======

Unit tests are sufficient since only data fetching/exporting need test.

Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/developer/ceilometer/measurements.html

The new configuration for deployment need to be documented.

IPMI agent shouldn't be deployed if a node is managed with Ironic, which need to
be documented.

References
==========

http://www.intel.com/content/www/us/en/servers/ipmi/ipmi-home.html


