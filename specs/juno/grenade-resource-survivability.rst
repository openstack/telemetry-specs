
..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============================
Grenade Resource Survivability
==============================

https://blueprints.launchpad.net/ceilometer/+spec/grenade-resource-survivability

Integrated projects are required to participate in the `Grenade`_ upgrade
testing harness. In addition to testing the upgrades themselves Grenade has
facilities, called javelin, for testing survivability of resources through the
upgrade process. Ceilometer needs to participate in this testing.

.. _grenade: https://github.com/openstack-dev/grenade

Problem description
===================

To be certain that Ceilometer is robust across an upgrade it must be possible
to process metrics and events from resources that exist before and after
the upgrade. Grenade provides a feature named javelin which is designed
to allow assertions that confirm resource A, present prior to the upgrade,
is present after the upgrade.

In the Juno cycle Grenade is being updated to `support javelin2`_ which, unlike
the original javelin, should work well and has a declarative syntax for making
assertions.

A `previous spec`_ describes adding basic Ceilometer support to Grenade. This
spec is specifically about adding testing via javelin2.

.. _support javelin2: https://review.openstack.org/#/c/96445/
.. _previous spec: https://github.com/openstack/ceilometer-specs/blob/master/specs/juno/grenade-upgrade-testing.rst

Proposed change
===============

Add support for Ceilometer resource checking to javelin2. This involves two
types of changes (detailed below): Adding support for Ceilometer queries in
the `javelin code`_ and adding Ceilometer specific entries to the resource
definitions.

The main check that will be facillitated by javelin2 is ensuring the sanity of
api queries with a time range that spans the entire window of time within which
the Grenade test runs (e.g. -+12 hours from now).

.. _javelin code: https://github.com/openstack/tempest/blob/master/tempest/cmd/javelin.py

Alternatives
------------

It may be that Ceilometer queries do not map well to the resource model in use
in javelin2. If this is the case then it might make sense to architect some
other kind of before and after upgrade test specifically for Ceilometer. Doing
so would be a shame, though. Better to make javelin2 more flexible.

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

While Ceilometer has something of a reputation in this area, because it will
already be running in the Grenade environment, no additional impact is expected
by adding javelin tests.

Other deployer impact
---------------------

None.

Developer impact
----------------

As Ceilometer features grow or change, adjustments in the javelin2 check tests
may need to be made.

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
  emilienm

Ongoing maintainer:
  chdent

Work Items
----------

* Determine optimal form for handling ceilometer queries within javelin2. At a
  gross level there are two options: 1) Including the ceilometer queries as
  code within ``tempest.cmd.javelin`` itself, either built in or as plugin.
  2) Representing the queries declaratively in a checks section of the
  ``resources.yaml`` file that could potenetially be used by other projects.

* Add the queries (as described in `Proposed change`_) in whatever form is
  chosen.

As a first pass, option 1 above is most expedient. If chosen the other options
and the related `review discussion`_ should be considered for the future.

.. _review discussion: https://review.openstack.org/#/c/100575/

Future lifecycle
================

Ongoing maintenance of the Ceilometer portions of the javelin2 tests will be
the responsibility of the Ceilometer project team.

Dependencies
============

These changes require `javelin2`_ which was merged into Tempest on 30, May
2014.

.. _javelin2: https://blueprints.launchpad.net/tempest/+spec/javelin2


Testing
=======

Est quod est.

Documentation Impact
====================

None.

References
==========

* Ceilometer blueprint for `Grenade Upgrade Testing`_.
* Javelin 2 `blueprint`_, `spec`_ and `code`_.

.. _Grenade Upgrade Testing: https://blueprints.launchpad.net/ceilometer/+spec/grenade-upgrade-testing
.. _blueprint: https://blueprints.launchpad.net/tempest/+spec/javelin2
.. _spec: https://review.openstack.org/#/c/96445
.. _code: https://github.com/openstack/tempest/blob/master/tempest/cmd/javelin.py
