..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Dispatcher for Gnocchi Integration
==========================================

Ceilometer is planned to be re-based on a TimeSeries as a Service concept,
namely Gnocchi. We've reached the state, when these two projects can be
loosely integrated. This blueprint is about to describe the options for
implementing a dispatcher for realizing the data flow between the monitored
infrastructure and the database.

There are several ways for integrating these two projects. This blueprint
represents a first stage, proof of concept solution for testing how the new
design is performing. It could/should be refactored later to reach a better
performance and closer integration.

    https://blueprints.launchpad.net/ceilometer/+spec/dispatcher-for-gnocchi-integration


Problem description
===================

Gnocchi has several new terms comparing to Ceilometer, like entity and
archive/archiving policy. Ceilometer does not create/define these items now,
therefore it is the key point of the integration process, how to synchronize
the mechanisms that are needed for storing the samples and the corresponding
metadata in the new structure.


Proposed change
===============

This proposal describes the solution, which is about to implement a new
dispatcher that can be used in the collector to send data to Gnocchi, instead
of one of the currently supported database backends.

For storing time series data, we need to do several preparation steps in the
polling/notification handler workflow. The processed notifications and also
the polled data have metadata information attached to it. We need to define
resources that will hold this additional information.

The resource creation has to happen before storing any datapoints. To make
this mechanism effective and easy to implement and extend later the plan is
to create stevedore extensions for the different resource types, which
provides a pluggable solution for the future.

These resource creation extensions will each handle the creation of a
strongly-typed resource of a single type, and be aware of the
resource-specific attributes required in the Gnocchi representation. The
loaded resource creation extensions will be managed by the dispatcher as a
map keyed on meter-name (acting as a proxy for resource type). Resources of
an unrecognized type will be modelled in Gnocchi as generic resources.

The resource creation would happen using PUT request, if the given resource
does not exist.

The next step after having a resource is to create entity for the given
measurement, that is connected to the previously created resource. The entity
creation should also happen automatically, without any human interaction.
Meter name will be used to name each entity. As a first step, the entity
creation will happen in two steps. First to retrieve the resource and check
whether the particular entity exists or not and in the latter case create
the entity with a default archiving policy.

Later on as a next step, to avoid additional overhead of checking whether the
entity exists or not, we try to POST new measurements with using the
following request::

 POST /v1/resource/UUID/entity/{entity.name}/measures ...

If the request fails with missing entity error, then we create the entity
with using a default archiving policy, as it is described below associate
the entity with the resource, and then, and post the measurements again.
In the next polling cycle or when the next notification is received for the
given entity, the POST request will be successful. This option requires
a new operation to be supported on the Gnocchi API, that is why it will be a
future improvement.

There is also a need for at least one archiving policy to be existed before
storing any time series data. We can reuse the currently existing
configuration parameters that are used for the pipelines or the samples
lifecycle to define archives.

As these archives can be defined based on the lifespan and granularity values
the ttl value that specifies how long the samples should be kept in the
database can be reused to define the former parameter. The granularity
will be configured as one second by default in the first step.

Later on it will be also supported to change these values or adding new
archives via the Ceilometer's REST API, but currently we provide the mechanism
the can be built in the current data collection and storage mechanisms.

The data will be stored by calling the REST API of Gnocchi from the collector.


Alternatives
------------

The alternate solution is to remove the collector from the chain and store the
collected data directly from the pollsters. This would increase the
performance, as we do not add additional steps before storing the data in the
database.

This solution needs more effort, therefore it is a proposed way of future
improvement of the integration.


Data model impact
-----------------

New resource types were introduced by Gnocchi, which are handled inside that
module, Ceilometer is aware of the available types, for creating new
instances, when it's necessary.


REST API impact
---------------

The data retrieval will change after fully integrating Gnocchi into
Ceilometer. This will be introduced by the new v3 API, described in a separate
blueprint.


Security impact
---------------

None


Pipeline impact
---------------

None


Other end user impact
---------------------

The deployment of Gnocchi should be done before using this new solution.


Performance/Scalability Impacts
-------------------------------

One of the goals of this blueprint is to make it possible how this new
approach behaves comparing with the old one.


Other deployer impact
---------------------

None


Developer impact
----------------

The new way of storing data will not replace the currently existing solution.
If someone wants to develop some feature based on the new approach, he/she has
to be aware of how Gnocchi works.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  ildikov

Other contributors:
  jdanjou

Ongoing maintainer:
  ildikov, jdanjou, eglynn

Work Items
----------

1. create extensions and the additional steps for resource creation
2. finish the implementation of the dispatcher


Future lifecycle
================

The mid-term plans are to integrate Gnocchi closely into Ceilometer, if the
performance and functionality are both fulfills the requirements. This
means that this first step may be changed in some points and the final
solution will then be maintained by the Ceilometer core team and further
contributors.


Dependencies
============

The current state of Gnocchi supports this integration.


Testing
=======

Unit tests should be provided for the new implementation.


Documentation Impact
====================

The documentation should be updated with the configuration needs for deploying
Ceilometer with Gnocchi and also with the information about how to reach the
data via the Gnocchi API, until Ceilometer v3 API is not available.


References
==========

* `Initial dispatcher implementation`_

.. _Initial dispatcher implementation: https://review.openstack.org/98798