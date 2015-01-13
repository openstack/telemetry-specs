..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============
Event Pipelines
===============

https://blueprints.launchpad.net/ceilometer/+spec/notification-pipeline

Problem description
===================

Currently, when notifications are picked up by the notification agent, they
are all funnelled into a single endpoint, filtered using event_definition and
pushed to the dispatchers, which is generally a database. There is a lack of
flexibility to transform, extend, trigger on these events and to publish
to this data to multiple consumers.

Proposed change
===============

Like samples, we have a pipeline for events. Like samples, we define sources
and sinks. Unlike samples, we don't have an interval defined in sources.

This will also allow ability to publish different data (ie. audit) to different
storages and also allow ability to ignore notifications completely (currently,
we capture a shell event for all notifications)

The proposed schema is::

  ---
  sources:
      - name: eventA_source # any unique name for source
        events: # list of event_types, same wildcard technique in samples
            - "*"
        sinks:
            - sink1
            - sink2
  sinks:
      - name: sink1 # any unique name for sink
        transformers:
            # one or many transformers that work on an Event.
        triggers:
            # potential for triggering (short term inline actions). i'd
            # envision a trigger that built performance samples. ie time
            # between corresponding start/end events and republish as a sample
            # alternatively, that same work could send an alarm if it didn't
            # meet a certain threshold
        publishers: # we will not support rpc it isn't great for performance
            - notifier://

Publishers here can be a little different. we currently publish events straight
to the database avoiding the collector. In this spec, we will continue this
offering in addition to the notifier, udp, file options. The collector offers
no discernible benefit apart from moving any synchronous task off the
notification agent to the collector. As we already requeue items for each
pipeline, this synchronous bottleneck exists only on the pipeline which has the
synchronous task and will not slow down the other pipelines.

Alternatives
------------

- publish to the collector instead and let it persist data.
- put this in same pipeline.yaml file as samples.

Data model impact
-----------------

None.

REST API impact
---------------

None currently but it'll affect pipeline in database work.

Security impact
---------------

Same security concerns as current publishers.

Pipeline impact
---------------

This is a completely new pipeline so yes there is impact: a new pipeline.yaml

Other end user impact
---------------------

You need another pipeline.yaml; You need to configure it appropriately.

Performance/Scalability Impacts
-------------------------------

This will by default cause no difference. There's potential for less data
because of ability to filter out events. There's potential for more data
because of transformers and multiple publishers. We have notification agent
coordination which will split pipeline and data across agents to handle
scaling.

Other deployer impact
---------------------

Adding a configuration option for new pipeline.yaml

Developer impact
----------------

Understand how pipelines work if they don't understand already.

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

- create event_pipeline.yaml
- redirect current event endpoint to use this pipeline
- add publisher support for events

Future lifecycle
================

Build transformers. Build triggers. Build publishers.

Dependencies
============

None.

Testing
=======

- extend pipeline testing to include event_pipeline

Documentation Impact
====================

- rewrite pipeline notes to include new event_pipeline and it's differences

References
==========

https://blueprints.launchpad.net/ceilometer/+spec/notification-pipelines
