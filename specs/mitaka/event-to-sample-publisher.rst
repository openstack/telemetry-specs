..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.
 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Event To Sample Publisher
===============================
https://blueprints.launchpad.net/ceilometer/+spec/event-to-sample-publisher

Problem description
===================
Bracket events like compute.instance.create.latency created from BP:
Transformers for events pipeline
https://blueprints.launchpad.net/ceilometer/+spec/events-pipeline-transformers
need to be published and stored as sample.

The bracket events created from transformers in events pipeline are new
events/metrics with any kind of latency time spans like time.instance.creation
or time.instance.life that is a delta time between
compute.instance.create.start/end, compute.instance.create.end/delete.end, etc.

Proposed change
===============
Add a new notifier publisher and use this publisher in event transformer,
when there is a new transformed event from events pipeline
transformers, convert it to notification, send it back to ceilometer notificaton listener,
the notification payload is generated from event traits,
then the sample pipeline will convert and publish it into sample.

For example, for compute.instance.create.latency event,
Firstly, add this publisher(for example: tosamplenotifier://)
to sink in event transformer for this event ::

    - name: instance_create_bracketer_sink
      transformers:
          - name: "bracket"
            parameters:
              ........
      publishers:
          - tosamplenotifier://

then, add meter schema in meters.yaml ::

  - name: 'compute.instance.create.latency'
    event_type:
      - 'compute.instance.create.latency'
    type: 'gauge'
    unit: 's'
    volume: $.payload.latency
    user_id: $.payload.user_id
    project_id: $.payload.tenant_id
    resource_id: $.payload.instance_id

The notification payload send back to message bus is generated from event traits, will be like this ::
 {'latency': 0.355098, 'user_id': u'0f1b1e94ec2045af9f49f9b7e1d6b409', 'service': u'network.yuntong-ThinkStation',
 'resource_id': u'67d1c4b3-84aa-42d1-a857-2d1481fe21dd', 'tenant_id': u'c8ce7938e38b4612a8b3daab441b804c',
 'request_id': u'req-597a85ec-ebab-492f-8288-b6b72fc476b5', 'project_id': u'c8ce7938e38b4612a8b3daab441b804c',
 'publisher_id': 'compute.yuntong-ThinkStation'}

and finally, the compute.instance.create.latency sample will be like this ::
 +--------------------------------------+---------------------------------+-------+----------+------+----------------------------+
 | Resource ID                          | Name                            | Type  | Volume   | Unit | Timestamp                  |
 +--------------------------------------+---------------------------------+-------+----------+------+----------------------------+
 | e8e8adf5-8ba1-4247-b1af-1e8c928563e7 | compute.instance.create.latency | gauge | 7.133763 | s    | 2015-09-16T07:04:58.540929 |
 +--------------------------------------+---------------------------------+-------+----------+------+----------------------------+


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

This spec proposes additional publisher for event transformers
in event_pipeline.yaml

Other end user impact
---------------------

None.

Performance/Scalability Impacts
-------------------------------

None

Other deployer impact
---------------------

Configuration options in event_pipeline.yaml.
Configuration options in meter.yaml.

Developer impact
----------------

None.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  yuntongjin

Ongoing maintainer:
  yuntongjin


Work Items
----------

- Add a new publisher that will send the convert notification to sample listener.


Future lifecycle
================

None

Dependencies
============

Transformers for events pipeline https://review.openstack.org/#/c/162167/

Testing
=======

- Extend messaging publisher testing.

Documentation Impact
====================

- Capture new meters which from event brackelet transformers.

References
==========

https://blueprints.launchpad.net/ceilometer/+spec/events-pipeline-transformers
