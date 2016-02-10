..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Declarative HTTP API Tests
==========================

https://blueprints.launchpad.net/ceilometer/+spec/declarative-http-tests

This spec proposes a suite of "grey-box" tests for the Ceilometer HTTP
API that are driven by one or several human-readable text files that
declare URL requests alongside expected responses. These tests will
provide end to end tests of API endpoints that in addition to confirming
the functionality of the API will provide a cogent and readable overview
of the HTTP API.

The suite of tests augments rather than replaces existing tests
presuming that several small tools, each doing a focused job, are more
effective than one job doing everything. In this case the balance of the
focus is on traversing the breadth of the HTTP API to confirm it is in
rude health and demonstrating expected behaviors, not validation of data
provided by the API (unit tests ought to do that).

Transparency over the API ought to lead to better testing and
ultimately better APIs.

Problem description
===================

Existing Ceilometer `HTTP API`_ unit tests are enshrouded by and
encumbered with Python code and traditional unit test infrastructure.
These amount to noise when the purpose of a test is both to test *and*
to make visible the testing of the HTTP API. What should be most visible
is HTTP and the API. When these things are not visible, the
effectiveness of the tests as tools for discovery by a developer who
needs to make changes is limited.

The integration testing harness, `Tempest`_, suffers from some of
the same problems as above while also undergoing a refinement in purpose.
Rather than testing *all* the things, the future Tempest will evolve
to efficiently test the integration of *most* things. In this future
projects which wish to affirm their systems in detail will need to
do that testing themselves.

Tempest also more strictly requires that testcases have no knowledge
of the internals. This proposal would be free of such
constraints, which may prove convenient for test setup mechanics.

In addition, Tempest is now "branchless", so that the same test
branch is run against stable/{icehouse|juno} and master. This
requires that tests for newly testable features either discover the
availability of such features dynamically, or are explicitly
configured to be skipped depending on the branch Tempest is being
run against (neither option being ideal). This proposal would be free of
such complications, as the tests would live in-tree and thus be branched
in the same way as the rest of the codebase.

.. _HTTP API: https://github.com/openstack/ceilometer/tree/master/ceilometer/tests/api
.. _Tempest: https://github.com/openstack/tempest

Proposed change
===============

To address the shortcomings in HTTP testing (readability and
completeness) a new target for ``tox`` will be defined that runs a
suite of HTTP tests which are expressed as HTTP requests with
expected responses. The exact format should be determined from
discussion as these sorts of things always cause a lot of
contention. Best to get that out of the way early.

This same suite of tests will also be registered as a check job to
be run against patches proposed to the code review process.

In the ideal form the tests will run all the usual HTTP methods
against every URL made available by all the active endpoints
presented over HTTP by Ceilometer. This means that it should cover
both the pending v3 API that will be presented by Gnocchi as well as
the soon to deprecated older APIs.

To ensure the tests are lightweight and fast no web server will be used
and only one of whichever lightweight datastore (mongo, sqlite3) for
limited persistence. These tests are for testing the HTTP API aspect of
Ceilometer and as such are not scenario tests: We don't need to run
these tests against every available datastore.

"No web server" can be achieved by using `wsgi-intercept`_,
`WebTest`_ or similar. The choice of tool should be driven by
determining which is best able to facilitate easy translation from
the declarative format to tests that can be handled by the runner.

The declarative format of the tests need to balance between easy for
a human to write and read and easy for a computer to parse. There
are two general classes of style:

* Something that looks a bit like the output of ``curl -v``. This
  has the advantage of looking just like HTTP on the wire but the
  disadvantage that it is hard to express partial expectations or
  partial requests.

* A list of dictionaries that represent requests and expected
  responses. This is easy to express in a common format like
  YAML, can deal with partial information well, and handle a form of
  inheritance. The author has some `good success`_ with this format
  so tends to prefer it.

Assuming the latter format a possible implementation would include:

* A test runner that parses YAML files to generate tests which will
  be captured by the testing harness. This runner, besides making
  the HTTP requests and processing responses, will be responsible
  for setup: establishing the necessary services and other
  prerequisites of the tests.

