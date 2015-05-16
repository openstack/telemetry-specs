..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================
Spliting Ceilometer alarming
============================

https://blueprints.launchpad.net/ceilometer/+spec/split-ceilometer-alarming

Ceilometer evolved from a a simple meter gathering component to a lot of
different component doing different things. The storage layer has been
abstracted during Juno and Kilo and is going to be handled by Gnocchi. This
spec proposes that the work continues so that the alarming subsystem of
Ceilometer is split out as an autonomous component connected to the rest of
Ceilometer and Gnocchi.

This should help with code review, maintenance, deployment, upgrade… as having
several small service-oriented components to manage is going to be easier than
a single monolithic project.


Problem description
===================

Currently, Ceilometer features can be grouped in different components, mainly:

* Metric polling and events storage (Ceilometer polling and notifications agents and collector)
* Metric storage (Gnocchi)
* Alarm evaluation and triggering

These all 3 components are currently merged in one code base and repository,
whereas they are different service talking to each other. This leads to a bit
of spaghetti code and architecture when digging the project.


Proposed change
===============

We want to improve project management, code review, upgrade, etc, by spliting
the alarming subsystem out of Ceilometer main repository and having it living
its own life in its own repository.

The new repository and project will be called either *aodh*.

Alternatives
------------

Do nothing and continue growing the project until it collapses.

Data model impact
-----------------

The main data model impact was storing alarms in a different database than the
samples and events. This change already been implemented in Juno.

REST API impact
---------------

The alarming API should not change and continues to be contained in
`/v2/alarms`. However, the `ceilometer-api` endpoint and daemons is going to be
replace by a specific REST API daemon only implementing the alarming subsystem.
This means a new Keystone endpoint will need to be created and registered.

We'll also provide a documentation about how to support configuring a proxy
that can route requests to the Ceilometer API endpoint or the new alarm API
endpoint.

Security impact
---------------

None

Pipeline impact
---------------

None

Other end user impact
---------------------

As stated above, this will require a new Keystone endpoint to manipulate
alarms.

Performance/Scalability Impacts
-------------------------------

It's easy to envision a better scalability if anything. It'll be easier to
scale the API endpoint for alarming only. Also, performances might be a little
improved as the alarming codebase could be smaller.

Other deployer impact
---------------------

The configuration file used by the alarming subsystem when it'll be part of its
own project/repository should be split out of `ceilometer.conf`. That means a
new configuration file, but close to the current `ceilometer.conf` file, should
be created.

Other than that, no option should be added or removed.

The daemon will remain the same and the API will be deployed in the same way –
either with a (new) daemon either through the recommended WSGI interface.

The subsystem will be released independently than Ceilometer on its own
schedule, like it's already the case for Gnocchi and now other OpenStack
projects. A coordinated release at the end of each cycle will happen too.

Developer impact
----------------

The new code repository hosting Ceilometer alarming will require a new core
review team which will be composed of a copy of the current ceilometer-core
reviewers.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  jdanjou


Work Items
----------

1. Isolate Ceilometer alarming codebase from the other part of Ceilometer.
2. Create a new Python package and repository with that isolated codebase.
3. Publish this repository as openstack/<projectname>
4. Enable Tempest, devstack, grenade, etc, tests
5. Deprecate alarming code from Ceilometer
6. After Liberty: remove alarming code from Ceilometer
7. Contribute to the a downstream switch-over effort to migrate existing
   Fedora/Deb packages and puppet modules.


Future lifecycle
================

This Ceilometer subproject will be able to be released on its own like the rest
of the OpenStack projects/components.

Work items 4 can be removed if the project agrees to not having a deprecation
period. Otherwise, core reviewer will have to be careful that the no code
changes/fixes are merged into Ceilometer rather than the new repository.

Dependencies
============

None

Testing
=======

The testing coverage should remain the same, using unit tests and functional
tests with Tempest.

Documentation Impact
====================

The documentation will have to be updated to include this new component in lieu
of the current Ceilometer alarm.

References
==========

* Liberty Summit Etherpad https://etherpad.openstack.org/p/ceilo-multi-identity

* Name discussion http://eavesdrop.openstack.org/meetings/ceilometer/2015/ceilometer.2015-06-04-15.00.log.html
