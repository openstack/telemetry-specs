..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Track DBaaS(Trove) Notifications
====================================

https://blueprints.launchpad.net/ceilometer/+spec/track-dbaas-notifications

The goal of this blueprint is to capture the notification events emitted by Trove
(DBaaS) during create, modify volume, resize, delete operations and periodic exist
events.


Problem description
===================

Trove is Database as a Service for OpenStack. It automates the creation, configuration
and management of relational or non-relational database without the burden of handling
complex administrative tasks.

The trove-taskmanager service dispatches tasks such as provisioning database instances,
managing the lifecycle of database instances, and performing operations on the database
instance and emits notification event for each such task.

These notifications are CRUD events, that notify the creation, update or modification
of a trove instance and its underlying volume, and also have measurement values such
as instance size and volume size.

Ceilometer needs to be updated to consume and record these notifications as samples and
events (traits).

From Trove perspective they are running instances of a database, not compute instances.
Metering and billing will be based on hours that the database has been running, not
necessarily how long the hosting instance has been run. This requires a unique set of
metering records(samples) to be generated and stored to enable usage tracking and billing
of the individual database instances.

If Ceilometer processes and record these notifications, other services will be
able to use this for aggregation and billing purpose.


Proposed change
===============

To represent as events, define event definitions and field mappings
(trait definitions) for Trove resource type - instance in the
/etc/ceilometer/event_definitions.yaml file.

The trait field syntax in trait definitions follow a variant of the JSONPath.

Example of the event definition and field mappings for the domain notifications
that need to go in event_definitions.yaml,

-  event_type: trove.instance.*
-  traits: &trove_traits
-    instance_size:
-      fields: payload.instance_size
-    resource_id:
-      fields: payload.instance_id
-    instance_name:
-      fields: payload.instance_name
-    nova_instance_id:
-      fields: payload.nova_instance_id
-    created_at:
-      fields: payload.created_at
-    launched_at:
-      fields: payload.launched_at
-    volume_size:
-      fields: payload.volume_size
-    volume_id:
-      fields: payload.nova.volume_id
-    modify_at:
-      fields: payload.modify_at
-    deleted_at:
-      fields: payload.deleted_at

To enable event processing in Ceilometer, configuration is needed in ceilometer.conf,

* Configure store_events as True under the notification section,

eg -
[notification]
store_events = True

* Specify the event_definitions file name under the event section,

eg -
[event]
definitions_cfg_file = event_definitions.yaml

Using events provides better querying of metadata and avoids the effort to aggregate
samples with a fixed measurement value of 1.


To represent as samples, a notification plugin is required in Ceilometer,
to hear notifications at an exchange and topic and transform them into samples.

This is needed to account for the database uptime hours, which is based on
the trove.instance.exists event. Specifically, the audit_period_beginning
and audit_period_ending for each exists event will be used to compute the uptime
hours

A counter type - database can be used to associate the trove samples

Add a new listener class - TroveMetricsNotificationBase that extends
plugin.NotificationBase for DBaaS service to transform Trove event data
into samples

A sub-class of TroveMetricsNotificationBase that emits samples
for the trove.instance.create and trove.instance.exists events and
another for the trove.instance.modify* and trove.instance.delete events

The Trove event types have the following pattern,

* trove.<resource>.create
* trove.<resource>.modify_volume
* trove.<resource>.modify_flavor
* trove.<resource>.delete
* trove.<resource>.exists

, where resource is instance.

Alternatives
------------


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

Enable Trove exchange in ceilometer.conf

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

* Event definitions and field mappings for all supported Trove resource types.

* Event validation.

* Trove Notifications Listener class to process create/update/delete/exists events for
  trove instance.

* Test coverage for the above listener class and sample validation.

Future lifecycle
================

There could be new types of notifications when new resource types are added
or new features are supported like backup/restore, patching, monitoring etc.

When Trove conforms to the PaaS event format, the field mappings for integrated
events will need to be refactored to adjust to the new event format.

With the impending move towards declarative metering in Ceilometer, the proposed
notification plugin for Trove will become redundant/deprecated and replaced with
declarative meter definitions.

Dependencies
============

DBaaS is construed as a PaaS service by some and might have it own message bus
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

https://wiki.openstack.org/wiki/Trove/trove-notifications

https://github.com/kennknowles/python-jsonpath-rw