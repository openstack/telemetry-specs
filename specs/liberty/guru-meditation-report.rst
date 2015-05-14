..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================
Adopt Oslo Guru Meditation Reports
==================================

https://blueprints.launchpad.net/ceilometer/+spec/guru-meditation-report

This spec adopts Oslo Guru Meditation Reports in Ceilometer. The feature will
enhance debugging capabilities of all Ceilometer services, by providing an easy
and convenient way to collect debug data about current threads and
configuration, among other things, to developers, operators, and tech support
in production deployments.

Problem description
===================

Currently, Ceilometer doesn't provide a way to collect state data from active
service processes. The only information that is available to deployers,
developers, and tech support is what was actually logged by the service.
Additional data could be usefully used to debug and solve problems that occur
during Ceilometer operation. We could be interested in stack traces of green
and real threads, pid/ppid info, package version, configuration as seen by the
service, etc.

Oslo Guru Meditation Reports provide an easy way to add support for collecting
the live state info from any service. Report generation is triggered by sending
a special(USR1) signal to a service. Reports are generated on stderr, and can
be piped into system log based on need.

Nova has supported Oslo Guru Meditation Reports.

Proposed change
===============

First, a new oslo-incubator module (reports.*) should be synchronized into
Ceilometer tree. Then, each service process needs to initialize the error
report system before the process executes launch method in the main function.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

This feature could expose service internals to someone who is able to send the
needed signal to a service. That said, we can probably assume that the
operator is already authorized to achieve a lot more than just having an access
to stack traces and configuration used. Of course, if deployers are afraid of
the information leak for some reason, they could also make sure their stderr
output is channeled into safe place.

Pipeline impact
---------------

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

The feature does not require any additional resources until it's triggered by
the user and reports are not expected to be generated too often, so it has
little impact on performance and scalability.

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
  zhangtralon

Work Items
----------

* sync reports.* module from oslo-incubator
* adopt it in all ceilometer services under ceilometer/cmd/
* add some developer docs on how to use this feature

Future lifecycle
================

None

Dependencies
============
Currently, the reports.* module from oslo-incubator is graduating into
oslo.reports in Library. In case it's graduated into oslo.reports before
Ceilometer switches to it, we will not need to synchronize the reports.* module.

Testing
=======

None

Documentation Impact
====================

Documentation should be extended to describe the new feature.

References
==========

[1] oslo-incubator module: http://git.openstack.org/cgit/openstack/oslo-incubator/tree/openstack/common/report

[2] nova guru meditation reports: https://blueprints.launchpad.net/nova/+spec/guru-meditation-report

[3] blog about nova guru reports: https://www.berrange.com/posts/2015/02/19/nova-and-its-use-of-olso-incubator-guru-meditation-reports/

[4] oslo.reports repo: https://github.com/directxman12/oslo.reports
