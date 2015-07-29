..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Events RBAC via Policy
==========================================

OpenStack needs to support granular, customizable Role-Based Access Control
(RBAC) for ceilometer events. Policy should be dependent on policy.json rather
than simply hard-coded.

https://blueprints.launchpad.net/ceilometer/+spec/events-rbac

Problem description
===================

The only RBAC validation for ceilometer’s /events REST API that is in place
today is hard-coded logic to restrict requests to admins. This is not granular
enough. In a multi-tenant environment, there must be provision to restrict data
from events to the scope of the token given on the request (e.g. an admin from
one project should not be able to view the events of another project). There
should also be a way to allow non-admins access to some events (e.g. events
for their resources). This issue was originally raised as a bug [1].


Proposed change
===============

This blueprint proposes the following changes in the behavior of ceilometer’s
/events REST API:

Add the following rules to ceilometer’s default policy.json:

“telemetry:events:index”: “role:admin”
“telemetry:events:show”: “role:admin”

Modify the code to uses those checks to determine who is allowed to request a
list of events (index) or a specific event (show). This pattern is in line with
other OpenStack projects.

Further modify the code to, if/when those checks pass, filter which events will be returned based on the scope of the token. If the token is for an admin user
(determined by checking the special context_is_admin rule from policy.json),
only return events corresponding to the project to which the supplied
X-Auth-Token is scoped. This is also consistent with other projects. If the
token is for a non-admin user, further filter the results to only return
events corresponding to the user to which the supplied X-Auth-Token is scoped,
as well as the project.

The events REST API will be processed only if the user passes a project-scoped
token. This is required to filter the events based on the project. If the user
passes an unscoped or domain-scoped token, a 403 error will be thrown.

There may be events for which no project/user information is recorded. Most if
not all of these should have project/user information, so event-generating code
will need to be modified to include that when creating future events. Updating
existing events within the database is outside the scope of this spec. Until
all events have project/user information, events which lack this data will be
returned along with events corresponding to the token’s project, if the token's
user is an admin on that project.

For the first implementation, common name-reserved traits will be used to store
project/user information.

Alternatives
------------

We could use policy.json checks to filter results as well as determine whether
the request is allowed. This does not fit with the role and purpose of
policy.json, and may also have significant performance implications.

Storing project/user information in base attributes rather than traits was
considered. This may be better long-term, but there has been some debate on the
appropriateness of using base attributes if there will be some events that are
not scoped to a project/user. Implementing this using traits first will help us
determine whether that is a valid case. A switch to base attributes could come
later.

Rather than return both project/user-scoped events and unscoped events in a
single response, we could require a separate API call for unscoped events
if/when that is what the user desires. E.g. /broadcasts instead of /events. As
it is unclear whether we will end up with any events that do not have
project/user information, considering this now would be premature.

We could create another policy check to identify whether a non-admin role
should be considered an admin for the purpose of the /events API (and thus able
query events for the entire project rather than just for themselves). But since
it is expected that events will be split out of ceilometer like meters
(gnocchi) and alarms (aodh), that would seem to be a short-term solution.

We could support domain-scoped tokens to allow domain admins to query events
for the entire domain with a single request. This may be needed in the future,
but does not appear to be something that must be done as part of this effort.
And there could be benefits to waiting for the keystone reseller spec [2]
implementation.

Data model impact
-----------------

None, as long as we stick using traits.

REST API impact
---------------

Whereas before this change an admin of any project saw events for EVERY project
in the response to a GET /events request, now GET /events will filter out
events belonging to projects other than the one in the scope of the request's
auth token. This closes a security hole that allowed admins of one project to
gain information about another project in which they have no role.

Security impact
---------------

This will greatly enhance security by default. It will also provide a mechanism
for operators to customize policy related to events, so operators taking
advantage of this capability will need to consider the potential security
impacts of their configuration changes.

Pipeline impact
---------------

None

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

Filtering on project/user may have a slight negative performance impact, though
this should be offset by the likely much more substantial performance
improvement of returning less data to the caller.

Allowing events that do not have project/user information via the same API
queries will mean making two internal calls (one to get events scoped to the
project/user and another to get events not scoped to a project/user) and then
merging them. This may have a slight negative performance impact.

Other deployer impact
---------------------

Updating from a previous release will need to include moving to a new version
of policy.json including the new checks, else those would resolve to the
default which is to allow for anyone.

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  edmondsw

Other contributors:
  dikonoor

Work Items
----------

* Check context_is_admin to determine appropriate response filtering
* Check policy.json to determine whether the request is allowed
* Add project/user to events that are currently lacking those details


Future lifecycle
================

This is essentially a large bug fix, not a feature. As such, all members of the
Telemetry program will be expected to keep this from breaking again.

Several potential future enhancements are discussed under the Alternatives
section. It is expected that those would require a separate spec if someone
wants to pick up any of them in the future.


Dependencies
============

None


Testing
=======

Unit tests should be sufficient.


Documentation Impact
====================

Developer documentation [3] will be updated to add user_id to the list of
default traits, and to explain that non-admins will only be able to view events
with their user_id, while admins will only be able to view events with their
tenant_id plus events not associated with a project (aka tenant).


References
==========

[1] https://bugs.launchpad.net/ceilometer/+bug/1461767
[2] https://github.com/openstack/keystone-specs/blob/master/specs/liberty/reseller.rst
[3] http://docs.openstack.org/developer/ceilometer/events.html

