..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

======================
Self-disabled Pollster
======================

https://blueprints.launchpad.net/ceilometer/+spec/self-disabled-pollster

Avoid loading one pollster if required environment is not ready. Remove
resources that cause continuous exception in polling time.


Problem description
===================

Currently pollsters are used in following way.

1. define via setup.cfg
2. load each pollster in initialization
3. pollster's get_samples get called in periodic way to collect metrics

This fixed process is not flexible enough to handle various situation, like
following cases:

1. One pollster depends on specific environment, e.g compute node pollsters
   need right hypervisor inspector, ipmi pollsters need ipmitool installed.
   Loading pollster unconditionally produces extra exceptions or dummy samples
   with minor performance drop. Check in deployment can not resolve all the
   issues, as environment changes dynamically. The perfect solution is that
   check in loading time and do not load the pollster if no required
   environment.
2. One pollster gets loaded and runs well, then some polling resource becomes
   unavailable, so the pollster fails to produce samples and probably throws
   exceptions with performance drop. We need detect such changes, then remove
   this resource from polling. Also need put warning in log, so operator can
   track and handle it.


Proposed change
===============

Provides infrastructure to get pollster specific response then take different
action.

For case 1, need provide a function in pollster constructor, to raise specific
exception if required hardware is missing. When loading extensions, add
on_load_failure_callback, which checks and suppresses the exception to avoid
loading this extension. Probably need one configuration list to indicate which
pollster can be skipped.

For case 2, we need pollsters throw 2 different exceptions in run time:

* transient failure: treat like existing ones
* permanent failure: Not poll that resources from this pollster any more

To achieve this, we have PollingTask to catch that permanent failure, then
block polling for that resources.


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

None

Pipeline impact
---------------

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

This BP improve performance due to avoid loading unnecessary pollster and
remove unavailable resources to eliminate continuous exceptions.

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
  edwin-zhai

Other contributors:
  lianhao-lu

Ongoing maintainer:
  edwin-zhai

Work Items
----------

* For loading time pollster

  * Add environment check function in pollster constructor
  * Add new on_load_failure_callback to avoid loading failed extension

* For running time pollster

  * Modify required pollster to throw permanent failure exception
  * Modify PollingTask to skip failure resources


Future lifecycle
================

Once this feature enabled, need test and bug fixing in next 2 releases to avoid
regression


Dependencies
============

None


Testing
=======

Need unit test


Documentation Impact
====================

None


References
==========

None

