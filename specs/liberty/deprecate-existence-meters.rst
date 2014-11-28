..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Existence Meters Deprecation
============================

https://blueprints.launchpad.net/ceilometer/+spec/deprecate-existence-meters

Problem description
===================

As Ceilometer has expanded to capture more notifications from the OpenStack
message ecosystem, the number of samples it generates grows even faster as
many samples are derived from a single notification or polling request.

The "Existence of xyz" meters we store in our samples database
represents a significant portion of the data we store. These meters however
offer no useful measurement value and it's true value is to capture the
state of the resource at a given time -- a value that is also available in the
meters generated alongside the existence meters. Additionally, the volume=1
value is very confusing to consumers of data as for a while, Horizon used that
value as the total number of values and often users would wonder why there was
always 1 record of an instance, network, port, etc...

As we move to a more time-series focused storage for samples, the "volume=1"
meters we collect has not just an impact on storage size, but also the overhead
of rolling up and computing statistics on something as trivial and meaningless
as the constant 1. Additionally, the rollup of samples will diminish the
value of said meters as valid auditable datapoints.

At a high-level, Samples are the children of Events. Samples are a derived
subset of an Event. Because of that, the Samples we create should capture an
explicit datapoint of interest from an Event and not just be a shadow of an
Event.

Proposed change
===============

The "Existence of xyz" and "volume=1" meters can be better represented as what
they really are: Events. The core event functionality was implemented in
previous cycles and should be expanded to better cover these non-measurement
meters.

This will offer better querying of data in Events, less storage of data in
general, and less confusion as to what the volume attribute in a sample
represents.

The proposed solution is to::

  1. Mark 'volume=1' meters as available as Events in the documentation and
     add them to the event_definitions file. The vast majority of them are
     created from notifications so no work needs to be done aside from adding
     them to event_defintion. **COMPLETED IN KILO**
  2. Change default pipeline to not enable these meters. NOTE: this does not
     affect existing solutions nor will it break anything. An option was added
     in Kilo to enable/disable meters. **COMPLETED IN KILO**
  3. Some meters such as network related meters (LBaaS, VPNaaS, SDN,
     etc...) come from pollsters. We will need to convert these pollsters to
     republish information as events.  An example would be health check polls
     made for network meters[5]. Instead of creating a sample, it would create
     an event.
  4. Henceforth, Samples will be measurements which can be aggregated and
     rolled up in meaningful ways (within gnocchi). Events will be the raw
     base that Samples come from and they should be stored accordingly. Both
     are time series driven data.

A rough list of meters to be dropped can be found at the correspond bug[1].


Alternatives
------------

None. We could leave step 3 as optional as they are essentially healthcheck or
existence events. We can also support a tool to migrate 'non-metric' meters to
events but this is arguably not important.

Data model impact
-----------------

None. Events already exist. This may have a side dependency on adding alarming
on Events to maintain feature parity.

REST API impact
---------------

None. we may want to expand Events but nothing is directly impacted here.

Security impact
---------------

Audit data. This is an issue that already exists and will just be migrated from
Samples to Events. It should be possible to add a tag to the event_definition
file to mark data as audit. It is technically possible to redirect audit data to
a separate db using event pipeline[4]

Pipeline impact
---------------

None.

Other end user impact
---------------------

Users STILL have the option to keep these meters but we should advocate they
use Events instead. As of Liberty, these meters will be disabled by default
and remove completely as of M*. This has been documented in docs already.

Performance/Scalability Impacts
-------------------------------

Less sample data. Less equivalent data in Events (because of trait filtering).
Events are a bit more scalable in it's design (maybe not the SQL backend).

Other deployer impact
---------------------

We should turn on store_events option by default. This will probably require
deployers to use event_connection option as it probably isn't advisable to
store sample and event data together but they could.

Developer impact
----------------

Learn events. Make events better.


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

- add pollster to Events functionality
- turn on store_events by default

Future lifecycle
================

- add alarming to Events.[3]


Dependencies
============

None.


Testing
=======

None.


Documentation Impact
====================

- mark meters in measurement as available as Events and add a link to events.


References
==========

[1] https://bugs.launchpad.net/ceilometer/+bug/1384874
[3] https://blueprints.launchpad.net/ceilometer/+spec/notifications-triggers
[4] https://bugs.launchpad.net/ceilometer/+bug/1376915
[5] https://github.com/openstack/ceilometer/blob/master/ceilometer/network/services/fwaas.py
