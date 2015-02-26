..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================================
Create functional and integration tests in ceilometer project
=============================================================

https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-functional-tests

At this moment there are no integration tests for Ceilometer outside of Tempest,
which are fully covering all its functionality. On Paris summit it was decided
to move part of the tests (and the implementation of new ones) in the code tree of
Ceilometer itself. Also tests for different databases should be moved to this
module as well (currently they are living in the folder of unit tests and
this doesn't correspond, generally speaking, to their aim). And all unit tests
should be moved from ceilometer/tests/ path to ceilometer/tests/unit folder,
so that ceilometer/tests tree contains only three folders: unit, functional,
integration.

It is why we should develop integration tests and create Jenkins job for
them, which will run these tests, located in a separate module within
the Ceilometer code tree (tests for different backends will be moved
to this module as well). These tests will be designed in DSVM-style.
Tests that require communication with real other OpenStack services,
will follow Tempest practice to use master branches for these components.

Problem description
===================

Implementation of the plan will take a long time because of many subtasks
and interaction with other projects (in particular with OpenStack infra).
In this approach we may implement tests which are not assumed in tempest:
* grey/white-box tests that are not pure API tests and may assumed some
knowledge of the implementation
* tests that require acceleration of the polling interval in order to complete
within a reasonable time, e.g. changing the cpu_source interval
in the pipeline.yaml and restarting the compute agent so as
to gather sufficient cpu_util datapoints to drive alarming tests
* tests that depend on the timeliness of incoming notifications
from other OpenStack services
The implementation of this task will greatly increase test coverage
for ceilometer project, including the features which weren't tested before.

Proposed change
===============



Alternatives
------------

Contribute new integration tests to the tempest project. This approach
does not satisfy us because OpenStack QA team rejects too specific
integration tests for projects, as only projects teams can decide
if they are good or not.

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

None

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
  vrovachev <vrovachev@mirantis.com>

Other contributors:
  chdent <chdent@redhat.com>

Work Items
----------

Work items:
* Add ceilometer/tests/{integration|functional} testcases
that use the python-clients directly.
* Move ceilometer/tests testcase to ceilometer/tests/unit.
* Move all existing unit tests to ceilometer/tests/unit.
* Move existing DB scenario tests to ceilometer/tests/{integration|functional}.
* Add new tox target(s)(functional, integration and others
if necessary) to ceilometer/tox.ini.
* Add {pipeline}-ceilometer-dsvm-functional-{datastore} job template
to os-infra/project-config/jenkins/jobs/ceilometer.yaml
where datastore in ['sqlalchemy', 'mongodb', ‘hbase’].


Future lifecycle
================

None


Dependencies
============

None


Testing
=======

None

Documentation Impact
====================

None

References
==========

None
