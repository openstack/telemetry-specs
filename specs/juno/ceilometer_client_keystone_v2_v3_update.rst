..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

============================================
Ceilometer Python Client Keystone V3 Upgrade
============================================

Keystone declared V2 deprecated in K cycle with the intent of maintaining
V2 for another two cycles after that. All the services clients are going
to add support to V3 in order to smooth this transition.

Problem description
===================

Keystone V3 API has introduced the following major changes:

* Authentication: Custom auth method can be developed as plug-ins, this enables
  support for: OAuth 1.0 and SAML based federation

* Authorization: in V2 API there are only two levels "admin" or none. V3 API
  enables control of who can call each method if the policy file is correctly
  specified.

* Richer set of APIs. Vendor specific extensions are not supported directly
  anymore and there is a set of optional extensions supported by Keystone.

Proposed change
===============

Since Ceilometer Client imports keystoneclient, the main changes to client.py
are to selectively import the correct keystoneclient library version and pass a
number of new options. In shell.py the new options are added to the parser.

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

None.

Other end user impact
---------------------

There will be new commands to be used with the ceilometer client to scope the
request to Domain/Project and support different auth methods.
Alarms are using the ceilometer client and there could be a potential impact.

Performance/Scalability Impacts
-------------------------------

None.


Other deployer impact
---------------------

None.

Developer impact
----------------

None.


Implementation
==============

Assignee(s)
-----------

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  fabgia

Other contributors:
  robsparker

Ongoing maintainer:
  fabgia

Work Items
----------

* Add new parameters to the client and cli.
* Call the V3 Keystone client.
* Validate the authorization and authentication via python-keystoneclient.


Future lifecycle
================

None.


Dependencies
============

* Keystone client: python-keystoneclient.


Testing
=======

Unit tests will be added to the Ceilometer client to support the new parameter
submission as well as validation of the authorization and authentication with
Keystone client.


Documentation Impact
====================

Documentation changes specific to new parameters added to the client.


References
==========

Related blueprints:
Ceilometer Client
https://blueprints.launchpad.net/python-ceilometerclient/+spec/support-keystone-v3-api
Keystone
https://blueprints.launchpad.net/keystone/+spec/document-v2-to-v3-transition


