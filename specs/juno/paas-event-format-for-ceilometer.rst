..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================
PaaS Event Format
=================


https://blueprints.launchpad.net/ceilometer/+spec/paas-event-format-for-ceilometer

The general mechanism for setting up Ceilometer collection for a new service
has been to either create a targeted agent or use the existing service APIs to
meter these systems. While this approach has worked for the existing set of
services, it will run into a several problems as the number of PaaS and SaaS
services that need to be metered expand. This blueprint provides a proposal for
streamlining the format of integrations so that Ceilometer can consume
metering data from any conforming PaaS application without doing any custom
integration.


Problem description
===================

There are a number of PaaS services that are currently in development
and a growing number of applications running on top of OpenStack
infrastructure. Many of these new applications will need to be metered
for a variety of reasons: internal systems management, license
management, end customer billing. Ceilometer currently supports the
collection of metering information for many infrastructure-level services,
however it won't be viable to do custom integrations for the
potentially hundreds (and eventually thousands) of services that may be
hosted in OpenStack. We want to head off that issue by defining a
universal PaaS notification payload so Ceilometer can consume PaaS
notifications without significant integration work.

A few use cases come to mind:
**Database as a Service**

Database as a Service (DBaaS), i.e. Trove, has an architecture where a service
controller manages special Nova compute instances on behalf of the customer.
From their perspective they are running instances of a database, not compute
instances. Metering and billing will be based on hours that the database has
been running, not necessarily how long the hosting instance has been run. This
requires a unique set of metering records to be generated and stored to enable
usage tracking and billing of the individual database instances.

**DNS as a Service**

DNS as a service runs on a similar service controller architecture to DBaaS. In
this case, the meters that need to be measured are the existence of a DNS zone
and the number of DNS queries that have been served. Once again these are
application level meters that do not correlate directly to the host instances
that are running. Queries could be served by a variety of hosts and a DNS
zone's existence does not depend on a specific compute instance. Application
level metrics must be tracked by the DNS service and reported out so that these
systems can be tracked.

**Load Balancing as a Service**

A load balancer is a logical system that consists of some number of independent
backends that host the load balancing software. Meters that must be measured
would be things like the existence of a load balancer VIP, its members, and the
amount of data that is being transferred through the system. Once again, this
does not directly correlate to the underlying infrastructure but must be
reported at the application level


Proposed change
===============

We need to standardize the content of the PaaS payload to include a set of data
that will include data needed for metering and billing. This essentially
constitutes a documentation change and a set of tests to verify key fields.

The following is the proposed schema:
::

   [
    {
        "Field": "event_type",
        "Type": "enumeration",
        "Description": "for event type records, this describes the actual event that occurred",
        "Compliance": "required for events",
        "Notes": "depends on service, defaults to create, exists, delete"
    },
    {
        "Field": "timestamp",
        "Type": "UTC DateTime",
        "Description": "timestamp of when this event was generated at the resource",
        "Compliance": "required",
        "Notes": "ISO 8859 date YYYY-mm-ddThh:mm:ss"
    },
    {
        "Field": "message_id",
        "Type": "String",
        "Description": "unique identifier for event",
        "Compliance": "required",
        "Notes": ""
    },
    {
        "payload": [
        {
            "Field": "version",
            "Type": "String",
            "Description": "Version of event format",
            "Compliance": "required",
            "Notes": ""
        },
        {
            "Field": "audit_period_beginning",
            "Type": "UTC DateTime",
            "Description": "Represents start time for metrics reported",
            "Compliance": "required",
            "Notes": "Format ISO 8859 date YYYY-mm-ddThh:mm:ss"
        },
        {
            "Field": "audit_period_ending",
            "Type": "UTC DateTime",
            "Description": "Represents end time for metrics reported",
            "Compliance": "required",
            "Notes": "Format ISO 8859 date YYYY-mm-ddThh:mm:ss"
        },
        {
            "Field": "record_type",
            "Type": "enumeration ",
            "Values": {
                "event": "events describe some kind of state change in the service",
                "quantity": "quantity describes a usage metric value"
            },
            "Compliance": "required",
            "Notes": ""
        },
        {
            "Field": "project_id",
            "Type": "UUID",
            "Description": "Keystone project_id identifies the owner of
                            the service instance",
            "Compliance": "required",
            "Notes": ""
        },
        {
            "Field": "user_id",
            "Type": "UUID",
            "Description": "Keystone user_id identifies specific user",
            "Compliance": "optional",
            "Notes": ""
        },
        {
            "Field": "service_id",
            "Type": "UUID",
            "Description": "Keystone service_id uniquely identifies a service",
            "Compliance": "required",
            "Notes": ""
        },
        {
            "Field": "service_type",
            "Type": "String",
            "Description": "Keystone service_type uniquely identifies a service",
            "Compliance": "required",
            "Notes": ""
        },
        {
            "Field": "instance_id",
            "Type": "UUID",
            "Description": "uniquely identifies an instance of the service",
            "Compliance": "required",
            "Notes": "assuming instance level reporting"
        },
        {
            "Field": "display_name",
            "Type": "String",
            "Description": "text description of service",
            "Compliance": "optional",
            "Notes": "used if customer names instances"
        },
        {
            "Field": "instance_type_id",
            "Type": "enumeration",
            "Description": "used to describe variations of a service",
            "Compliance": "required",
            "Notes": "needed if variations of service have different prices or
                      need to be broken out separately"
        },
        {
            "Field": "instance_type",
            "Type": "String",
            "Description": "text description of service variations",
            "Compliance": "optional",
            "Notes": ""
        },
        {
            "Field": "availability_zone",
            "Type": "String",
            "Description": "where the service is deployed",
            "Compliance": "optional",
            "Notes": "required if service is deployed at an AZ level"
        },
        {
            "Field": "region",
            "Type": "String",
            "Description": "data center that the service is deployed in",
            "Compliance": "optional",
            "Notes": "required if service is billed at a regional level"
        },
        {
            "Field": "state",
            "Type": "enumeration",
            "Description": "status of the service at the time of record generation",
            "Compliance": "optional",
            "Notes": "required for existence events"
        },
        {
            "Field": "state_description",
            "Type": "String",
            "Description": "text description of state of service",
            "Compliance": "",
            "Notes": ""
        },
        {
            "Field": "license_code",
            "Type": "enumeration",
            "Description": "value that describes a specific license model",
            "Compliance": "optional",
            "Notes": "this field is TBD depending on dev_pay design work"
        },
            {
                "metrics": [
                    {
                        "Field": "metric_name",
                        "Type": "String",
                        "Description": "unique name for the metric that is represented
                         in this record",
                        "Compliance": "required",
                        "Notes": ""
                    },
                    {
                        "Field": "metric_type",
                        "Type": "enumeration",
                        "Description": "gauge, cumulative, delta",
                        "Compliance": "required",
                        "Notes": "describes the behavior of the metric, from ceilometer"
                    },
                    {
                        "Field": "metric_value",
                        "Type": "Float",
                        "Description": "value of metric for quantity type records",
                        "Compliance": "required for quantities",
                        "Notes": ""
                    },
                    {
                        "Field": "metric_units",
                        "Type": "enumeration",
                        "Description": "describes the units for the quantity",
                        "Compliance": "optional",
                        "Notes": ""
                    }
                ]
            }
        ]
    }
  ]


