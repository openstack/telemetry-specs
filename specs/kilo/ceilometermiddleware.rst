..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Example Spec - The title of your blueprint
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/ceilometermiddleware

The swift middleware packaged in Ceilometer enables the generation of swift
metrics which can be used to properly meter Swift activity. This cross
project dependency causes issues during the upgrade chain.

Problem description
===================

Currently the swift middleware exists in the Ceilometer package as it leverages
the functionality of the pipeline. This process is rather heavyweight as it
moves a lot of the processing (building samples, republishing to correct
targets) to the Swift service where as it should really exist on Ceilometer's.
Additionally, having the middleware live in the Ceilometer package brings in
a lot of additional dependencies unrelated to the middleware which is causing
upgrade issues[1].

Proposed change
===============

This spec is to propose branching off swift middleware into its own library:
ceilometermiddleware. This work is similar to the keystonemiddleware library
which split the auth_token middleware into its own library to avoid additional
requirements of a larger package.

Additionally, the middleware will be modified to solely compute the required
metric values as it does now and publish a single notification straight to the
message queue rather than loading in the Ceilometer pipeline and pusblishing 3
separate samples as it currently does.

Alternatives
------------

- We can continue to publish via the pipeline but it is far too verbose and
  will not solve all dependency issues.
- Have swift own it's own metrics and have it exist swift package. This is
  dependent on swift accepting something not completely scoped to swift
  internal functionality.
- Drop support of swift middleware meters (ie. we won't test it but it'll just
  existed if a deployer does want to try to use it)

Data model impact
-----------------

We need to capture swift meters in notification agent.

REST API impact
---------------

None

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

There should be a performance increase we'd be using notifications which are
async and we'd just be sending one notification (which would be picked up by a
listener in ceilometer and parsed into the 3 samples we expect). this moves all
the processing overhead to ceilometer's services (where it should be)

Other deployer impact
---------------------

None.

Developer impact
----------------

New middleware should be place in new library

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  chungg

Other contributors:
  osanai-hisashi

Ongoing maintainer:
  chungg

Work Items
----------

- create ceilometermiddleware library
- move edited swift middleware to new lib
- add listener to ceilometer to pick up new notification and parse into
  three samples
- create shell in middlware that exists in ceilometer to use new lib
- update devstack to use new lib

Future lifecycle
================

look at statsd approach.

Dependencies
============

None.

Testing
=======

- Add new tests to ceilometermiddleware
- continue to leverage existing tempest tests

Documentation Impact
====================

- reference new middleware

References
==========

[1] https://bugs.launchpad.net/ceilometer/+bug/1403024
