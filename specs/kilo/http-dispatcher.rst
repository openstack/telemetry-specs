..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Http Dispatcher Spec
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/http-dispatcher


Ceilometer dispatcher framework allows various dispatchers to be developed and
configured to make ceilometer meter destination to be fully configurable.
Currently there are two dispatchers available for Ceilometer, these two
dispatchers are used to save Ceilometer meters either to database or log like
files. In some applications, meters or filtered meters are wanted to be sent
to other third party systems especially using http protocol. This spec
provides the specification for a Ceilometer dispatcher using http protocol.

Problem description
===================

The usecase is as follows:
Enterprise Monitoring system QRadar needs to capture activities happening in
OpenStack keystone such as user creation, deletion, log in etc. Keystone posts
these activities onto OpenStack message queues, Ceilometer collector
ultimately collects these messages and turn them into meters, so these meters
will be sent over to QRadar via the http post request, once the meters get
posted via http post request to QRadar, QRadar will analyze the meters and
take actions accordingly.

Proposed change
===============

A new dispatcher should be added in Ceilometer/dispatcher package, the
dispatcher will be named HttpDispatcher by inheriting the dispatcher.Base
class and implement the record_metering_data method along some dispatcher
specific configuration parameters. The record_metering_data method will
wrap meters in JSON format and post the data by using HTTP POST method to
a target. The http request content type should be application/json. If the
post target is not defined, then an error should be logged. The post request
should use a configurable time to determine if the connection should be timed
out. The implementation file should be named http.py so that the module naming
pattern consistency is kept. If the meter data contains auditing information
(CADF), then the auditing message will be sent rather than the meter itself
since most of the information in the meter will be duplicate of the auditing
information.

Alternatives
------------

* develop an agent and read data from database, but the impact will be data
  get saved in the database, but the goal is to have the meter info saved in
  the system which integrates with Ceilometer.

* develop an agent read data from queue, but there will be a lot more code
  needed to do that.

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

Tong Li (IBM)

Primary assignee:
  litong01

Other contributors:
  None

Ongoing maintainer:
  Tong Li

Work Items
----------

* Http dispatcher
* Document on how to configure this dispatcher
* test cases

Future lifecycle
================

None

Dependencies
============

No new dependencies required. It only requires requests, oslo.config which have
been the common dependencies of this project.

Testing
=======

Tempest tests will be added in ceilometer/tests/dispatcher/test_http.py.
These test cases can be invoked at the gate or locally in a dev environment.

Documentation Impact
====================

file at /doc/source/install/manual.rst will be changed to reflect this new
dispatcher along other existing dispatchers. Since this is a new dispatcher,
there will not be changes to any existing components, only additional
information will be added to the doc on how to enable and configure this
dispatcher.


References
==========

None