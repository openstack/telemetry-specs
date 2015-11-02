..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==============
Tempest Plugin
==============

https://blueprints.launchpad.net/ceilometer/+spec/templest-plugin

As with devstack and grenade, the tempest program is encouraging
projects to manage their own tempest tests via code hosted in each
project's code repository. This spec proposes that we do this for
those projects living under the ceilometer umbrella.

Problem description
===================

Managing tempest tests within tempest itself in the world of the big
tent is problematic: The people who want the tests to exist and the
people who review tempest are not congruent. This can lead to delays
in getting relevant tests in place. When projects host their own
tests as plugins the projects decide what happens when, according
to their own requirements.

For ceilometer which is now spread over a few repos and has
increasingly complex integration scenarios these problems are
multiplied.

Proposed change
===============

Using the `tempest plugin`_ documentation and `tempest-lib`_ new tempest
tests will be created in ceilometer related repos that duplicate
existing telemetry-related test functionality from tempest. Initially
these will be done in the ``ceilometer`` repo itself. There is a `sahara
example`_ and a `manila example`_ that can be used for guidance.

When these have proven themselves the test in tempest will be
removed and the project should be in a position to greatly increase
the number of tempest tests that are available and run.

Alternatives
------------

We can leave things as they are now but this is contrary to the
goals of the tempest project.

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

If there is improved ceilometer coverage in tempest then operators
who use tempest to validate their clouds will have increased confidence
in more services in their cloud.

Developer impact
----------------

It will be easier for developers to find and improve tempest tests
related to ceilometer.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  chdent

Other contributors:
  you

Ongoing maintainer:
  The ceilometer team

Work Items
----------

* Evaluate pre-existing work from other projects.
* Create a version for ceilometer.
* Create experimental gate jobs to test them against commits.
* Remove existing tempest tests.
* Update gate jobs.

Future lifecycle
================

The ceilometer team will need to be responsible for the ongoing care
and feeding of the in-tree tempest tests.

Dependencies
============

None.

Testing
=======

See work items above.

Documentation Impact
====================

Tempest documentation will need to be updated to reflect that use of
ceilometer will require referencing the plugin.

References
==========

* `tempest plugin`_
* `tempest-lib`_
* `sahara example`_
* `manila example`_

.. _tempest plugin: http://docs.openstack.org/developer/tempest/plugin.html
.. _tempest-lib: http://docs.openstack.org/developer/tempest-lib/
.. _sahara example: https://github.com/openstack/sahara/tree/master/sahara/tests/tempest/scenario/data_processing
.. _manila example: https://github.com/openstack/manila/tree/master/manila_tempest_tests
