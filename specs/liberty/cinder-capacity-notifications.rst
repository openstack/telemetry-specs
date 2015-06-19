..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Track cinder capacity notifications
====================================

https://blueprints.launchpad.net/ceilometer/+spec/cinder-capacity-notifications

The goal of this blueprint is to capture the capacity notifications emitted
by cinder service when it notifies storage capacity information to ceilometer.

Problem description
===================

Cinder service collects storage capacity information about each pool/backend.
These includes total/free/allocated/provisioned/virtual_free capacity info.
Cinder service emits the formatted information as notifications periodically.

It's better for ceilometer service to add a notification plugin to listen to
the topic. If the information payload can be consumed and transformed into
samples, it will be helpful for admin users to have an estimation of the future
capacity planning.

Proposed change
===============

Add new metrics for different capacity information carried by capacity payload
from notifications emitted by Cinder. They will include:

* CapacityTotalSize
  It is the total physical capacity of a pool/backend.

* CapacityFreeSize
  It is the real physical available capacity of a pool/backend.

* CapacityAllocatedSize
  It is the physical capacity allocated directly thru Cinder.
  Note: The capacity which is not allocated thru Cinder is not included in.

* CapacityProvisionedSize
  It is the capacity which has been provisoned in a pool/backend.

  Note: It includes the capacity both allocated directly thru Cinder and not.
  The provisioned capacity size is equal or greater than the allocated size.

* CapacityVirtualfreeSize
  It is the apparent availabe virtual capacity means how much capacity can
  still provision besides the capacity which has been provisioned already
  in the pool/backend.

  Note: It is different from free capacity, free capacity is related to real
  available physical capacity. For thin provisioning support, due to the
  max_over_subscription_ratio, the provisioned capacity can be much larger
  than the real physical capacity.

  Please reference https://review.openstack.org/#/c/129342/ to get the detail
  explanation of above terminology.

Each metric extracts the capacity information which it is interested in from the
notification payload. The admin users can utilize ceilometer statistics API to
get the sum of each kind of capacity. Also they can utilize the horizon resource
usage page to get the chart of the trend of each kind of capacity information.

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

None.

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

Primary assignee:
  XinXiaohui

Ongoing maintainer:
  XinXiaohui

Work Items
----------

* Add capacity.total.size metric

* Add capacity.free.size metric

* Add capacity.allocated.size metric

* Add capacity.provisioned.size metric

* Add capacity.virtual_free.size metric

* Test coverage for the above metrics and samples validation.

Future lifecycle
================

In the future new types of capacity notifications maybe expected from the
Cinder service to account for the statistics data. These will need to be
handled later.

Dependencies
============

None.

Testing
=======

Unit and integration Tests will be added to cover the necessary metrics
and validate samples generated.

Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/admin-guide-cloud/content/section_telemetry-measurements.html

References
==========

https://etherpad.openstack.org/p/kilo-cinder-capacity-headroom

https://review.openstack.org/#/c/170380