Note: Required means that it must be present and described as in the specification.
Optional means it can be present or not, but if present it must be described as
in the specifications.


Alternatives
------------

The alternative approach to this standardization is to allow PaaS service
providers to determine the content of the notifications and the associated
plugins, requiring missing data to be addressed post implementation through
patching.


Data model impact
-----------------

* What new data objects and/or database schema changes is this going to
  require?

This format should fit within the existing schema.


* What database migrations will accompany this change, treating the SQLAlchemy
  and NoSQL cases separately.

No DB migrations will be required.

* Will this add to the on-the-fly data massaging cruft that we've accreted
  over time?

The purpose of this change is to avoid the explosion of cruft that could
potentially be generated as PaaS services implement variations on the
notification payload.


REST API impact
---------------

The API should transparently handle the PaaS data, there should be no API
impact.


Security impact
---------------
As this BP only specifies a standard format for PaaS samples (and one that
closely resembles the current sample content) current content of Ceilometer
samples) there is no security impact.


Pipeline impact
---------------

The implementation of the PaaS collection agent is documented in a separate BP;
this standardization requires no pipeline changes in itself.


Other end user impact
---------------------

None.


Performance/Scalability Impacts
-------------------------------

None.


Other deployer impact
---------------------

There should be a positive impact to deployers from implementing this standard:
unifying the PaaS content up-front will reduce the likelihood of missing data
required for downstream processing.


Developer impact
----------------

There is the increase in up-front effort to define, for each PaaS service being
implemented, a set of data that fulfills the standard. There is also an
implication that there may be evolution of this standard, and therefore
evolution of the documentation/tests.


Implementation
==============

Assignee(s)
-----------

This is tied to the work documented in the PaaS Event Collection blueprint and
will be implemented in parallel with that. That said, there's certainly benefit
in opening this up to those who have an interest in PaaS events, especially if
there are data elements that we haven't considered here.

Primary assignee:
 nealph (phil.neal@hp.com)

Other contributors:
 fabiog (fabio.gianetti@hp.com)

Ongoing maintainer:
 nealph (phil.neal@hp.com)

It's likely that the standard will expand: the intent here is to define and
clearly document the core elements required. For that reason, I expect
contributions to this documentation will be done by others beside myself.

Work Items
----------

1. Identify impacts to existing services (if any) and submit appropriate
requests (via Launchpad)

2. Create documentation changes


Future lifecycle
================

I expect there will be pressure to expand the specification in future release
cycles.


Dependencies
============

None


Testing
=======

There is some implication for testing, since we want to check for existence of
the fields defined in the specification.


Documentation Impact
====================

Documentation for this change is the core deliverable. While it won't require a
lot of documentation, the readability and descriptiveness of the documentation
is critical. We expect to leverage much of the content at
https://wiki.openstack.org/wiki/Ceilometer/blueprints/StandardPaaSEventFormat
as well as add updated event payload examples.


References
==========
The full blueprint for this change is available at:
https://wiki.openstack.org/wiki/Ceilometer/blueprints/StandardPaaSEventFormat

The related blueprint outlining changes to the PaaS event collection mechanism
(that benefits from on this standard being in place)
is available at:
https://wiki.openstack.org/wiki/Ceilometer/blueprints/PaaSEventUsageCollection
