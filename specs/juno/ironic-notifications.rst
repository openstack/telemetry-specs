..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================
Ironic Notifications
====================

https://blueprints.launchpad.net/ceilometer/+spec/ironic-notifications

Work is `in progress`_ to get Ironic to emit notifications from data provided by
IPMI sensors (such as cpu temp and voltage). Ceilometer needs to be updated to
consume, transform and record these notifications.

.. _in progress: https://blueprints.launchpad.net/ironic/+spec/send-data-to-ceilometer

Problem description
===================

The Ironic project is doing work to emit notifications on the message bus
containing IPMI sensor data. If Ceilometer processes these notifications other
services will be able use queries and alarms to monitor and scale as required.

It is not possible to simply dump a message on the bus and for Ceilometer to
handle it. Instead a notification plugin is required, in Ceilometer, to
hear notifications at an exchange and topic and transform them into samples.

Proposed change
===============

Add a new notification plugin for Ironic to transform sensor data into samples,
using existing ``NotficationBase`` implementations as a model. To do this
effectively a complete sample of the expected sensor data is required to
determine the relevant types of samples.

Alternatives
------------

Handling notifications is the standard and preferred method for gathering data
into Ceilometer. The other option is polling which has known scalability
issues, including over-frequent `polling of IPMI`_. If Ironic is sending
notifications it can continue to own the credentials for the IPMI access and
also control the cadence with which sensors are polled. Using notifications also
allows other services to consume the sensor data.

.. _polling of IPMI: http://lists.openstack.org/pipermail/openstack-dev/2014-March/031101.html

Data model impact
-----------------

None.

REST API impact
---------------

There will be additional valid values in query parameters but no changes to API
endpoints.

Security impact
---------------

None.

Pipeline impact
---------------

Unknown. The data being provided by the Ironic notifier is still being decided.
Only when it is available will it be possible to determine what, if any,
transformations may be usefully done in the pipeline.

Other end user impact
---------------------

None.

Performance/Scalability Impacts
-------------------------------

No new impacts. Pre-existing concerns with capacity at the notification and
storage handling layers remain.

Other deployer impact
---------------------

None.

Developer impact
----------------

None.

Implementation
==============

Assignee(s)
-----------

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  chdent

Other contributors:
  whaom

Ongoing maintainer:
  chdent

Work Items
----------

* Establish expected data.

* Create tests of transformation of ``sensordata`` to samples.

* Create notification plugin to consume ``sensordata``.

* Create tests of notifications across fake bus.

* Create sample query tests.

Future lifecycle
================

In the future new types of notifications are expected from the Ironic
controller. These will need to be handled either by additional notification
plugins or (hopefully) generic notification handling. The Ceilometer team will
be responsible for collaborating with the Ironic team to ensure these are
handled smoothly.

Dependencies
============

The primary dependency is work described in an `Ironic spec`_. Once
that is implemented, notifications will be present on the bus.

Testing
=======

In addition to unit tests, Tempest tests which confirm that Ceilometer consumes
notifications produced by Ironic would be useful. Such tests depend on there
being a tool to provide sensor data in the testing environment. Such a tool
would be similar to `bm_poseur`_ which fakes baremetal instances in devstack.
In the event that such a tool is not available as long as the sample data used
in unit tests is sufficiently representative then the tests themselves ought
to be as well.

.. _bm_poseur: https://github.com/tripleo/bm_poseur

Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/developer/ceilometer/measurements.html

References
==========

* `Ironic spec`_
* `Review in progress`_ for sending notifcation from Ironic.
* `Sample data`_

.. _Ironic spec: https://blueprints.launchpad.net/ironic/+spec/send-data-to-ceilometer
.. _Review in progress: https://review.openstack.org/#/c/72538/
.. _Sample data: http://paste.openstack.org/show/85053/
