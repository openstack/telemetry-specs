..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
Metering Network Services - VPN as a Service
=====================================================

https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-meter-vpnaas



Problem description
===================

Ceilometer currently has no support for metering VPN as a Service. Cloud
providers/Operators have the need to monitor and meter various aspects of
the Network Services. This spec deals with metering VPN as a service(VPNaaS).

Proposed change
===============

The measurements needed for metering VPNaas are categorized into two, provider
and service level. Following are the measurements targeted to be included in
Ceilometer:

* Provider level metrics:

    * Type of VPN (Openswan, Cisco etc..)
      - depends on flavor framework in neutron(See Dependencies)

* Service level metrics:

    * VPN service Status
    * Number of Connections
    * Bandwidth
      - Needs changes to neutron VPNaaS(See Dependencies)

* Metric Definitions:

===================================        ====           =============   ======
Name                                       Type           Unit            Origin
===================================        ====           =============   ======
network.services.vpn                       g              vpn             p
network.services.vpn.type                  g              vpn             p
network.services.vpn.connections           g              connections/s   p
network.services.vpn.incoming.bytes        c              B               p
network.services.vpn.outgoing.bytes        c              B               p
===================================        ====           =============   ======

g = gauge, c = cumulative, p = pollster

* Status is captured in an enum-style value in the sample volume, as opposed to
  the resource metadata, for each vpn.

The resources associated with these metrics are captured as part of resource discovery.
Neutron exposes apis to capture this data which are invoke via pollsters from the
ceilometer side. The notifications on neutron services side are a bit slim. As we
add these notification messages to the neutron side, we will enhance the ceilometer
side to capture these events through notification handlers.

For reference implemenation on neutron side, there will be an api call to retrieve
stats such as connections and bandwidth.

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

New sources need to be included in pipeline.yaml for each group of pollsters that share a
discovery extension. An example below::

    sources:
      - name: fw_source
        interval: 600
        meters:
          - "network.services.vpn"
        discovery:
          - "vpn"
        sinks:
          - network_services_sink

Similarly, we will have sources for connections and bandwidth.

Other end user impact
---------------------

The end user should be able to interact via the existing API and CLI.

* It would be good to expose these metrics on horizon dashboard, but
  that is outside the scope of this spec.


Performance/Scalability Impacts
-------------------------------

This change should not have any major impact on performance/Scalability.


Other deployer impact
---------------------

None


Developer impact
----------------

This feature should have minimal impact on developers for ongoing maintenance


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  * pkilambi

Other contributors:
  * None

Ongoing maintainer:
  * pkilambi


Work Items
----------

* Add neutron client APIs to query VPNaaS calls in neutron_client.py

* Add new Pollsters and notification handlers

* Add Unit/Integration test coverage

* Update measurement docs


Future lifecycle
================

New measurements around VPNaaS and other network services will be part of the
network pollsters and notifications. So ongoing maintenance will be handled
by the Ceilometer team, myself included.


Dependencies
============

* Flavor Framework in Neutron to determine the type of firewall.
  - https://blueprints.launchpad.net/neutron/+spec/neutron-flavor-framework
* Need statistics calls for hit counts on neutron VPNaaS side to support
  connections and bandwidth.


Testing
=======

Unit and integration Tests will be added to cover the necessary neutron_client calls,
pollsters and notifications.


Documentation Impact
====================

The Measurement docs need to be updated to reflect the new meters captured
from VPNaaS API and notifications.


References
==========

* https://etherpad.openstack.org/p/juno-summit-metering-network-services

