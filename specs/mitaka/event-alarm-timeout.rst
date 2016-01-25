..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================
Event Alarm Timeout
===================

https://blueprints.launchpad.net/aodh/+spec/event-alarm-timeout

This BP adds timeout mechanism for event-alarm. End users can specify a
timeout, 0 (no timeout) by default, for each event-alarm. The alarm status
becomes 'TIMEOUT' after timeout reached without receiving desired event.


Problem description
===================

After event-alarm were introduced in Liberty, end users or operators could set
alarm for desired event and get alarmed when it receive them. But in some
circumstances, operator want to know otherwise: when desired event is not
received.

For example, "compute.instance.create.end" is the final event sent to message
bus to indicate success of instance creation. Not receiving it after a long
time is a signal of creation failure, in which operator should be notified.
Unfortunately, current event-alarm doesn't support it.


Proposed change
===============

When creating event-alarm, a new parameter 'timeout' is proposed to define a
expiry time length, so that alarm gets fired when desired event is not received
in expected time. Otherwise, alarm status becomes 'TIMEOUT'.

Currently, 3 states are supported in alarm: 'UNKNOWN', 'ALARM' and 'OK', so a
new state 'TIMEOUT' will be added to reflect timeout situation.

For timeout implementation, an 'alarm.timeout.start' notification is sent out
by the AODH api process after the event-alarm is created. After receiving it,
evaluator asks its timeout thread/process to handle timeout request. In this
way, avoid new process in AODH api and make all alarm handling jobs inside
evaluator.

Synchronization handling is critical in evaluator, as both evaluators original
process and timeout process can change status for same alarm. To avoid
complicated lock, timeout process just send a 'alarm.timeout.end' event with
related alarm/project id to 'alarm.all' topic, where evaluator original process
handle it along with desired event.

Each 'alarm.timeout.*' event should carry enough info in payload, including
alarm_id, project_id, timeout, and desired event. These info could be used for
evaluation and future partitioning infrastructure.

Each evaluator has only one timeout thread, which gets timeout requests from
evaluator into a queue. Small footprint of timeout process is guaranteed, as
it always sleeps unless one timeout happens. After wake up, it only does 2
things:

* sends out 'alarm.timeout.end' event
* pick up nearest timeout request and start sleeping for it

The final alarm status depends on the order of events. If 'timeout.end' event
comes first, alarm status becomes 'TIMEOUT' and following desired event is
ignored.  Otherwise, alarm status becomes 'ALARM' and following 'timeout.end'
event is ignored.  In this way, synchronization is well handled with new
'timeout.end' event and simple logic in evaluator.

After adding timeout, event-alarm state transition changes as following:

* EVENT - desired event arrive
* TIMEOUT - timeout happen

Registered action is only triggered when:

* transition between different states, like UNKNOWN => ALARM
* receiving desired event again at ALARM state if repeat_actions is true

::


 +-----------+            +---------+  EVENT
 |           |            |         +---------+
 |  UNKNOWN  +------------> TIMEOUT |         |
 |           |  TIMEOUT   |         <---------+
 +-----+-----+            +---------+
       |
       |
       |                  +---------+
       |                  |         +---------+
       +------------------>  ALARM  |         |
               EVENT      |         <---------+
                          +-+-----^-+  TIMEOUT
                            |     |
                            +-----+
                             EVENT




Also need changes in DB layer to get all the event-alarm with timeout. In
following cases, we need restart timeout thread if the alarm has timeout, and
previous timeout thread already exit:

* reset the alarm state to 'UNKNOWN'

* enable the alarm via the 'enabled' attribute


Alternatives
------------

There is another BP from Igor to illustrate a different high level design and
usage model. Pls. check
https://blueprints.launchpad.net/ceilometer/+spec/timeout-event-alarm-evaluator

It adds a new alarm type 'event_timeout' to track sequence of
events, like: "compute.instance.create.start", "compute.instance.create.end".
So this new alarm includes 2 events: start and end -- only after receiving
"start" event, timeout is created for "end" event. It is not as simple and
flexible as this BP, which just adds timeout.

To handle the synchronization, an alternative is to lock critical section
between evaluator timeout and original thread, so each of them need handle
alarm data structure. But it's tricky and bug prone.


Data model impact
-----------------

None

REST API impact
---------------

API needs minor changes to add 'timeout' to 'event_rule'. One simple
event-alarm definition is as following::


  {"alarm_actions": ["log://"],
   "ok_actions": ["log://"],
   "alarm_id": null,
   "enabled": true,
   "name": "alarm01",
   "repeat_actions": false,
   "state": "insufficient data",
   "event_rule": {"query": [{"field": "traits.name",
                             "type": "string",
                             "value": "cirros-0.3.4-x86_64-uec-ramdisk",
                             "op": "eq"}],
                  "event_type": "image.update",
                  "timeout": 10},
   "type": "event"}

Security impact
---------------

None

Pipeline impact
---------------

None

Other end user impact
---------------------

End user need to know a new 'timeout' parameter when create event alarm.


Performance/Scalability Impacts
-------------------------------

No obvious performance issue, because of small footprint of timeout thread. No
obvious scalability issue, as timeout handling is done in evaluator who will
support good partition.

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
  edwin-zhai

Work Items
----------

* Add new parameter 'timeout' for event-alarm creation in aodh-client

* Add new alarm state 'TIMEOUT' for timeout expired alarms

* Add new interface in aodh-client and DB layer to get all event-alarm with
  timeout

* Modify AODH api's event-alarm creating code to send out 'alarm.timeout.start'
  notification

* Modify AODH event-alarm evaluator so that:

  * spawn a new timeout thread to handle all timeout requests
  * timeout thread works in a loop of sleeping for timeout seconds then sending
    out 'alarm.timeout.end' event
  * set related alarm status as 'TIMEOUT' when receive 'alarm.timeout.end'
    event.

* Add extra action to restart timeout thread when reset alarm state to
  'UNKNOWN' or enable the alarm if previous timeout thread already exit


Future lifecycle
================

To be maintained by edwin-zhai for bug fixing and enhancement.

In future, we need timeout thread disaster-recovery capability, that is, no loss
of timeout info when evaluator crash. Need store pending timeout requests in
DB, and feed evaluator when restarting.


Dependencies
============

None


Testing
=======

Add new test case besides current event-alarm test to cover timeout


Documentation Impact
====================

Administrator Guide and Installation Guide in OpenStack Manuals should be
updated to describe usage of 'timeout' parameter.


References
==========

Blueprint Timeout mechanism for Event Alarm Evaluator
https://blueprints.launchpad.net/ceilometer/+spec/timeout-event-alarm-evaluator
