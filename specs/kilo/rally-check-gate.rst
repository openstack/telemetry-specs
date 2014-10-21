..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

================
Rally Gate Check
================

https://blueprints.launchpad.net/ceilometer/+spec/rally-gate-check

`Rally`_ provides support for what it calls `rally-gate`_ jobs. These
automate a set of non-voting, custom benchmark scenarios that
effectively make it possible to observe the performance impact of
changes. Ceilometer's overall performance is the result of complex
interactions in several systems, automating benchmarks of these
systems will save time and provide useful information.

.. _Rally: https://wiki.openstack.org/wiki/Rally
.. _rally-gate: https://wiki.openstack.org/wiki/Rally/RallyGates

Problem description
===================

At this time there are no generally available automated benchmarks
for Ceilometer performance. Both developers and deployers would
appreciate knowing if code changes will improve or damage
performance.

Proposed change
===============

A non-voting gate check job will be created to run a set of
benchmark scenarios against each Ceilometer patchset. This job is
controlled through multiple changes:

* A ``rally-scenarios`` directory in the Ceilometer code tree
  containing a ``ceilometer.yaml`` file that describes the Rally
  scenarios to be used in the job.
* An addition to `projects.yaml`_ that adds ``rally-jobs`` to the
  ``jobs`` list.
* An update to `zuul layout`_ that adds
  ``gate-rally-dsvm-fake-ceilometer`` to the ``check`` list of the
  ``openstack/ceilometer`` section.

Rally includes a suite of pre-built scenarios which can run against
Ceilometer. At the moment these primarily interact with Ceilometer
over the API. This is not a requirement. Scenarios can do as much or
as little as required and can be added as necessary as plugins in the
``rally-scenarios`` directory. As long the code is properly decorated
by Rally code, the scenarios can be measured.

Initially this change will implement the basic structure for doing
benchmarks, using simple API tests to flesh out and confirm the
setup.

Once confirmed, additional scenarios will be created to measure:

* Latency in polling and notification agents (getting samples into the
  pipeline).
* Latency in ingestion (getting samples into the data store).

These latter measurements will require more complex scenario code that
is responsible for creating and destroying VMs, volumes and other
resources.

Scenario code which proves effective should migrate from plugins to
the Rally codebase to allow general use.

.. _projects.yaml: https://github.com/openstack-infra/project-config/blob/master/jenkins/jobs/projects.yaml
.. _zuul layout: https://github.com/openstack-infra/project-config/blob/master/zuul/layout.yaml

Alternatives
------------

Alternatives include:

* The Rally team is automating performance checking to gather historical
  performance data to share with projects.  This serves a similar but not
  identical purpose to the changes propsed here: it would help to evaluate
  the improvement of Ceilometer at large but not specific changes.

* Use the Rally job as an experimental job, only run when a "check
  experimental" comment is left in gerrit.

* Once this initial spec has been implemented and the principles
  involved validated in the ceilometer context, it may be worthwhile
  to implement what are called "SLA checks" via Rally. These allow a
  check job which can fail based on a severe degradation in
  performance. These are not being done now to first gain experience
  with Rally as well as to comparmentalize work in actionable
  chunks.

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

Some end users may be able to use the benchmark information to make
decisions about how they configure their installations. It seems
unlikely, however, that they will want to review this data from
gerrit.

Performance/Scalability Impacts
-------------------------------

These tests will provide a safety net to guard against
degradations in performance and provide data to identify which
changes have caused issues if there is degradation.

Other deployer impact
---------------------

None.

Developer impact
----------------

As features are added, the Rally scenarios may be insufficient and need
to be updated. Ideally the scenarios will be high-level enough that this
will be rare. When they are not it will be up to the feature developer
to update the Rally scenarios and, as necessary, add Rally plugins.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  chdent

Other contributors:
  boris-42, rediskin, sileht

Ongoing maintainer:
  chdent

Work Items
----------

In addition to the steps listed in the Proposed Solution section
above the major chunk of work is in selecting and/or creating the
necessary Rally tasks. Rally itself includes many sample scenarios
and runners and includes a `gate check`_ in its own tree which
includes several Ceilometer checks that may be used as starting
point.

.. _gate check: https://github.com/stackforge/rally/tree/master/rally-scenarios

Future lifecycle
================

As stated in "Developer Impact" new features may require new
scenarios.


Dependencies
============

* Rally.

Testing
=======

The proposal adds a new (non-voting) gate job which will act as its
own test.


Documentation Impact
====================

None.

References
==========

* An existing related patchset: https://review.openstack.org/#/c/132649/
