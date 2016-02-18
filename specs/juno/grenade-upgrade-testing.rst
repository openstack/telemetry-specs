..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Grenade Upgrade Testing - Phase 1
=================================

https://blueprints.launchpad.net/ceilometer/+spec/grenade-upgrade-testing

Integrated projects are required to participate in the `grenade`_ upgrade
testing harness. Ceilometer was integrated before these requirements were
added but the requirements apply retroactively. Therefore, ceilometer must be
added to the harness.

.. _grenade: https://github.com/openstack-dev/grenade

Problem description
===================

Testing with grenade involves:

* Installing a basic DevStack of the old version, confirming it
  with smoke and scenario `tempest`_ tests.
* Establishing a javelin project which makes assertions about
  resources in the project.
* Shutting down the original installation, upgrading to the new
  version of the code but not configuration, applying the relevant
  database migrations to update the schema, and repeating the
  `tempest`_ tests.
* Testing the javelin project again to assert the integrity of
  resources across the upgrade.

This document considers the upgrade and `tempest`_ testing but not the javelin
testing. Javelin will be addressed in another (forthcoming) document.

Prior to the Juno cycle ceilometer was neither a default service in DevStack
nor enabled in tempest. The relevant configuration must be changed to allow
testing in Grenade.

.. _tempest: https://github.com/openstack/tempest

Proposed change
===============

Enable ceilometer in Grenade (see Work Items for details).

Alternatives
------------

Given the commitment to Grenade and the project graduation requirements, this
is the way to go.

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

These changes will mean a more positive upgrade experience for end users.

Performance/Scalability Impacts
-------------------------------

There have been concerns that running ceilometer in the gate will negatively
impact performance there. This had been true but recent changes in the
sqlalchemy driver and database schema have shown enough improvement that
ceilometer can be turned on.

Other deployer impact
---------------------

None

Developer impact
----------------

Active testing will mean the discovery of bugs that someone will have to fix.

Implementation
==============

Assignee(s)
-----------

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  emilienm

Other contributors:
  vrovachev
  dbelova

Ongoing maintainer:
  chdent

Work Items
----------

* Add upgrade calls to ``grenade.sh``.
* Add ceilometer upgrade scripts to grenade.
* Add ceilometer directories to grenade cleanup procedures.
* Ensure ceilometer data dumped between upgrades.

Note: These work items are captured in a `pending patchset`_

.. _pending patchset: https://review.openstack.org/#/c/94468/

Future lifecycle
================

As/when ceilometer expands to include additional services, it will be necessary
to adjust devstack and grenade to manage (stop, start, cleanup) those services.

Dependencies
============

Other than those listed above, no additional dependencies.

Testing
=======

These changes improve testing and will be validated there.

Documentation Impact
====================

None.

References
==========

None.
