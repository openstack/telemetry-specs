..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=====================
Event Alarm Evaluator
=====================

https://blueprints.launchpad.net/ceilometer/+spec/event-alarm-evaluator

This blueprint proposes to add a new alarm evaluator for handling alarms on
events passed from other OpenStack services, that provides event-driven alarm
evaluation which makes new sequence in Ceilometer for handling alarms on events
separated from other types of alarms handled in the existing polling-based
Alarm Evaluator, and realizes immediate alarm notification to end users.

Problem description
===================

As an end user, I need to receive alarm notification immediately once
Ceilometer captured an event which would make alarm fired, so that I can
perform recovery actions promptly to shorten downtime of my service.
The typical use case is that an end user set alarm on "compute.instance.update"
in order to trigger recovery actions once the instance status has changed to
'shutdown' or 'error'. It should be possible for an end user to receive a
notification within 1 second of a fault being observed, as with other health-
check mechanisms can do in some cases.

The existing Alarm Evaluator is periodically querying/polling the databases
in order to check all alarms independently from other processes. This is good
approach for evaluating an alarm on samples stored in a certain period.
However, this is not efficient to evaluate an alarm on events which are emitted
by other OpenStack servers once in a while.

The periodical evaluation leads delay on sending alarm notification to users.
The default period of evaluation cycle is 60 seconds. It is recommended that
an operator set longer interval than configured pipeline interval for
underlying metrics, and also longer enough to evaluate all defined alarms
in certain period while taking into account the number of resources, users and
alarms.

Proposed change
===============

The proposal is to add a new event-driven alarm evaluator which receives
messages from Notification Agent and finds related Alarms, then evaluates each
alarms;

* New alarm evaluator receives event notification from Notification Agent
  by which adding a dedicated notifier to publish event in a new topic.
  The topic name is 'alarm.all'.

* When new alarm evaluator received event notification, it queries alarm
  database by Project ID written in the event notification.
  To reduce heavy load of Ceilometer API, this alarm evaluator would cache
  alarm definitions which queried by Project ID in a certain period.

* Found alarms are evaluated by referring event notification.

* Depending on the result of evaluation, those alarms would be fired through
  Alarm Notifier as the same as existing Alarm Evaluator does.

This proposal also adds new alarm type "event" and "event_rule".
This enables users to create alarms on events. The separation from other alarm
types (such as "threshold" type) is intended to show different timing of
evaluation and different format of condition, since the new evaluator will
check each event notification once it received whereas "threshold" alarm can
evaluate average of values in certain period calculated from multiple samples.

The new alarm evaluator handles "event" type alarms, so we have to change
existing alarm evaluator to exclude "event" type alarms from evaluation
targets.

Project ID of events would be retrieved from traits named 'project_id' and
'tenant_id' by this alarm evaluator, since there is no common field to hold it.
Not all events have project ID; all events of virtual resource like VM instance
should have project ID which belongs to, but events of physical resources like
host don't have project ID. Since those raw infra events have to be hidden from
end users, those will be treated as labeled with PROJECT_NONE ('' in API) which
can be set in an alarm definition only by admin.

Alternatives
------------

There was similar blueprint proposal "Alarm type based on notification", but
the approach is different. The old proposal was to adding new step (alarm
evaluations) in Notification Agent every time it received event from other
OpenStack services, whereas this proposal intends to execute alarm evaluation
in another component which can minimize impact to existing pipeline processing.

Another approach is enhancement of existing alarm evaluator by adding
notification listener. However, there are two issues; 1) this approach could
cause stall of periodical evaluations when it receives bulk of notifications,
and 2) this could break the alarm partitioning i.e. when alarm evaluator
received notification, it might have to evaluate some alarms which are not
assign to it.

Caching all alarm definitions can reduce the number of query to API/DB in each
period and reduce time of evaluation in other projects, but it may lead longer
time in first query and larger footprint of cache since there can be large
number of projects whereas alarms in a project could be limited by quota.

Event ID can be used instead of Project ID in alarm query, but it cannot ensure
the number of alarms retrieved from DB and difficult to get all alarms when we
allow users to use wildcard in "event_type" of an alarm definition, which is
proposed in this spec.

Resource ID could be added to Alarm data model as an optional attribute.
This would help the new alarm evaluator to filter out non-related alarms
while querying alarms, otherwise it have to evaluate all alarms in the project.
To focus on creating basic framework of event alarms, we decided not to handle
Resource ID in this blueprint.

Data model impact
-----------------

None

REST API impact
---------------

Alarm API will be extended as follows;

* Add "event" type into alarm type list

* Add "event_rule" to "alarm"

  * "event_rule" has "event_type", which can include "*", to indicate which
    type(s) of event to be evaluated on the alarm

  * "event_rule" has "query" which is a list of conditions combined by AND as
    the same as AlarmThresholdRule

