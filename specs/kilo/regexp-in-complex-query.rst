..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================
Regexp in complex query
=======================

https://blueprints.launchpad.net/ceilometer/+spec/regexp-in-complex-query

Problem description
===================

Currently some instance or network related service have extra identifier in
resource_id. Example is disk.device.* metrics. They have resource_id
with format - <instance_id>-<disk-id>. We can't make query for getting
all samples from these meters with specified instance_id because disks have
different ids. This behavior not easy for end users and adds troubles for
someone who wants know more information about cluster work.

Proposed change
===============

To support complex queries described above we should add new regex
operator with corresponding "=~" designation.

Implementation depends on backend:

1) MongoDB and DB2. These dbs has $regex query support.
It provides regular expression capabilities for pattern matching strings
in queries. MongoDB and DB2 use Perl compatible regular expressions
(i.e. “PCRE” ) version 8.30 with UTF-8 support. For correct work we map
"=~" from filter query to "$regex" in pymongo request.

2) SQLAlchemy. This backend supports "regexp" MySQL operator, also it's mapped

3) HBase supports regexp queries with RegexStringComparator.

Alternatives
------------

Add extra information to resource metadata

Data model impact
-----------------

None.

REST API impact
---------------

Add new simple operator "=~" in complex queries.

Security impact
---------------

None.

Pipeline impact
---------------

None

Other end user impact
---------------------

You may make regex request in complex query.

Performance/Scalability Impacts
-------------------------------

Will affect performance for large regular expressions,
api service may be affected.

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
  ityaptin

Ongoing maintainer:
  enovokshonova

Work Items
----------

- add "=~" to simple operators list in controller
- add regexp operator processing to backends

Future lifecycle
================

None

Dependencies
============

None.

Testing
=======

- extend storage scenarios testing to include regexp operator

Documentation Impact
====================

- update complex query notes to include new operator

References
==========

https://blueprints.launchpad.net/ceilometer/+spec/regexp-in-complex-query
