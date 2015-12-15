..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.
 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Only support sqlalchemy in Aodh
===============================

https://blueprints.launchpad.net/ceilometer/+spec/only-support-sqlalchemy-in-aodh

This change want to deprecate others storage backends (mongodb, hbase) except
sql and remove them in future O* cycle, because sql is totally competent for
alarm data and we don't need to maintain multiple storage backend.

Problem description
===================

We support multiple storage drivers(habase, mongodb, sqlalchemy) in
Ceilometer, that is mainly for the recording/query of performance based massive
metric data. Aodh continue those drivers after alarming service splitting, but
Aodh only has alarm related data which unlikely be a large number and sql can
well meet the requirement.

Most of other projects in OpenStack only use sql as database. So we don't need
to keep the hbase and mongodb support in Aodh. The advantages of removing the
hbase and mongodb support are:

* make the code of Aodh more clean that reduce the maintenance works of storage
  drivers and related tests.

* focus on the functionality and avoid considering multiple drivers support if
  new features coming.

Proposed change
===============

Deprecate mongodb/Hbase support firstly and remove them in future cycle.

In current cycle:

* deprecate mongodb and Hbase storage support and log warning messages if
  config mongodb or Hbase as storage driver
* add alarm data migration tool for migrating data from mongodb/Hbase to sql

In future O* cycle:

* remove the mongodb storage implementation
* remove the hbase storage implementation
* remove the mongodb and hbase related tests
* remove the gate jobs based on mongodb and hbase

Alternatives
------------

None.

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

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

None

Other deployer impact
---------------------

mongodb or Hbase as storage driver will be not recommended in deployment.

Developer impact
----------------

Developers will be happy, they won't need to consider mongodb and Hbase storage
support and related tests.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  liusheng

Ongoing maintainer:
  None


Work Items
----------

In current cycle:

* send a email to do a user survey to get feedbacks about this change
* deprecate mongodb and Hbase storage support and log warning messages if
  config mongodb or Hbase as storage driver
* add alarm data migration tool for migrating data from mongodb/Hbase to sql

In future O* cycle:

* remove the mongodb storage implementation
* remove the hbase storage implementation
* remove the mongodb and hbase related tests
* remove the gate jobs based on mongodb and hbase
* update the related docs


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

Update the documentation about these changes.

References
==========

https://blueprints.launchpad.net/ceilometer/+spec/only-support-sqlalchemy-in-aodh
