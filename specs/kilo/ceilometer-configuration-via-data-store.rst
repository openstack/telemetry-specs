..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Configuration via Data Store
============================

https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-configuration-via-data-store

By updating Ceilometer to monitor values recorded in a persistent data store
such as MySQL, the application could be made to dynamically activate/deactivate
collection targets or similar functions, have distinctly different
configurations for multiple nodes in different environments (i.e. dev/test/prod
or HA scenarios) and could ultimately be updated with new collection targets
“on-the-fly”.




Problem description
===================

Currently, Ceilometer relies on multiple configuration files for determining
run-time parameters used in polling and notification handling. The current
configuration file approach limits the extent to which we can customize
Ceilometer functionality (e.g. polling targets, intervals, publishing) and
becomes a potential issue in a large-scale deployment. Migrating the
configuration options to a data store gives flexibility and lays the
foundation for on-the-fly updates to Ceilometer.


Proposed change
===============

The transition of configuration parameters to a data store can be chunked into
three efforts:

1. Migration of pipeline configuration
2. Migration of event traits configuration
3. Migration of general (Oslo) configuration

Because there's a significant Oslo project investment in managing the general
configuration, we won't attempt to transition that in the Kilo timeframe. The
migration of the pipeline configuration and event traits, being limited to Kilo
and not having any specific implementation (beyond Pyaml), seem feasible to
address.


Alternatives
------------

There's no arguing that deployment tools such as Puppet, Chef, etc. support the
population of configuration files on a reasonable scale. For small deployments
it might be preferable to maintain a file-based configuration. To this end we
can support a configuration parameter enables a file-based configuration
(likely to be the default for the sake of backward compatibility).


Data model impact
-----------------

This will require the addition of multiple table schemas to the Ceilometer data
model to handle this "metadata". At minimum, there will be one table per
configuration type, likely more due to normalization.

Alternately, a separate configuration database could be instantiated, with the
benefit of providing better segregation of access and support for a multi-node
deployment.

REST API impact
---------------

This configuration data store would require a dedicated set of api requests for
both the configuration (ceilometer.conf) and pipeline (currently pipeline.yaml).
Generally modeled on the Alarm api, at minimum the pipeline configuration api
would consist of:

GET /v2/meta/pipelines
    Return all pipeline configuration items
    Parameters: q - Filter rules for items to be returned
    Return type: list(configuration)

PUT /v2/meta/pipelines
    Create a new pipeline configuration item with a user-defined id
    Parameters: data(configuration) - a configuration within the
    request body
    Return type: configuration

GET /v2/meta/pipelines/(pipeline_id)
    Return this pipeline configuration item
    Return type: configuration

PATCH /v2/meta/pipelines/(pipeline_id)
    Modify this pipeline configuration item
    Parameters: data(pipeline) - one or more attributes of the
    configuration item
    Return type: configuration

DELETE /v2/meta/pipelines/(pipeline_id)
    Delete this pipeline configuration item

Where "pipeline configuration" is the set of attributes currently defined in
the pipeline.yaml. Generally: name, interval, associated transformer(s),
associated publisher(s).

The configuration (ceilometer.conf) api will follow similar format using
/v2/meta/configurations/ as the endpoint.


Security impact
---------------

Data segregation and access control is critical to this change, as this
effectively moves an administrative data set into the database and provides
api access to it.


Pipeline impact
---------------

The significant impact to the pipeline will be the (optional) sourcing of
configuration data from the database. We'll retain backward compatibility of
file-based configuration: the pipeline configuration file (currently
pipeline.yaml) will be deployed and will include the minimum-required
configuration by default, as will the initial database deployment. The
deployer will have the option of activating db-based configuration and
using the API for configuration changes.

A second impact to the pipeline will be the need to reconstruct the polling
tasks when a change to the configuration is made. The current long-running
tasks will need to be stopped and re-started, while accounting for in-process
tasks.


Other end user impact
---------------------

Configuration via the Horizon metering dashboard could be considered for a
future iteration. Out of scope for initial implementation.

Performance/Scalability Impacts
-------------------------------

The absolute number of records in this database should be relatively small,
however the number of API requests may be significant.

Other deployer impact
---------------------

Backwards compatibility with existing configuration files will be required.
This will be an explicitly enabled option, driven by ceilometer.conf
parameter(s).

Developer impact
----------------

New configuration items would need to be represented in the database, rather
than the configuration files.

Implementation
==============

Assignee(s)
-----------


Primary assignee:
  nealph

Other contributors:
  TBD

Ongoing maintainer:
  nealph

Work Items
----------

1. Database schema design
2. Database pre-population, deployment, migration
3. Updates to pipeline to read config via api or natively
4. API design
5. Tempest tests
6. Doc updates

Future lifecycle
================

We'll want to continue to iterate on this for future cycles to gain additional
functionality:
Run-time monitoring of config changes
Configuration updates from the Horizon dashboard
Oslo config migration

Dependencies
============

Relates to:
https://blueprints.launchpad.net/ceilometer/+spec/dedicated-event-db
https://review.openstack.org/#/c/119077/ (central and compute agents merge)

Testing
=======

Extensions to Database and API tests should be sufficient for this change.

Documentation Impact
====================

Documentation will be required/updated for:
Configuration api
Installation guide

References
==========

https://etherpad.openstack.org/p/configuration_via_data_store (includes
database schema)
https://wiki.openstack.org/wiki/Ceilometer/blueprints/Configuration-via-data-store


