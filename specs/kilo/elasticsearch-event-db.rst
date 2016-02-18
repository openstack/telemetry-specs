..
   This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================================
ElasticSearch backend for Events
================================

https://blueprints.launchpad.net/ceilometer/+spec/elasticsearch-driver

As we split our backends to store segregated data (alarm, metering, events), we
have to ability to choose backends tailored to store said data.

Problem description
===================

While SQL is able to store unstructured data, it's true use case is to
effectively store data in a defined schema and it's relationships so that
it's easily queryable.

Events in OpenStack are for the most part schema-free; each notification
message may contain any combination of attributes based on when and where the
message originated from.

Proposed change
===============

ElasticSearch is designed primarily to store unstructured real time data flows,
similar to the events in OpenStack and has worked with relative success in
other projects[1].

ElasticSearch will be added in as an alternative backend to the current
offerings of HBase, MongoDB, and SQL. The current implementation of filtering
attributes using a definition file will continue to be used. A Logstash
implementation of capturing and processing events is not in the scope of this
blueprint[2]

Additionally, the api will continue to match the events api currently offered
rather than implementing logstash.

Samples backend is not in the scope of this patch as TSDBs offer arguably
better support for capturing measurements. That said, alarms database could be
feasible and may be included in subsequent patches.

Alternatives
------------

As mentioned, existing solutions using MongoDB, HBase, and SQL exists. They
will continue to be viable options.

Using Kibana to retrieve/analyze data will not be the default solution to query
data. That said, deployers should be able to bypass Ceilometer's api and use
Kibana if desired.

It should be noted, ElasticSearch currently isn't recommended as a primary
storage engine [3][4]. There is debate around how consistent it is.

There is another solution to use Mongodb as a primary storage and pipe data
to ElasticSearch for better querying.[5]

ElasticSearch parallels MongoDB as it is also based on JSON. The reason for
having ElasticSearch as an alternative is because it allows us to create
indices based on time so we can effectivley shard data as well as expire data.
Also, in addition to INFO notifications, services also emit ERROR notifications
which can be richer in textual information, and ElasticSearch is designed
specifically for such cases.

Data model impact
-----------------

No data model changes are required. Events will continue to have the following
attributes::

   * message_id
   * event_type
   * generated
   * list of traits

REST API impact
---------------

None

Security impact
---------------

Nothing new

Pipeline impact
---------------

None

Other end user impact
---------------------

None, unless ElasticSearch is chosen as the backend. Then ElasticSearch will
need to be configured accordingly.

Performance/Scalability Impacts
-------------------------------

None, as it's an optional backend. ElasticSearch has the ability to cluster to
provide HA and it is built to scale horizontally[6]

Other deployer impact
---------------------

The ElasticSearch storage driver is to have feature parity with the rest of
the currently available event driver backends. It will add an elasticsearch-py
client dependency should the driver be selected.

Developer impact
----------------

Devs will need to use ElasticSearch client when modifying ElasticSearch driver

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  chungg

Ongoing maintainer:
  chungg

Work Items
----------

* write equivalent event driver functions and test.

Future lifecycle
================

Depending on evolution of events in Ceilometer, ElasticSearch driver will need
to be updated to support new features (if feasible/logical). ElasticSearch may
be extended to cover alarms (and samples) but is not currently in scope because
time series databases are probably a better solution for measurements.

Dependencies
============

* elasticsearch-py client library

Testing
=======

testing will need to be done against a mocked db or a real ElasticSearch database

Documentation Impact
====================

Update driver docs

References
==========

[1] http://www.elasticsearch.org/case-studies/
[2] http://logstash.net/docs/1.4.2/
[3] http://www.bigdatamontreal.org/?p=305 - elasticsearch employee
[4] http://aphyr.com/posts/317-call-me-maybe-elasticsearch
[5] http://stackoverflow.com/questions/20080189/index-mongodb-with-elasticsearch/20120927#20120927
[6] http://www.elasticsearch.org/overview/elasticsearch
