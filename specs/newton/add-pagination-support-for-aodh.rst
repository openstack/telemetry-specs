..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===============================
Add pagination support for Aodh
===============================

https://blueprints.launchpad.net/aodh/+spec/support-pagination

This BP proposed to add pagination support for Aodh. User can use this
feature to set limit, marker and sort when they query their alarm and
alarm history.

Problem description
===================

Currently when list alarm and alarm history, all the existed alarm and
history will return at one time, and it is really not user friendly for
users.

Proposed change
===============

Allow Aodh user to use  the general pagination mechanism with the help of
`limit`, `marker`, `sort_key`, `sort_dir` optional parameters to list alarm
and alarm history.

* **sort_key**: Key used to determine sort order

* **sort_dir**: Direction for with the associated sort key ("asc" or "desc")

* **marker**: The last alarm ID of the previous page. Displays list of
  alarms after "marker".

* **limit**: Maximum number of alarms to display. If limit == -1,
  all alarms will be displayed.

Alternatives
------------

Keep the current anti-friendly situation.

Data model impact
-----------------

None

REST API impact
---------------

New optional parameters `limit`, `marker`, `sort_key`, `sort_dir`
will be added to GET /v2/alarms and GET /v2/query/alarms/history

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

Primary assignee:
  Zhenyu Zheng

Other contributors:
  liusheng

Work Items
----------

* Add pagination support for alarm and alarm history;
* Add the related support in alarm client.


Future lifecycle
================

None

Dependencies
============

None

Testing
=======

Related tests will be added.

Documentation Impact
====================

References about how to use pagination will be added.

References
==========
