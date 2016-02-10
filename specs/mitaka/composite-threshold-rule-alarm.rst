..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================================
Support composite threshold rule alarm
======================================

https://blueprints.launchpad.net/ceilometer/+spec/composite-threshold-rule-alarm

This proposal want to support creating alarm that can specify composite
threshold rule and try to replace combination alarm.

Problem description
===================

The combination alarm was designed to handle alarm triggered against multiple
conditions. A combination alarm need to depend on other alarms, the dependent
alarms can be threshold alarms or combination alarms. This will form a
dependency chain if the situation is complex. There are some potential problems
of combination alarm evaluation, which have been described by ZhiQiang Fan in
blueprint[1]:

- dependency chain is too long, which causes the state of combination alarm
  will not be evaluated timely, in the worst case, it will cause the
  combination alarms never be updated.

- dead loop will be introduced when update alarm, which causes the state of
  combination alarm will not be set properly.

- dependency chain will be broken when delete dependent alarm.

For more information, please see: blueprint[1] and spec[2]

ZhiQiang Fan have made some efforts for these problems by
resolve-alarm-dependency-chain proposal[2]. That proposal tried to check and
resolve the dependency chain in both API and alarm-evaluator layers. To address
the above problems, the implementation of that proposal will be complex, and,
consider the alarm-evaluator working with partition coordinator, the situation
is worse, and hard to be addressed by that proposal.


Proposed change
===============

This proposal allow specifying multiple threshold rules when creating a
threshold alarm, and the threshold rules combined with logical operators: and,
or. following is a sample for the new alarm body::

    {
        ......
        "threshold_rules": {
            "or": [{
                "type": "threshold" (or "gnocchi_threshold")
                "meter_name": "cpu_util",
                "evaluation_periods": 1,
                "period": 60,
                "statistic": "avg",
                "threshold": 0.8,
                "query": [{
                    "field": "metadata.metering.stack_id",
                    "value": "36b20eb3-d749-4964-a7d2-a71147cd8147",
                    "op": "ge"
                }],
                "comparison_operator": "ge",
                "exclude_outliers": false
            },
            {
                "and": [{
                    "type": "threshold" (or "gnocchi_threshold")
                    "meter_name": "mem_util",
                    "evaluation_periods": 1,
                    "period": 60,
                    "statistic": "avg",
                    "threshold": 0.8,
                    "query": [{
                        "field": "metadata.metering.stack_id",
                        "value": "36b20eb3-d749-4964-a7d2-a71147cd8147",
                        "op": "ge"
                    }],
                    "comparison_operator": "ge",
                    "exclude_outliers": false
                },
                {
                    "type": "threshold" (or "gnocchi_threshold")
                    "meter_name": "disk.usage",
                    "evaluation_periods": 1,
                    "period": 60,
                    "statistic": "avg",
                    "threshold": 0.8,
                    "query": [{
                        "field": "metadata.metering.stack_id",
                        "value": "36b20eb3-d749-4964-a7d2-a71147cd8147",
                        "op": "ge"
                    }],
                    "comparison_operator": "ge",
                    "exclude_outliers": false
                }]
            }]
        }
    }

The change items of this proposal:

* Allow specifying a list of threshold rules and the threshold rules combined
  with two logical operators: "and", "or", see the above example. The
  combination of threshold rules can be multi-hierarchy to support complex
  scenario.

* Alarm evaluator will resolve the alarm threshold rules combined with
  operators, and evaluate the threshold condition with statistics list queried
  from Ceilometer in proper order. In this process, the short-circuit logic
  should be considered to avoid unnecessary statistics query and comparison.

* This proposal cannot be applied to EventAlarm, it mainly because event alarm
  evaluation is not periodical, the event alarm evaluator listening message bus
  and the evaluation action is passively triggered by external notifications.
  This is disadvantage if there is alarm usage that need to combine event
  alarm and threshold alarm, but I'd rather this situation should be handled in
  users end. That means if users want alarm fired when threshold reached and
  specify event occurred meanwhile, they can combine threshold alarm and event
  alarm in upstream application.

* Also considered above disadvantage, the old combination alarm functionality
  will also be kept as supplementary for composite rules alarm, but need to
  add documentation to describe the potential problem.

These change will solve the above issues easily and hope to replace combination
alarms. And also, this change will reduce the amount of alarms definition in
our cloud, because currently, we may need to create the dependent alarms
firstly if we want combination alarms. For end users, it is also convenient to
show the alarm triggering multiple conditions than combination alarms (which
need to query dependent alarms successively).

Alternatives
------------

Adopt the proposal[2] and struggle to make it works with partition coordinator.

Data model impact
-----------------

* The "threshold_rules" will be a dict that can be include multiple threshold
  rules and the key of the dict will be "and" or "or". But this has no
  effect on data model, because the rules is stored as json in db.

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
  liusheng

Other contributors:
  you

Ongoing maintainer:
  liusheng

Work Items
----------

* Support Aodh API to allow creating alarms with this new threshold rules
  definition, and hopefully, the API can keep compatibility with former usage.

* Implement the related changes in storage layer

* Change the threshold evaluator to support multiple threshold rules and
  logical operators attached with an alarm.


Future lifecycle
================

None

Dependencies
============

None

Testing
=======

* The exist declarative tests and new tests with gabbi should be changed/added
  to test the alarm CRUD actions.

* Unit tests should be added for new threshold evaluator process.

Documentation Impact
====================

* API documentation should be updated for adding description about this
  feature.

* Users guide should be updated for add the this usage description.


References
==========

[1] https://blueprints.launchpad.net/ceilometer/+spec/resolve-combination-alarm-dependency-chain

[2] https://review.openstack.org/#/c/98047/
