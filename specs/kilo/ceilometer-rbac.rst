..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================================================
Ceilometer API RBAC - Granular Role-based Access Control
========================================================

Current access control for the API presents a clear lack of granularity. In
fact it is possible to have a "global" admin role or a simple "user" within the
project and nothing in between.

With upcoming Keystone v3 enhancements we can expand the granularity of access
control to allow non "global" admins, i.e. Domain Admins or Parent Project
admins.

We will accomplish this by using a decorator on the API functions. The
decorator will control access based on user/project roles and rules specified
in the policy json file.

acl.py could be deprecated since its only function is to determine if a user is
admin, and the decorator accomplishes this.

Example policy expansions:

Current policy.json only verifies user is admin:

.. code-block:: json

    {
        "context_is_admin": [["role:admin"]]
    }


New rules allow separation of access control by method and expanded roles. Also
compatible with Keystone v3 expanded functionality where domains are supported.

.. code-block:: json

    {
         "context_is_admin": [["role:admin"]],
         "admin_or_cloud_admin": [["rule:context_is_admin"],
                  ["rule:admin_and_matching_project_domain_id"]],
         "telemetry:alarm_delete": [["rule:admin_or_cloud_admin"]]
    }


Problem description
===================

Ceilometer API currently supports all or nothing authentication for API calls.

This limits the functionality of the reporting API as managers of more than one
project must either be given the admin role or re-scope each request to view
multiple projects.

Use cases where this limits functionality include:

* Resellers managing more than one project

* Domain administrators managing more than one project

* Support personnel who manage above cases

Proposed change
===============

We propose to solve the problem by moving access control from calls to the ACL
to applying a decorator to the API methods.  Each publicly accessible API
method would have a decorator pointing to a new RBAC module.  The RBAC module
decorator would use rules defined in policy.json to determine accessibility of
methods by a caller.

This would allow fine-grained, role-dependent, method-specific access control.

It would also align Ceilometer API wit the Keystone V3 Domain capabilities as
well as the hierarchical project proposal since both domain or project could
be used in rule creation.

Alternatives
------------

It would be possible to place an RBAC filter in front of the Pecan webserver.
This filter would implement RBAC through calls to Keystone to verify roles,
projects and domains.

While we believe this is a reasonable solution, it diverges from the way
Ceilometer API is currently implemented.  It would require insertion of an
access control filter between the middleware and Pecan.  It would also require
significant code changes to the current RBAC scheme which handles access
control within the API.

The proposed model will minimize code changes.  It would simply add decorator
statements to the external API methods, create an additional module, and add
configuration changes to policy.json

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

The new model would improve security because access control would become
centralized in the decorator module.  After ensuring each external method has a
default decorator call, access control would remain as admin or project-only
unless changed in the policy configuration file (policy.json).

The current model places calls to the ACL module in varying locations
throughout the API code.  For example, GET methods call _query_to_kwargs which
eventually calls the ACL, and PUT methods call the ACL directly.  This could
lead to confusion on how to handle access control in new methods or how to
change it for existing ones.

Security improvements include the ability to allow user/project combinations to
be granted roles other than the powerful admin role and still accomplish
meaningful activities.  Removal of the maximum-privilege admin role and
limiting access to the least possible set of operations is an improvement.

Possible security impacts involve the fact that this approach is more
role-dependent.  If complex rules are specified but the Keystone role granting
privileges are not tightly controlled, there are more opportunities to grant
unwarranted access to users.

In other words, with more roles and access schemes available there is more to
manage.

We believe this risk is mitigated by the ability to ship the basic code with
only the current context_is_admin rule enabled.  Such configuration would not
allow additional Keystone roles to grant new privileges unless the system
operators explicitly added new rules to the policy file.

Pipeline impact
---------------

None

Other end user impact
---------------------

This will have no direct impact on on python-ceilometerclient as roles and
their associated rules would be established in keystone and interpreted by
Ceilometer API. Nevertheless, the python-ceilometerclient will benefit from the
increase security provided by the new policy support. For instance, collector
agent (or any other ceilometer service) can have a special role associated
with it disallowing other services (with admin status) to post data in the
database.

If end users were to take advantage of the expanded RBAC capabilities, there
would be end user impacts in securing appropriate user and project roles to
match those defined in the policy file.

Performance/Scalability Impacts
-------------------------------

None

Other deployer impact
---------------------

By default there will be no impact during deployment.  The change will ship
with a configuration that preserves the current access control behavior.  The
operator can simply leave everything as is and expect the system to behave as
it did before this change.

There will be an option to define and use different policy files if the
operator wants to take advantage of the expanded access control capabilities
offered by this fix.  This fix will also offer compatibility with Keystone v3
features (again, optional not out of the box).

Deployment options:

* Optional policy configuration changes to enable expanded access control

* Change must be explicitly enabled to allow expanded access control, otherwise
  it defaults to current access control behavior.

* No changes in the package deployment are required.

* Existing policy definition files will continue to work as they currently do
  without any special changes.

Developer impact
----------------

Developers adding new Ceilometer API endpoints will need to add the appropriate
access control mechanisms to exposed API methods.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  eap-x, fabgia

Other contributors:
  eap-x, srinivas-sakhamuri

Ongoing maintainer:
  fabgia, srinivas-sakhamuri

Work Items
----------

Work items:

* Implement RBAC validation module

* Apply decorators to all external Ceilometer API methods (such as v2/meters,
  etc.)

* Deprecate acl.py usage

* Make policy.json rule additions if desired (optional)

Future lifecycle
================

The HP Ceilometer development team is actively interested in improving and
maintaining API access control.  We foresee a need by commercial cloud
providers to configure access control for reselling and for private cloud
management.

We would like to be actively engaged in cotinued development and maintenance of
this feature for many cycles.

Dependencies
============

* Keystone v3 adoption

* No new libraries or programs required

* No external dependencies

Testing
=======

Existing API tests will ensure current access control functionality is
preserved.  New unit tests will cover expanded functionality.

Documentation Impact
====================

Documentation around enabling the expanded RBAC features will be required.


References
==========

Keystone V3 Policy:
https://github.com/openstack/keystone/blob/master/etc/policy.v3cloudsample.json
