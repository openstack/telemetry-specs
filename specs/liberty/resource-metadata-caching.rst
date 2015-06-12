..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================
Resource Meta-data Caching
==========================

This was discussed at the YVR summit as follow up to issues raised during the
Operators feedback session.


Problem description
===================

Ceilometer polls regularly via the compute-agent to get resource meta-data'
for all instances. This polling adds load to Nova, Neutron, and Keystone
APIs, which at scale causes a significant performance impact. This meta-data
does not change very often, and the load is likewise often unnecessary.


Proposed change
===============

Lets take advantage of the changes-since parameter in the nova api call.
We'll start by creating an in-memory cache to store the resource meta-data
on the initial polling and the time of that polling. Then the compute agent
will use the changes-since parameter in all subsequent pollings. This will
reduce the amount of data queried and returned by the Nova API. The results
of this call will update the cache, and the cache will be used to return
the data. Also, we should populate the instance flavor and image data in
the cache as well.

Alternatives
------------

Do nothing

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

Should provide greater overall scalability due to the reduction in API load
and reduced traffic flows.

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
  jasonamyers

Other contributors:
  None

Ongoing maintainer:
  jasonamyers

Work Items
----------

 - Implement an in-memory cache in the compute agent.
 - Change compute-agent to populate an in-memory cache.
 - Change compute-agent to use the changes-since parameter in the Nova API call .
 - Change compute-agent to record the polling timestamp in the cache.
 - Change compute-agent to cache the flavor and image metadata as well.


Future lifecycle
================

None

Dependencies
============

None


Testing
=======

We will need to add tests for meta-data cache flag, API changed-since
queries, compute-agent cache control, and image/flavor caching.


Documentation Impact
====================

We will need to document the changes to the polling process, the new meta-data
cache config option, and create new documentation that describes the caching
work-flow.


References
==========

https://etherpad.openstack.org/p/YVR-ops-ceilometer