* A directory including multiple YAML files each representing a
  transaction/scenario of some kind in the API *or* all the routes
  in one version of the API.

Note that this proposal makes no mention of client code because it is
direct HTTP requests that are being tested, not calls to a client
library. This is entirely the point: a client library hides HTTP and
what we are trying to do here is expose HTTP interactions to the harsh
light of review.

There are a few questions which will need to be resolved to reach a
complete solution. It should be possible to resolve these in flight.
The questions include:

* What format for the tests?
* How to handle authN and authZ in a minimal fashion? Ideally keystone
  won't be running. If that's the case, what needs to be done?
* How to aggregate request and response groups into single tests?
  One option is for each YAML file to be treated as a single test.

.. _wsgi-intercept: https://pypi.python.org/pypi/wsgi_intercept
.. _WebTest: http://webtest.readthedocs.org/
.. _good success: https://github.com/tiddlyweb/tiddlyweb/blob/master/test/httptest.yaml

Alternatives
------------

There are several alternative options to a standalone suite of
declarative HTTP tests within the Ceilometer code tree:

* Propose a spec `to qa`_ for building a general purpose solution to
  the problem of declarative HTTP tests in-tree. Long term this
  would be a great outcome, but this outcome seems more likely if
  people are able to see a working version against which
  improvements can be made.

* Don't use a standalone suite, have the tests integrated with a
  larger set of in-tree functional and/or integration tests. While
  this may be a good idea from a management of complexity standpoint
  it has limited value in ensuring the tight focus of purpose that
  is required for something to be good at its purpose.

* Don't use declarative tests, instead continue with
  object-oriented ``unittests`` style that is the norm throughout many
  tests. Doing this will defeat the purpose of the proposal (exposing
  and expressing HTTP) so is not recommended.

* Run the tests against various storage and web server scenarios to
  achieve maximum coverage across a variety of situations. While
  this may have value, it would lead to slow tests and increase
  complexity of the harness significantly. Slow tests discourage people
  running them and complexity leads to breakage.

* Do nothing. Use existing unit tests and tempests test. This is
  not a good option as neither the existing unit tests nor Tempest tests
  present good (and readable) coverage.

.. _to qa: https://github.com/openstack/qa-specs

Data model impact
-----------------

None. The data model should be invisible to these tests.

REST API impact
---------------

These tests will not add to the API, but with luck will improve it.

Security impact
---------------

None.

Pipeline impact
---------------

None.

Other end user impact
---------------------

Writing and running these tests may demonstrate that the API is
confusing and hard to use which may then lead to making it better.
This would be awesome.

Performance/Scalability Impacts
-------------------------------

The additional gate check job may have some impact on performance there
but these changes are not expected to impact the performance of
Ceilometer itself.

Other deployer impact
---------------------

None.

Developer impact
----------------

If a developer adds to or changes the HTTP API those changes will need
to reflected in this test suite.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  chdent

Other contributors:
  dbelova

Ongoing maintainer:
  chdent

Work Items
----------

* Decide on the declarative format.
* Determine extent or depth of testing (are web servers being used?
  other data store?).
* Write harness.
* Write first round of tests.
* Integrate with tox and testr.
* Create gate job.


Future lifecycle
================

As features are added and removed to and from the API tests for
those features will need to be changed in this suite. Most of the
time that should be done by the implementor of the feature but there
will be times that larger cleanups are required.

Similarly if storage handling becomes a part of the test suite, then
as new storage systems are implemented (or removed), they will need
to be handled.

Dependencies
============

This work is self-contained but may add to the libraries required
for testing (e.g. wsgi-intercept).

Testing
=======

As a suite of tests, this system will test itself. It should,
however, include some sanity tests within itself to make sure it is
behaving.

Documentation Impact
====================

None.


References
==========

See links above and:

* https://wiki.python.org/moin/PythonTestingToolsTaxonomy for prior
  art and potential formats.
* http://lists.openstack.org/pipermail/openstack-dev/2014-October/049056.html
  related mailing list thread.
