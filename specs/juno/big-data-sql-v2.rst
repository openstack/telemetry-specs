..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
'Big Data' SQL part two
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/bigger-data-sql

In step 1 of sql refactoring, we denormalised the data to capture and store
data as close to its raw form as possible. This removed all the overhead
caused by deprecated data design requirements. This blueprint further
refactors the data model to organise data closer to api model.


Problem description
===================

Currently, the Metering data model is completely denormalised. We store
Samples, which are the raw data points, and Meters, which are the definition
of said data points. This optimisation allows for good write performance but
due to size of Sample table, can cause issues with read performance
particularly with get_meter and get_resources. Specifically, joins can cause
performance issues.

The current schema is as follows::

    * meter - meter definition
        * id: meter id
        * name: meter name
        * type: meter type
        * unit: meter unit
    * sample - the raw incoming data
        * id: sample id
        * meter_id: meter id (->meter.id)
        * user_id: user uuid
        * project_id: project uuid
        * resource_id: resource uuid
        * source_id: source id
        * resource_metadata: metadata dictionaries
        * volume: sample volume
        * timestamp: datetime
        * message_signature: message signature
        * message_id: message uuid


Proposed change
===============

The proposed change is to re-implement a normalised model that is tailored to
current API requirements. This means grouping Resource specific data in
Resource table and Meter specific data in Meter Table.

the proposed schema is as follows::

    * resource - resource data
        * internal_id: resource id (to handle assumption resources may not be
                       unique ie. diff user/project/source/meta per resource)
        * resource_id: resource uuid
        * user_id: user uuid
        * project_id: project uuid
        * source_id: source id
        * resource_metadata: metadata dictionary
        * metadata_hash: hash of metadata to allow comparison
    * meter - meter definition
        * id: meter id
        * name: meter name
        * type: meter type
        * unit: meter unit
    * sample - the raw incoming data
        * id: sample id
        * meter_id: meter id (->meter.id)
        * resource_id: resource id(->resource.internal_id)
        * volume: sample volume
        * timestamp: datetime
        * message_signature: message signature
        * message_id: message uuid

Alternatives
------------

As a schema can be defined in quite a few ways, there are many alternatives in
that sense.

Data model impact
-----------------

Api model will remain unchanged. the sql backend model will change to proposal
described above:

* creating a new Resource table
* moving appropriate values from sample table to resource table

These changes will require database migration.

REST API impact
---------------

We will need to adapt existing api model to interface with new backend schema
but from user POV, there will be no change.

Security impact
---------------

None

Pipeline impact
---------------

None

Other end user impact
---------------------

None, we will continue to store a new resource per sample

Performance/Scalability Impacts
-------------------------------

The read performance should improve as we will not have a giant Sample
table anymore but smaller, tailored Resource, Meter, and Sample tables.
The write performance is not expected to degrade noticeably.

It is expected any degradation in write performance would be caught by
existing tempest tests.

Use of Ilya's performance tool will be used to verify there is improved read
performance and negligible write performance degradation.[1]

[1] https://github.com/ityaptin/ceilometer/blob/master/tools/sample-generator.py


Other deployer impact
---------------------

None

Developer impact
----------------

None, just a new schema to learn about


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  chungg

Other contributors:
  None

Ongoing maintainer:
  chungg

Work Items
----------

* Migration script to add new attributes to Meter table and new Resource table
* Modify impl_sqlalchemy get_meters, get_resource, record_metering_data,
  expirer and any other affected methods to use new schema


Future lifecycle
================

Most contributors know the sql backend to some degree. The community will
maintain until v3 backend is phased in.


Dependencies
============

None


Testing
=======

* Existing test cases should cover change
* Tempest test cases should cover performance degradation
* Need to add test to handle data expiration


Documentation Impact
====================

None


References
==========

Discussion with Mike Bayer:
https://etherpad.openstack.org/p/ceilometer-sqlalchemy-mike-bayer

