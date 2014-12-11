..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================================
Integrating Kafka Publisher into Ceilometer Publisher
======================================================

https://blueprints.launchpad.net/ceilometer/+spec/kafka-publisher

Apache kafka is messaging broker which enables the ability to publish real-time
metering data to applications outside of OpenStack Projects with low-latency
and high-throughput.

Problem description
===================

Ceilometer monitors and collects data from OpenStack via notifications sent
from existing services or by polling the infrastructure. These real-time
collected data have great potential to be used for various kind of analysis.
There are many external projects which analyze, serialize, and visualize these
metering data, for example, Storm, Elastic Search, Mahout, Jubatus etc. In
OpenStack, oslo.messaging library provides nice communication paths, however
these communication paths are not so general especially in external to
OpenStack projects. Thus, this blueprint is trying to publish ceilometer
metering data to such external projects using Kafka, distributed messaging
system, and achieve the full potential of ceilometer metering data.

Proposed change
===============

To reduce the cost of implementing a Kafka publisher independently,
Ceilometer has to provide the kafka publisher as a publisher plugin.
This automates publishing metering data, and developers only have to
configure the kafka publisher plugin in pipeline.yaml file like::

    publisher:
        - kafka://<broker_ip>?topic=<topic_name>

This way any application that is trying to consume streaming ceilometer
metrics via Kafka, can directly consume the ceilometer samples. For example
projects like monasca - https://github.com/stackforge/monasca-thresh can consume
the ceilometer metrics that are published by the Ceilometer Kafka publisher.

Alternatives
------------

Inside of the OpenStack, to communicate with each modules is achieved by
oslo.messaging, however defacto usage of oslo.messaging is for OpenStack projects
and Kafka is supporting for more general usages.

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

Current security support of Kafka is none. Authentication of clients and ecrypting
connections are under implementation. More information can be found in the references.

Pipeline impact
---------------

Provide new options to specify kafka publiser as a ceilometer publisher

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

The paper related to Kafka says that producers can publish about 50,000
messages/sec on the condition that message size is 200 byte and messages are
sent one by one. However, this number is affected by parameters such as message
size, the number of replication, batch size (kafka can send multiple messages
together), and environments. Roughly, RabbitMQ can publish about 10,000
messages/sec, it means this kafka publihser might not be the bottleneck of
performance.

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
  <kshimamu>

Other contributors:
  <yudupi>

Ongoing maintainer:
  <kshimamu>

Work Items
----------

* adding a kafka publisher as a ceilometer publisher

Future lifecycle
================

None

Dependencies
============

Kafka python package

* https://pypi.python.org/pypi/kafka-python

Testing
=======

None

Documentation Impact
====================

None

References
==========

Apache Kafka Project

* http://kafka.apache.org/

Kafka: a Distributed Messaging System for Log Processing

* http://research.microsoft.com/en-us/um/people/srikanth/netdb11/netdb11papers/netdb11-final12.pdf

Kafka Security

* https://cwiki.apache.org/confluence/display/KAFKA/Security

KAFKA 0.8 PRODUCER PERFORMANCE

* http://blog.liveramp.com/2013/04/08/kafka-0-8-producer-performance-2/

Performance of Kafka

* http://kafka.apache.org/07/performance.html

RabbitMQ Performance Benchmarks

* http://blogs.vmware.com/vfabric/2013/04/how-fast-is-a-rabbit-basic-rabbitmq-performance-benchmarks.html
