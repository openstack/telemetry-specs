..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Highly Distributed Coordinated Notifications
============================================

https://blueprints.launchpad.net/ceilometer/+spec/disributed-coordinated-notifications

In Kilo, support for coordinated notifications agents was added. This enabled
users to deploy multiple notification agents and ensured related messages
within a pipeline were funnelled into the same agent to allow for proper
aggregation calculations.

Problem description
===================

In the initial implementation, all the data relating to a pipeline was
funneled into a single queue per pipeline. While it ensured all corresponding
data is sent to the same place, it removed the ability to scale horizontally
as the pipeline queues can only have a single consumer listening to it.
This means that while multiple agents/handlers could be used to pull data off
main OpenStack queue, once the data reached pipeline processing, it was
relegated to a single worker.

Data can be handled in parallel even at the pipeline level. For example, when
there are no transformers, datapoints do not need to be handled sequentially.
Additionally, when transformers are present, datapoints of different resources
have no relevance to each other and can be handled in parallel.

Proposed change
===============

To parallelise and scale out processing, we will create multiple copies of
each pipeline. When a datapoint arrives, we will bucketise each datapoint by
a hashed grouping key. Each transfromer will have a grouping key assigned to it
to note any dependency requirements (ie. transformers that work on resource
ids). When setting up pipeline, all the keys of transformers in the pipeline
will be combined to ensure that related datapoints will be consistently sent
to the same pipeline for processing.

The basic workflow is as follows::

  * on notification agent startup, create a listener for main queue
  * for each pipeline definition, we create x queues and x listeners where x
    corresponds to the number of notification agents registered to group
  * when a datapoint is received, agent builds sample.
  * after sample is built, we hash fields defined by transformer requirements
    and mod by number of agents.
  * using mod value we push datapoint to corresponding pipeline queue.
  * same processing steps here on out (listener grabs data -> pipeline -> pub)

This solution CANNOT handle multiple grouping_keys in pipeline. To properly
handle multiple grouping_keys in a pipeline we need to requeue after each
transform. The logic would become: main queue -> build sample ->
pipe1.transform1 queue -> pipe1.transform2 queue -> etc -> publish.

Studying the existing transformers we have::

  * Accumulator - this does not really have any grouping requirements, it just
                  batches samples.
  * Arithmetic - this is grouped by resource_id
  * RateOfChange - this is grouped by name+resource_id. but it can more
                   be more generally grouped by just resource_id
  * Aggregator - this is grouped by name+resource_id+<custom> but can also be
                 more generally grouped by just resource_id

Based on the above, it seems like resource_id is always a valid general
grouping key and the transformers may do more granular groupings themselves.
Because of this, it seems safe to assume we don't need to support multiple
grouping keys (for now).


Alternatives
------------

1. Dumb easy fix is to detect whether a pipeline has transformers. If not, it
   can be consumed by any number of consumers and thus we can assign multiple
   listeners to those queues. This doesn't help distribute load of pipelines
   with transformers but allows for transformers spanning resources.

2. We implement a shared memory/storage mechanism so any worker can discover
   the historical context. This is hard. I feel like i would end up recreating
   Storm/Spark

3. Magic

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

Nothing from user point of view. Internally, we will have more pipeline
queues.

Other end user impact
---------------------

None. Unless we decide to add option to define number of copies of pipeline
queues.

Performance/Scalability Impacts
-------------------------------

Positive. It will distribute pipeline processing when running in coordinated
mode. Message queues are consistently used with thousands of queues so
creating copies of pipeline queues (a relatively small set) should not be an
issue.

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
  chungg

Ongoing maintainer:
  chungg

Work Items
----------

* add grouping key to each transformer to define grouping of datapoints
  * will be just resource_id
* add support to build pipeline hashing from above grouping keys
* add functionality to create pipelines queues per agent and distribution
  logic.


Future lifecycle
================

Support different grouping keys in a pipeline.

Dependencies
============

None

Testing
=======

Test already exists. Just need to validate that we create appropriate amount
of copies.

Documentation Impact
====================

None. Maybe dev docs.

References
==========

https://docs.google.com/presentation/d/1QgjDOLRnKDboqP8P1LvV0kR5aQEv_VJsDtMlh6u7tIY/edit?usp=sharing
