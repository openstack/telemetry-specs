..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Track DNS Service Notifications
====================================

https://blueprints.launchpad.net/ceilometer/+spec/dns-service-notifications

The goal of this blueprint is to capture the notification events emitted by Designate
(DNSaaS) during create, update and delete operations.

Problem description
===================

Designate (designate-central) emits notifications in response to user actions by
designate-api and other openstack service (nova, neutron) events by designate-sink.

These notifications are CRUD events, that notify the creation, update or delete
of a DNS resource, with no specific measurement value.

Ceilometer needs to be updated to consume and record these notifications as events
and traits.

If Ceilometer processes and record these notifications, other services will be
able to use this for aggregation and billing purpose.


Proposed change
===============

Define event definitions and field mappings(trait definitions) for DNS resource types
- domain, record in the /etc/ceilometer/event_definitions.yaml file.

The trait field syntax in trait definitions follow a variant of the JSONPath.

Example of the event definition and field mappings for the domain notifications
that need to go in event_definitions.yaml,

-  event_type: dns.domain.*
-  traits: &dns_domain_traits
-    status:
-      fields: payload.status
-    retry:
-      fields: payload.retry
-    description:
-      fields: payload.description
-    expire:
-      fields: payload.expire
-    email:
-      fields: payload.email
-    ttl:
-      fields: payload.ttl
-    action:
-      fields: payload.action
-    name:
-      fields: payload.name
-    id:
-      fields: payload.id
-    created_at:
-      fields: payload.created_at
-    updated_at:
-      fields: payload.updated_at
-    version:
-      fields: payload.version
-    parent_domain_id:
-      fields: parent_domain_id
-    serial:
-      fields: payload.serial

To enable event processing in Ceilometer, configuration is needed in ceilometer.conf,

* Configure store_events as True under the notification section,

eg -
[notification]
store_events = True

* Specify the event_definitions file name under the event section,

eg -
[event]
definitions_cfg_file = event_definitions.yaml


The dns event types have the following pattern,

* dns.<resource>.create
* dns.<resource>.update
* dns.<resource>.delete

, where resource is domain, record, pool.

Alternatives
------------

Transform DNS notifications as samples using notification plugin to listen for notifications
at an exchange and topic. This could be useful when the notifications represent something other
than CRUD events that contain a value of interest that can be measured.

Using events provides better querying of metadata and avoids the effort to aggregate samples
with a fixed measurement value of 1.


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
  rjaiswal

Ongoing maintainer:
  rjaiswal

Work Items
----------

* Event definitions and field mappings for all supported DNS resource types.

* Event validation.

Future lifecycle
================

In the future Designate will emit the dns.domain.exists event. This will need to be
handled when designate emits this notification.

The set of dns resource types - (domain, record, pool) may change going
forward. There could be new types of notifications which will need to be supported.

When Designate confirms with the PaaS event format, the field mappings for integrated
events will need to be refactored to adjust to the new event format.

Dependencies
============

DNSaaS is construed as a PaaS service by some and might have it own message bus
where it is sending notifications. Ceilometer recently was extended to consume
notification from multiple message bus - https://review.openstack.org/#/c/77612/

If there are multiple message buses, then ceilometer will need multiple transport
endpoints configured in ceilometer configuration. (ceilometer.conf)

Testing
=======

Unit and integration Tests will be added to validate events generated.

Documentation Impact
====================

None.

References
==========

http://docs.openstack.org/developer/ceilometer/events.html

http://docs.openstack.org/trunk/config-reference/content/ch_configuring-openstack-telemetry.html

http://specs.openstack.org/openstack/ceilometer-specs/specs/juno/paas-event-format-for-ceilometer.html

https://wiki.openstack.org/wiki/Designate

https://github.com/kennknowles/python-jsonpath-rw