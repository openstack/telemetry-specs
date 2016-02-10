..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================
Pollsters No Transform
=======================

https://blueprints.launchpad.net/ceilometer/+spec/pollsters-no-transform

The ceilometer polling agents currently perform two roles: They poll various
services to generate samples and they optionally transform those samples, using
pipelines defined by ``pipeline.yaml``, to compose other samples eventually
publishing them as measurements. This latter transformation functionality raises
the complexity of the pollster code. An alternative would be for the pollsters
to send notifications for the meters it creates and have them be processed
by the notification agent (which has a transformation pipeline of its own) and
then be published.

Problem description
===================

Including the transformation pipeline in the pollsters increases complexity
in the pollsters and duplicates functionality found in the notification agent.

Proposed change
===============

One way to resolve this complexity is to not do transformations in the
pollsters. Instead when new samples are polled, format them as notifications
and push them onto the notification bus to be retrieved by the notification
agent. The pipeline within that agent will then do any required transformations.

Besides clarifying the focus of the polling agents it is likely this change
will make it easier to test and build performance improvements both in code
and in deployments. This is murky speculation but it is challenging to test the
theory without doing the work.

We also get an opportunity to review and audit the polling code, which is a
useful thing to do on a regular basis.

Alternatives
------------

The primary alternative is to leave things as they are. Obviously if we do this
we lose the benefits described above.

Data model impact
-----------------

The proposal does not impact the structure of the data, but it would
impact the flow of the data. Testing will be required to see what
kind of impact this will have on the timeliness of data acquisition.

REST API impact
---------------

No direct impact on the resources presented by the API.

Security impact
---------------

None beyond existing concerns.

Pipeline impact
---------------

Describing pipeline impact is somewhat complex because the word pipeline
can be used:

* to describe the overall process whereby a suite of samples is
  gathered and transformed
* to identify the pairing of a ``source`` with a ``sink`` in the
  ``pipeline.yaml`` file
* to point at the file ``pipeline.yaml`` which configures many
  aspects of gathering, transforming and publishing samples as well as
  transforming and publishing samples that have been produced from
  notifications

In this change transformation and publishing will be localized in the
notification agents. Gathering and producing of samples will be done by
polling agents, the api server (via posting meters) and the notification
agents.

In order to ease migration the polling agents will continue to use
the ``pipeline.yaml`` file to configure the sources they will poll
and the intervals at which that polling will be done. The same file
may be used, or a new file without any sinks. The idea is that both
should work.

The most significant difference is that the existing validation
which confirms that there is a pairing between ``source`` and
``sink`` will be removed. The polling agents may poll whatever
they like. If the notification agent doesn't want the samples that
are generated it can simply drop them on the floor. This provides
two benefits:

* it makes the pipeline (on the polling side) less complex
* it focuses authority in the notification agent

Other end user impact
---------------------

None.

Performance/Scalability Impacts
-------------------------------

There is hope that this will decrease the footprint of the pollsters making it
easier to deploy many of them, thus increasing the overall performance of the
polling system.

On the negative side there could be increased load on both the messaging and the
notification agents but this could be compensated for by:

* Having the pollsters publish their notifications on their own topic or even
  bus.
* Deploying more notification agents (in a horizontal fashion).

Other deployer impact
---------------------

Deployers will need to be aware of their options for deploying suitable numbers
of notification agents.

Developer impact
----------------

Testing sample transformation from point of sample creation to point of storage
will become more complex as it will traverse several services. With the advent
of in-tree functional testing there is a place where this testing can and
should be done. Having those tests will be very helpful in the long run despite
the initial pain in setting them up.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  chdent

Other contributors:
  you and all your friends

Ongoing maintainer:
  chdent

Work Items
----------

* Ensure existence of robust tests of both polling and notification agents.
* Replace source and sink handling in the pollster code with a simpler read of
  the source descriptions in pipeline.yaml (leaving open the future option of
  using a different file). Create samples will be republished as notifications.
* Update documenation to reflect new architecture.

Future lifecycle
================

The members of the Telemetry program will do any required ongoing
maintenance for this feature.

Dependencies
============

No new dependencies.

Testing
=======

Testing as described above.

Documentation Impact
====================

Deployment documentation will need to be updated to reflect the connection
between the polling and notification agents, the option to use a custom
messaging bus and other opportunities for customization.

References
==========

None at this time.
