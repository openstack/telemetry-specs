..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================================================
Metering Network Services - LoadBalancer as a Service
=====================================================

https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-meter-lbaas



Problem description
===================

Ceilometer currently has no support for metering Network Services. Cloud
providers/Operators have the need to monitor and meter various aspects of
the Network Services. This spec deals with metering Load Balancer as a
service(LBaaS).

Proposed change
===============

The measurements needed for metering LBaas are categorized into two, provider
and service level. Following are the measurements targeted to be included in
Ceilometer:

* Provider level metrics:

    * Type of LB ( HAproxy, VPx etc..)

    * Features (SSL, Non-SSL, stickiness etc..)

* Service level metrics:

    * LB pool
        * Status
        * Number of Connections
          - Use statistics API call from LBaaS side
        * Bandwidth
          - Use statistics API call from LBaaS side

    * Members
        * Status
        * Bandwidth
          - Use statistics API call from LBaaS side

    * Health monitors
        * Status of the health probe

* Metric Definitions:

 ======================================   ========         =============   =========
              Name                         Type             Unit           Origin
 ======================================   ========         =============   =========
 network.services.lb.type                     g              type            p
 network.services.lb.pool                     g              pool            p
 network.services.lb.vip                      g              vip             p
 network.services.lb.member                   g              member          p
 network.services.lb.health_monitor           g              monitor         p
 network.services.lb.total.connections        g              connection      p
 network.services.lb.active.connections       g              connection      p
 network.services.lb.incoming.bytes           c              B               p
 network.services.lb.outgoing.bytes           c              B               p
 ======================================   ========         =============   =========

  * g = gauge, c = cumulative, p = pollster

* Status is captured in an enum-style value in the sample volume, as opposed to
  the resource metadata, for each pool, vip, members and monitor.

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

* Add neutron client APIs to query LBaas calls in neutron_client.py

* Add new Pollsters and notification handlers

* Add Unit/Integration test coverage

* Update measurement docs


Future lifecycle
================

New measurements around LBaaS and other network services will be part of the
network pollsters and notifications. So ongoing maintenance will be handled
by the Ceilometer team, myself included.


Dependencies
============

None


Testing
=======

Unit and integration Tests will be added to cover the necessary neutron_client calls, pollsters and notifications.


Documentation Impact
====================

The Measurement docs need to be updated to reflect the new meters captured
from LBaaS API and notifications.


References
==========

* https://etherpad.openstack.org/p/juno-summit-metering-network-services

