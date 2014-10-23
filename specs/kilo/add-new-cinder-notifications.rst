..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================
Add new notifications types for volumes/snapshots
=================================================

https://blueprints.launchpad.net/ceilometer/+spec/add-new-notifications-types-for-volumes-and-snapshots

Now we have only two types of notifications about volumes/snapshots:
volume/snapshot existence and their size.
This change allows to collect and view notifications of different types.
We can get information about which events have occurred: volume/snapshot has
been created or deleted or updated(renamed or modified description), volume
has been resized or attached/detached. This will allow to process additional
events and will improve the overall Ceilometer functionality.

Problem description
===================

Now we have only two types of notifications about volumes/snapshots:
volume/snapshot existence and their size. But there is no information about
events like volume/snapshot was created or deleted or updated or volume was
resized or attached/detached.

Proposed change
===============

This change allows to collect and view notifications of different types -
volume/snapshot.create.start, volume/snapshot.create.end,
volume/snapshot.delete.start, volume/snapshot.delete.end,
volume/snapshot.update.start, volume/snapshot.update.end,
volume.resize.start, volume.resize.end,
volume.attach.start, volume.attach.end,
volume.detach.start, volume.detach.end.

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
    enovokshonova <enovokshonova@mirantis.com>

Work Items
----------

Implement appropriate handler classes.

Future lifecycle
================

None.

Dependencies
============

None.

Testing
=======

This change needs to be tested by unit tests.

Documentation Impact
====================

We need to add new meter types to
http://docs.openstack.org/developer/ceilometer/measurements.html.

References
==========

https://github.com/openstack/cinder/blob/master/bin/cinder-volume-usage-audit
https://github.com/openstack/cinder/blob/master/cinder/volume/manager.py
https://github.com/openstack/cinder/blob/master/cinder/api/v2/volumes.py
https://github.com/openstack/cinder/blob/master/cinder/api/v1/volumes.py
https://github.com/openstack/cinder/blob/master/cinder/api/v1/snapshots.py
https://github.com/openstack/cinder/blob/master/cinder/api/v2/snapshots.py
https://github.com/openstack/cinder/blob/master/cinder/volume/flows/manager/create_volume.py