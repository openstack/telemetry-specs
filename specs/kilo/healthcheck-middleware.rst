..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Healthcheck Middleware Metering
===============================

https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-healthcheck-middleware

Oslo.middleware is going to provide a healthcheck middleware in order to monitor
health of HTTP services. We want to monitor this middleware response code and
time in Ceilometer in order to have analytics available.

Problem description
===================

The healthcheck functionnality provided by oslo.middleware will provide
information about the health of a particular API service. It'll be useful to
retrieve periodically the state of the service in Ceilometer to have the ability
to analyse these data.

Proposed change
===============

Let's write a pollster that polls the healthcheck middleware to generate samples
measuring the response time of the middleware, and incorporating the status code
as metadata.

In order to find which endpoints to poll, the pollster will rely on the
EndpointDiscovery discover to find them.

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

None

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------


Primary assignee:
  jdanjou

Other contributors:
  sileht

Ongoing maintainer:
  jdanjou
  sileht

Work Items
----------

* Write a pollster

Future lifecycle
================

As all pollsters.

Dependencies
============

None

Testing
=======

We should be able to test using unit testing and testing inside Tempest.
Devstack should have support of this healthcheck middlware by default.

Documentation Impact
====================

The new pollster should be documented the same way we do for others.

References
==========

* oslo.middleware healthcheck blueprint
  <https://blueprints.launchpad.net/oslo.middleware/+spec/oslo-middleware-healthcheck>_
