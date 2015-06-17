..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============
API No Pipeline
===============

https://blueprints.launchpad.net/ceilometer/+spec/api-no-pipeline

The ceilometer API server allows new samples to be posted via the
API. These are then processed through the pipeline and then
dispatched as required. This means the API server must use the
pipeline code and have a ``pipeline.yaml`` file, raising the
complexity of the code and the testing and install profile of the API.
An alternative would be for the API server to publish notifications
for the meters it receives which would then be processed by the
notification agent (which has a pipeline of its own) and then
dispatched.

Problem description
===================

Including the transformation pipeline in the API server increases
complexity on at least three dimensions for a single feature in the
API (creating samples) which is not commonly used (most samples are
produced via polling agents and notifications).

The use of the pipeline creates dependencies in code, testing and
installation that could be avoided. In code we must include the
pipeline modules and classes. In testing and installation we must
have a ``pipeline.yaml``.

In the installation case, if there are multiple API servers, the
``pipeline.yaml`` must be kept in sync across the hosts on which
those servers are installed. That's an unnecessary burden.

Proposed change
===============

One way to resolve this complexity is to not use the pipeline in the
API server at all. Instead when new samples are posted, format them
as notifications and push them onto the notification bus to be
retrieved by the notification agent. The pipeline within that agent
will then do any expected transformations.

For convenience of testing, an optional flag will be add that can enable
posting samples directly to storage to avoid notification-agent process.

Alternatives
------------

An alternative which accomplishes the state goal would be:

* A more complex solution would be create notifications on the bus
  that are targeted to a specific transformation service. Such a
  service would only listen for ceilometer generated measurement
  notifications, transform them according to a pipeline description
  and then dispatch them onward to storage or other processing. This
  model would skip the notification agent entirely.

  Pro: extends the long term plan of decomposing to smaller pieces.
  Con: adds to the complexity of this change, increasing time to
  completion and risk of error.

Another alternative which are related but not quite the same include:

* The issues with testing of the API requiring a ``pipeline.yaml``
  could be addressed by having a code-based set of pipeline defaults
  which are used in the absence of a ``pipeline.yaml`` file (and
  could also be used to generated the default ``pipeline.yaml``).


Data model impact
-----------------

The proposal does not impact the structure of the data, but it would
impact the flow of the data. Testing will be required to see what
kind of impact this will have on the timeliness of data acquisition.

REST API impact
---------------

Samples created via the API will take a slightly different path between
the API and their eventual storage.

An optional boolean parameter 'direct' will be added to the post-samples API
that allow users posting samples directly to storage and avoid notification
agent processing(will skip pipeline), this is mainly used for testing.

Security impact
---------------

None beyond existing concerns.

Pipeline impact
---------------

Pipeline transformations that had been happening in the api server
will happen in the notification agent.

Other end user impact
---------------------

None.

Performance/Scalability Impacts
-------------------------------

This will potentially introduce some latency in the time between
when a sample is posted to the API and when it lands in the
data store.

Other deployer impact
---------------------

Deployers of the API server will no longer need to deploy a
``pipeline.yaml`` alongside the API server.

And deployers need to config the 'notification_driver' in ceilometer.conf if
need posting samples through notification-agent, such as::

    notification_driver = messaging

Developer impact
----------------

This change may increase the amount of asynchrony in some tests.
For example the time between posting a sample and being able to
retrieve that sample may become more unpredictable. As it is already
unpredictable any tests which rely on immediate retrieval are bad
anyway, so we should fix that.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  liusheng

Other contributors:
  chdent

Ongoing maintainer:
  liusheng, chdent

Work Items
----------

* Ensure existence of robust tests of posting meters that are transformed by
  the pipeline.
* Replace pipeline-based publishing in the API code with a notifier.
* Update API tests to deal with things like mocked pipelines that no
  longer exist.
* API option to perform direct writing to storage

Future lifecycle
================

The members of the Telemetry program will do any required ongoing
maintenance for this feature.

Dependencies
============

No new dependencies.

Testing
=======

In addition to existing unit tests of the API, the new `declarative
http tests`_ can be used to confirm that posted samples continue to
have expected transformations.

.. _declarative http tests: https://github.com/openstack/ceilometer-specs/blob/master/specs/kilo/declarative-http-tests.rst

Documentation Impact
====================

Deployment documentation may need some slight updates to indicate
the change in configuration file requirements for the api server.

References
==========

None
