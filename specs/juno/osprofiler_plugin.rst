..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================
OSprofiler notification plugin
==============================

https://blueprints.launchpad.net/ceilometer/+spec/osprofiler-plugin


This spec is about integration between osprofiler and ceilometer.
Actually osprofiler requires some kind on notification API and collector.
Which is actually Ceilometer.

More about OSprofiler:
https://github.com/stackforge/osprofiler


Problem description
===================

In OpenStack we have bunch of services inside every project. It's really hard
to understand where and why your request works slow. To resolve this issue we
introduce tiny library, that can be easily integrated in all existing service
and will allow us to get 1 trace (tree of elements) per API request.
It's simpler to see, then to explain sample:
http://pavlovic.me/rally/profiler/

To resolve this task, we have to send from all services to some common
collector special notifications. As there is already Ceilometer, that is
integrated with everything and as well notification API that perfectly works
for us we decide to use it (Ceilometer).

Proposed change
===============

Add new Ceilometer notification plugin, that will collect notifications
related to profiler.


Alternatives
------------

Make another collector of notifications (Ceilometer) and integrate it in
all projects ;)

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

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

It depends on how heavy you are using osprofiler. By default even if everything
is turned on, osprofiler won't sent any notifications.
profiler will send notifications only in case if person that knows hmac key
sends special trace info in special header.

Amount of data that will be send depends on API request. One of worst API call
is boot VM. If we trace all rest/rpc/db calls it will be about 150kb of
samples. Note with disabled tracing of DB we can reduce multiple times amount
of profiling data.

Warnings:

* Do not trace every request, especially such like "nova boot VM".

* Keep hmac in safe, if somebody knows it, he can create trace for every
  request or in another words DDOS Ceilometer.


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
  Boris Pavlovic <boris@pavlovic.me>


Work Items
----------

* Add new notification plugin (already done):
  https://review.openstack.org/#/c/100239/

* Turn it on by default in devstack


Future lifecycle
================

This plugin is essential for OSprofiler, and OSprofiler is essential for
OpenStack. So this code will be fully supported.


Dependencies
============

* There is no dependencies.


Testing
=======

For the begging we are going to have only unit tests.
Actually code is quite simple, so it doesn't require functional testing, as
well this code won't be changed a lot (at all).

The second reason why unit test are enough is that this plugin will be heavy
used in rally gates. So if something went wrong we will catch it quite fast.


Documentation Impact
====================

Add warning that profiler creates huge load on Ceilometer backend. It means
that it shouldn't be turned on for every request. As well we should say that
load can be drastically reduced in case of turned off tracing of DB request.

References
==========

* OSprofiler lib: https://github.com/stackforge/osprofiler

* oslo.messaging notification api for osprofiler: http://goo.gl/nCK21D

* Cinder integration: http://goo.gl/yY42jd

* Result: http://pavlovic.me/rally/profiler/