Sample data of Notification-type alarm::

  {
      "alarm_actions": [
          "http://site:8000/alarm"
      ],
      "alarm_id": null,
      "description": "An event alarm",
      "enabled": true,
      "insufficient_data_actions": [
          "http://site:8000/nodata"
      ],
      "name": "InstanceStatusAlarm",
      "event_rule": {
          "event_type": "compute.instance.update",
          "query" : [
              {
                  "field" : "traits.instance_id",
                  "type" : "string",
                  "value" : "153462d0-a9b8-4b5b-8175-9e4b05e9b856",
                  "op" : "eq",
              },
              {
                  "field" : "traits.state",
                  "type" : "string",
                  "value" : "error",
                  "op" : "eq",
              },
          ]
      },
      "ok_actions": [],
      "project_id": "c96c887c216949acbdfbd8b494863567",
      "repeat_actions": false,
      "severity": "moderate",
      "state": "ok",
      "state_timestamp": "2015-04-03T17:49:38.406845",
      "timestamp": "2015-04-03T17:49:38.406839",
      "type": "event",
      "user_id": "c96c887c216949acbdfbd8b494863567"
  }

Security impact
---------------

Since default event notification may include raw infra information,
operator/administrator should configure event definitions carefully when end
users is allowed to operate this event alarm API and can receive event alarm
notification.

Pipeline impact
---------------

This change needs to add new notifier into event pipeline in order to pass
event to this new alarm evaluator.

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

When Ceilometer received a number of events from other OpenStack services in
short period, this alarm evaluator can keep working since events are queued in
a messaging queue system, but it can cause delay of alarm notification to users
and increase the number of read and write access to alarm database.

All event alarms defined in the project will be evaluated every time this
evaluator received event. The number of alarms to be evaluated can be reduced
by adding new parameter (e.g. Resource ID) on Alarm data model and setting
filter while alarm querying.

Other deployer impact
---------------------

Notification Agent have to be configured to publish event to a new topic
'alarm.all' which is dedicated to this event alarm evaluator and different from
current messaging to store events (i.e. add 'notifier://?topic=alarm.all' in
event_pipeline.yaml). Note this configuration should be done when the new event
alarm evaluator runs in the deployment, otherwise it may fill up the queue.

New service process (this alarm evaluator) have to be run.

A deployer can run multiple evaluators in order to scale out event alarm
evaluation process. All event will be dispatched to all evaluators listening
the topic in a round robin fashion.

Developer impact
----------------

Developers should be aware that events could be notified to end users and avoid
passing raw infra information to end users, while defining events and traits.

All events related to virtual resources should have project ID and user ID
properly.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  r-mibu

Other contributors:
  lianhao-lu
  edwin-zhai

Ongoing maintainer:
  None

Work Items
----------

* Add new alarm type "event" as well as AlarmEventRule

* Modify existing alarm evaluator to filter out "event" alarms

* New event-driven alarm evaluator

* Make the new evaluator cache alarm definitions

Future lifecycle
================

This proposal is key feature to provide information of cloud resources to end
users in real-time that enables efficient integration with user-side manager
or Orchestrator, whereas currently those information are considered to be
consumed by admin side tool or service.
Based on this change, we will seek orchestrating scenarios including fault
recovery and add useful event definition as well as additional traits.

This feature will or can be enhanced by the followings in the future;

* Enabling this evaluator to get delta of alarm definitions in the last period,
  in order to reduce traffic on updating cache. This requires alarm storage to
  hold deleted alarms and alarm API amendment.

* Adopting similar coordination process as the notification agent has, for
  efficiency of this evaluator e.g. reducing cache of alarm definitions.
  Key for partitioning might be 'project_id', 'event_type' or 'resource_id'.
  For this coordination, all topic name will have the same prefix 'alarm.',
  so that evaluator can use topic='alarm.*' to listen all messages for event
  alarm evaluation. This is the reason why we use 'alarm.all' in this spec.

* Mechanism to update cache of alarm definition promptly; poisoning cache on
  evaluator in which assigned alarm definition has updated, etc.

* Filtering out uninterested events in notification agent by leveraging the
  mechanism of graceful pipeline update and reflecting alarm definitions
  configured by end users.

We can refactor function to retrieve project ID from event object after
creating common field in event object that requires DB and API changes.

Dependencies
============

None

Testing
=======

New unit/scenario tests are required for this change.

Documentation Impact
====================

* Administrator Guide and Installation Guide in OpenStack Manuals should be
  updated to describe new alarm type and rule as well as all deployer impacts.

* Proposed evaluator will be described in the developer document.


References
==========

* OPNFV Doctor project: https://wiki.opnfv.org/doctor

* Blueprint "Alarm type based on notification":
  https://blueprints.launchpad.net/ceilometer/+spec/alarm-on-notification

* Liberty Summit Note: https://etherpad.openstack.org/p/event_alarm
