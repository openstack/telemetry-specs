..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Enable LBaaS V2 for Ceilometer
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/lbaas-v2-enablement

The goal of this blueprint is to support LBaaS v2 to poll the LoadBalancer
data in ceilometer.

Problem description
===================

According to the Neutron api documentations[4], the LBaaS v1 has been
deprecated in the release Mitaka.

Based on the neutron LBaaS v2 documentation[1] and LBaaS v2 blueprints,
LBaaS v2 apis are supported. LBaaS v1 and LBaaS v2 can not be run in the
same time. The users can only choose one version of LBaaS to run in their
environment.

According to the ceilometer admin guide[3], Ceilometer support to collect
the data of LBaaS from Juno release.

When the users choose to use LBaaS v2 in their environment, they will
meet the problem to use the ceilometer to collect LBaaS meters for the
LBaaS v2 api is different from LBaaS v1.

For LBaaS v1 has been deprecated from the release Mitaka, Ceilometer will
lost the function to collect the LBaaS meters if Ceilometer doesn't support
the LBaaS v2.

The design of LBaaS v1 and LBaaS v2 is different. LBaaS v2 introduces the
meter 'loadbalancer' and it contains 'vip', so the meter 'vip' is not
important to the LBaaS v2 users. And LBaaS v2 has not supplied with the
APIs to achieve the data of the vips directly.

Meters are supported by LBaaS v1 and not supported by LBaaS v2.

Name                                  Type                Unit
network.services.lb.vip               Gauge               vip
network.services.lb.vip.create        Delta               vip
network.services.lb.vip.update        Delta               vip

All the LBaaS APIs have changed in the version 2. We need to use the new APIs
to collect the data.

The difference of APIs between LBaaS v1 and LBaaS v2.

Meter_Name                                v1 API
network.services.lb.pool                  /v2.0/lb/pools
network.services.lb.member                /v2.0/lb/members
network.services.lb.health_monitor        /v2.0/lb/health_monitors
network.services.lb.total.connections     /v2.0/lb/pools/%s/stats
network.services.lb.active.connections    /v2.0/lb/pools/%s/stats
network.services.lb.incoming.bytes        /v2.0/lb/pools/%s/stats
network.services.lb.outgoing.bytes        /v2.0/lb/pools/%s/stats

Meter_Name                                v2 API
network.services.lb.pool                  /v2.0/lbaas/pools
network.services.lb.member                /v2.0/lbaas/pool/%s/members
network.services.lb.health_monitor        /v2.0/lbaas/pools
network.services.lb.total.connections     /v2.0/lbaas/loadbalancers/%s/stats
network.services.lb.active.connections    /v2.0/lbaas/loadbalancers/%s/stats
network.services.lb.incoming.bytes        /v2.0/lbaas/loadbalancers/%s/stats
network.services.lb.outgoing.bytes        /v2.0/lbaas/loadbalancers/%s/stats

The difference of neutron client methods we use to collect the LBaaS between
LBaaS v1 and LBaaS v2.

Meter_Name                                v1 method
network.services.lb.pool                  list_pools
network.services.lb.member                list_members
network.services.lb.health_monitor        list_health_monitors
network.services.lb.total.connections     retrieve_pool_stats
network.services.lb.active.connections    retrieve_pool_stats
network.services.lb.incoming.bytes        retrieve_pool_stats
network.services.lb.outgoing.bytes        retrieve_pool_stats

Meter_Name                                v2 method
network.services.lb.pool                  list_lbaas_pools
network.services.lb.member                list_lbaas_members
network.services.lb.health_monitor        list_lbaas_healthmonitors
network.services.lb.total.connections     retrieve_loadbalancer_stats
network.services.lb.active.connections    retrieve_loadbalancer_stats
network.services.lb.incoming.bytes        retrieve_loadbalancer_stats
network.services.lb.outgoing.bytes        retrieve_loadbalancer_stats

Proposed change
===============

In order that the user can switch the LBaaS api version in current situation,
A new entry will be added in the configuration file to specify the api
version of LBaaS.

When the user chooses to use the LBaaS api v1, the meters supported by LBaaS
v1 will be collected. But LBaaS api v1 has been deprecated in the release
Mitaka, we should suggest the user should not choose to use LBaaS api v1.

When the user chooses to use the LBaaS api v2, the meters supported by LBaaS
v2 will be collected. But some meters will not be supported by LBaaS v2,
including 'network.services.lb.vip', 'network.services.lb.vip.create',
'network.services.lb.vip.update'. we will add some error messages in the log
files to identify these meters can not be supported by LbaaS v2. And The
meters are supported both by LBaaS v1 and LBaaS v2, we will switch to use
the new neutron client methods to achieve the data of them. In the neutorn
client[5], the new LBaaS v2 apis have characters '*lbaas*'.

These change will not change the data model. It just switches to use the
LBaaS v2 api in the client.

New meters are introduced and supported by LBaaS v2.

* Metric Definitions:

Name                                       Type      Unit           Origin
network.services.lb.loadbalancer           g         loadbalancer   p
network.services.lb.listener               g         listener       p
network.services.lb.loadbalancer.create    d         loadbalancer   n
network.services.lb.loadbalancer.update    d         loadbalancer   n
network.services.lb.listener.create        d         listener       n
network.services.lb.listener.update        d         listener       n

Note:
loadbalancer: existence of a loadbalancer
listener    : existence of a listener
g           : gauge
d           : delta
p           : pollster
n           : notification

* Metadata:

We extract the data from the response of calling neutron client to constitute
the metadata of sample. The fields of metadata we use are fixed in the code.
After checking the response of LBaaS v2 apis, most of the fields are still in
the response. Only the field 'status_description' is not in the response. But
the relationship between field 'status' and 'status_description' are list in
the documentation[5], we can use the valute of field 'status' to set the value
of the field 'status_description'.

Basd on the above reason, the structure of metadata has not changed. We will
not meet the problem when both the data collected by LBaaS v1 and LBaaS v2
are in the same database.

Alternatives
------------

Option one:
Add the limitation in the admin guide[3], let the users to know the ceilometer
can not support the LBaaS v2. But the LBaaS v2 is more powerful and more safe
than v1, more and more users will choose to use LBaaS v2 in the future. And the
LBaaS v1 is deprecated in Mitaka release. So this limitation is not satisfied.

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
jizilian

Primary assignee:
jizilian

Other contributors:

Ongoing maintainer:

Work Items
----------
a. Add the new entry in the configration file to specify the version of
LBaaS
b. Use the LBaaS v2 api in ceilometer neutron client when the user decide
to use the LBaaS v2
c. Add the new UT test cases

Future lifecycle
================
Ongoing maintenance will be handled by the Ceilometer team, myself
included.

Dependencies
============
None

Testing
=======
Unit and integration Tests will be added to cover the necessary metrics
and validate samples generated.

Documentation Impact
====================
Need to update the doc about LBaaS v2 support.

References
==========
[1]https://specs.openstack.org/openstack/neutron-specs/specs/kilo/lbaas-api-and-objmodel-improvement.html
[2]https://blueprints.launchpad.net/neutron/+spec/lbaas-api-and-objmodel-improvement
[3]http://docs.openstack.org/admin-guide-cloud/telemetry-measurements.html
[4]http://developer.openstack.org/api-ref-networking-v2-ext.html
[5]https://github.com/openstack/python-neutronclient/blob/master/neutronclient/v2_0/client.py
