..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Multi meter arithmetic transformer
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/arithmetic-transformer

Adding the ability to take more than one meter into account when performing
a transformation. An example of this is the memory utilization, where::

    memory_util = 100 * memory.usage / memory

Problem description
===================

Pipeline transformers are currently performing calculations only on one or
more values of the same meter. Some calculations need to take into account
multiple meters, like in the case of memory utilization, where::

    memory_util = 100 * memory.usage / memory

Proposed change
===============

We suggest an implementation of a flexible transformer capable of performing
arithmetic operations on multiple meters and/or their metadata. To prevent
irregular cadence of the produced meter, calculation is limited to meters
with the same interval.

The configuration of the new transformer would be::

    - name: "arithmetic"
      parameters:
        target:
          name: "memory_util"
          unit: "%"
          type: "gauge"
          expr: "100 * $(memory.usage) / $(memory)"

To demonstrate the use of metadata, here is the implementation of
a silly metric that shows average CPU time per core::

    - name: "arithmetic"
      parameters:
        target:
          name: "avg_cpu_per_core"
          unit: "ns"
          type: "cumulative"
          expr: "$(cpu) / ($(cpu).resource_metadata.cpu_number or 1)"

Expression evaluation would gracefully handle NaNs and exceptions. In such
a case it would not create a new sample but would only log a warning.

**Parsing the meter names**

Since meter names are not necessarily valid Python identifiers, we must
escape them. The expression string is only parsed at the time the pipeline
is initialized, so it does not cause any performance overhead.

The escaping is done as follows::

    $(cpu) -> cpu
    $(cpu_util) -> cpu_util
    $(cpu.util) -> _cpu_util_ESCAPED (in order to avoid clashes with legal
      meter names)
    $(class) -> _class_ESCAPED (avoid clash with Python keywords)

Also, if the meter name isn't followed by a dot ('.'), we default to the
volume parameter.

So, the expression::

    $(memory.usage) / $(memory)

will be translated to::

    _memory_usage_ESCAPED.volume / memory.volume

and eval'd with the following Namespace_ object

.. _Namespace: https://github.com/openstack/ceilometer/blob/master/ceilometer/transformer/conversions.py#L30)

::

    Namespace({
        "_memory_usage_ESCAPED": { ... memory.usage sample dict ... }
        "memory": { ... memory sample dict ... }
    })

**Gathering the values**

Not all the needed samples will arrive at the same time. Multiple approaches
are possible here.

1. *[CHOSEN APPROACH]* We can cache samples of the necessary meters (keyed by
resource ID) as they arrive. When the pipeline is flushed, we can check if we
have the necessary samples and do the transformation.

* *pro*: no need for specifying TTL for cached samples
* *con*: arithmetic operation limited only to samples that originate
    from the same polling task (=> same interval)

2. We cache the samples same as above, but do the transformation
as soon as we have the necessary samples (samples that are used in the
calculation expression). After the transformation
we clear the cache. The user provides the maximum age (TTL) of a sample for it
to be considered for calculation. E.g. we get a sample of 'memory.usage'
and we already have a sample of 'memory' in cache, which is less than
TTL=10s old. We can perform the calculation.

* *pro*: meters used not limited to the same polling task
* *pro*: largest flexibility of all options
* *con*: need to specify maximum age for samples to be used in calculation
* *con*: might produce samples with irregular cadence, as in the case of
    the calculation involving meter A with 60s cadence and meter B with 45s
    cadence

3. Similar as #2, but limit only to meters from sources with the same interval.

* *pro*: produced samples can't have irregular cadence
* *pro*: since all meters involved have the same interval, we can have a
    static TTL for samples, no need for user configuration. If we know that
    all the samples arrive at approximately the same time (because they are
    on the same interval and therefore in the same polling task), we only have
    a short period of time in which the samples are valid (say, 5 seconds).
* *con*: The next question is how to set the TTL? With a 600s cadence,
    5 seconds is OK, but with 10s cadence it probably isn't.
* *con*: how to technically enforce this restriction?

4. Same as #2, but provide a configuration option to not clear the cache
after calculation.

* *pro*: enables use cases like "we get 'memory.usage' every 10s and 'memory'
    every '600s'. If we don't clear the cache, we can calculate 'memory_util'
    every 10s using the cached value of 'memory'.
* *con*: extra configuration option might be confusing for the user. The
    benefit might outweigh this.

Alternatives
------------

* require user to assign the meters to "variables", e.g.::

    - name: "arithmetic"
      parameters:
        target:
          name: "memory_util"
          unit: "%"
          type: "gauge"
          a:    "memory.usage"
          b:    "memory"
          expr: "100 * a / b"

  Unnecessary since the parsing is only done at pipeline initialization and so
  does not cause any overhead in processing.

* somehow change the pipeline to only send the samples to transformer when
  it has all the required samples

  Our approach is simpler, since other transformers already use caching so it
  wouldn't make sense to push this functionality to the pipeline.

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

It impacts the format of the pipeline.yaml as described in section
Implementation.

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

None. Expression parsing is performed only at pipeline initialization.
In regard to caching, other transformers already cache values between
intervals. The only difference is that we would store values for more than
one meter.


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
  nejc-saje

Ongoing maintainer:
  nejc-saje

Work Items
----------

* create the new transformer
* write unit tests for the new transformer


Future lifecycle
================

None


Dependencies
============

None


Testing
=======

With the current capabilities of Tempest testing I think unit tests
should suffice.

Documentation Impact
====================

Documentation should be added for the new type of transformer.

Documentation must make it clear that the meters included in the expression
must have the same cadence. In case that the expression contains meters
with different cadences or nonexisting meters, the user should expect to have
a warning logged for each unsuccessful calculation.


References
==========

None

