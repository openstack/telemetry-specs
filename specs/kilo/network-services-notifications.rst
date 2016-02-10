..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Track Network Services Notifications
====================================

https://blueprints.launchpad.net/ceilometer/+spec/network-services-notifications

The goal of this blueprint is to capture the notification events emitted by neutron
network services during create, update and delete operations.

Problem description
===================

Neutron emits notifications from the data provided by the network services plugin
and agent components. These in particular are for FWaaS, LBaaS and VPNaaS. Ceilometer
needs to be updated to consume and record these notifications. If Ceilometer processes
these notifications other services will be able use queries and alarms to monitor and
scale as required.

A notification plugin is required, in Ceilometer, to hear notifications at an exchange
and topic and transform them into samples. We can leverage the existing Network Notification
plugins and build on top of it.

Proposed change
===============

Add a new listener classes for Network Services to transform neutron event data into samples,
using existing ``NetworkNotificationBase`` implementations as a model. These listeners
include create, update and delete events emitted by neutron for specific components of
firewall, VPN or a Loadbalancer.

The Neutron event topic names have the following pattern,

* <resource>.create.start
* <resource>.create.end
* <resource>.update.start
* <resource>.update.end
* <resource>.delete.start
* <resource>.delete.end

For this implementation we will track the end events as those give us the most information
with respect to event payload. Also end events are more informative to track the existence
of a resource and its usage over time.

Alternatives
------------

None.

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

None.

Performance/Scalability Impacts
-------------------------------

No new impacts. Pre-existing concerns with capacity at the notification and
storage handling layers remain.

Other deployer impact
---------------------

None.

Developer impact
----------------

None.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  pkilambi

Ongoing maintainer:
  pkilambi

Work Items
----------

* Load Balancer Listener class to process create/update/delete events for
  pool, vip, member, health_monitor.

* Firewall Listener class to process create/update/delete events for
  firewall, policy and rule.

* VPN listener class to process create/update/delete events for
  vpnservice, ipsec_policy, ike_policy, ipsec_site_connections.

* Test coverage for the above listener classes and sample validation.

Future lifecycle
================

In the future new types of notifications are expected from the Neutron services to
account for the statistics data. These will need to be handled separately.

Dependencies
============

None.

Testing
=======

Unit and integration Tests will be added to cover the necessary notification listener
classes and validate samples generated.

Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/developer/ceilometer/measurements.html

References
==========

.. _In Progress Implementation: https://review.openstack.org/#/c/121686/
