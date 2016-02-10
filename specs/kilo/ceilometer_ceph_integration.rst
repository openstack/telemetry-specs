..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Ceph object storage meters
==========================

https://blueprints.launchpad.net/ceilometer/+spec/ceph-ceilometer-integration

This spec proposes to add ceph object storage metering support to Ceilometer
by polling mechanism, using the ceph radosgw (rados gateway) APIs, when the
ceph is used as object storage, instead of swift.


Problem description
===================


Currently, ceilometer doesn't has the capability to get the meters from ceph
object storage, when the ceph is used as an object storage, instead of swift
object storage).


Proposed change
===============

Ceph object storage (i.e ragosgw) provide a few REST APIs to get the meters
like object/container list, object/container size, etc. Ceilometer can use
these REST APIs to get the needful metering information.

In Ceilometer - polling mechanism is required, to get the meter details from
the ceph object storage. This polling method could be similar to swift polling
mechanism.


Add new pollster classes for ceph object storage to get object storage meter
information into sample-list, using existing `storage.objects` implementation
as a model used with swift object storage.

The ceph object storage poller names have the following pattern:

* ceph.storage.objects
* ceph.storage.objects.size
* ceph.storage.objects.containers
* ceph.storage.containers.objects
* ceph.storage.containers.objects.size
* ceph.storage.api.request

To implement the above - we will call the ceph object storage (i.e radosgw)
REST APIs.


Alternatives
------------

Ceph object storage can push the notifications for the meters to ceilometer,
but currently ceph doesn't support the push notifications support, so its a
complex alternative to use ATM.


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

None.


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
  swamireddy

Ongoing maintainer:
  swamireddy

Work Items
----------

* Configure the setup (i.e setup.cfg) file based on the object storage used
  to get the metering information. If swift used, use swift collectors to get
  the metering information. If ceph used, use ceph collectors to get the
  metering information.

* Implement the ceph object storage (radosgw) APIs calls for appropriately
  for each metering item. Here, the python swfitclient will be used to
  interact with ceph rados gateway.

* Currently, ceph support the keystone authentication. To check if it works
  with ceilometer also.

* Update/modify the ceph radosgw REST APIs appropriately, based on ceilometer
  requirements. But at present, planned to use the currently supported APIs
  from ceph radosgw, without major changes.

* Add test coverage for the above pollster classes and meter samples validation.


Future lifecycle
================

In this spec, we have dealt with meters, which can be supported using the
polling mechanism. But a few meters like object input/output bytes size
need the push notifications support from ceph. If the push notifications
supports in ceph, we add these meters also in ceilometer in future.

In the future new types of metering items are expected from the ceph object
storage to account for the statistics data. These will need to be handled
separately.


Dependencies
============

Ceph - radosgw REST APIs support required to do this. If ceph API is not
supported, we may need to implement it in ceph - radosgw.


Testing
=======

Unit tests will be added to cover the necessary pollster classes and validate
them with appropriate meter samples generated.

The integration tests will be performed with help of 3rd party CI setup.


Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/developer/ceilometer/measurements.html

References
==========

None
