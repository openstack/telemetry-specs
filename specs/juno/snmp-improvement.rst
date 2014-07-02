..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Improve the snmp hardware inspector
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/snmp-improvement

We could make some improvement to the existing hardware snmp inspector for the
following objectives:

* Minimize the coding effort in snmp inspector when adding new hardware metrics

* Reduce the network traffic between the inspector and the remote snmpd agent


Problem description
===================

Currently, to add new hardware metrics, the developer needs to do the
following:

1. Add new inspect method in ceilometer.hardware.inspector.base.Inspector class.

2. Add new returned data type definition of the above new inspect method.

3. Implement the new inspect method in snmp inspetor.

4. Add new hardware pollsters and register it in setup.cfg.

This could result in quite a lot of coding, considering the tons of metrics
available through snmp which could be added.

Besides, since most of the existing snmpd agents support version 2c, we should
leverage the snmp GetBulkRequest introduced in version 2c to minimize the
network traffic between the snmp inspector and snmpd agent. If the snmpd agent
doesn't support version 2c and above, we should fall back to GetNextRequest.


Proposed change
===============

Because for each hardware metric, it could be mapped to one or more snmp OIDs.
Using these OIDs, the snmp inspector could get the sample value as well as
the sample's metadata.

So here we propose the following changes:

In ceilometer.hardware.inspector.base.Inspector:

We'll replace the current inspect_xxx() methods with a generic inspect
method::

    def inspect_generic(self, host, identifier, cache):
        """ return an iterator of (value, metadata, extra)
        param host: the target host
        param identifier: the identifier of the metric
        param cache: cache passed from the pollster
        return value: sample value
        return metadata: dict to construct sample's metadata
        return extra: dict of extra information to help pollster
                      to construct the sample
        """

In ceilometer.hardware.inspector.snmp:

Here we define a mapping dict like the following::

    MAPPING = {
      identifier: {
        'matching_type': EXACT or PREFIX,
        'metric_oid': oid of the sample value,
        'metadata': {
           metadata_name1: [oid1, value_converter],
           metadata_name2: [oid2, value_converter],
        },
        'post_op': special func to modify the return data,
      },
    }

The identifier parameter passed in to the method inspect_generic() serves as
the key to the above mapping.

The inspect_generic() method in snmp inspector would do the following things::

    1. map = MAPPING[identifier]
       value = None
       metadata = {}
       extra = {}

    2. Collect all the oids in the map, including the metric_oid as
       map['metric_oid'] and oids in metadata map['metadata'].

    3. Send out GetRequest or GetBulkRequest(based on the matching_type
       respectively) to get those oid values which are not in the cache,
       and save the response as key-value pairs like oid:value in the cache.

    4. Based on the matching_type, construct value and metadata using the
       oid:value key-value pairs in the cache. The metadata is constructed
       as a dictionary like:
       {
         metadata_name1: value_converter(select_data(cache, oid1, index)),
         metadata_name2: value_converter(select_data(cache, oid2, index)),
       }

    5. call post_op(host, map, value, metadata, extra)
       The post_op is assumed to be implemented by new metric developer. It
       could be used to add additional special metadata(e.g. ip address), or
       it could be used to add information into extra dict to be returned
       to construct the pollster how to build final sample, e.g.
           extra.update('project_id': xy, 'user_id': zw)


Based on the different matching_type, the way to construct the returned
(value, metadata) in the above step 5 is a little bit different:

If matching_type is EXACT, we use oid as the exact key into the cache to find
the corresponding value, i.e. cache[oid].

If matching_type is PREFIX, all the keys in the cache starting with oid as the
prefix matches, and remaining part in the key will be treat like the index.
The index could be used to find corresponding metadata value. For example,
suppose we have the following mapping::

    MAPPING = {
      'disk.size.total': {
        'matching_type': PREFIX,
        'metric_oid': "1.3.6.1.4.1.2021.9.1.6",
        'metadata': {
           'device': ["1.3.6.1.4.1.2021.9.1.3", str],
           'path': ["1.3.6.1.4.1.2021.9.1.2", str],
        },
        'post_op': None,
      },
    And in the cache, we have something like the following:
    {
      '1.3.6.1.4.1.2021.9.1.6.1': 19222656,
      '1.3.6.1.4.1.2021.9.1.3.1': "/dev/sda2",
      '1.3.6.1.4.1.2021.9.1.2.1': "/"
      '1.3.6.1.4.1.2021.9.1.6.2': 808112,
      '1.3.6.1.4.1.2021.9.1.3.2': "tmpfs",
      '1.3.6.1.4.1.2021.9.1.2.2': "/run",
    }
    So here we'll return 2 instances of (value, metadata, extra):
     (19222656, {'device': "/dev/sda2", 'path': "/"}, None)
     (808112, {'device': "tmpfs", 'path': "/run"}, None)


In ceilometer.hardware.pollsters:

We need to change accordingly to the new generic inspector interface.


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

The existing model is to use GetNextRequest first to get the index and then
use that index to GetRequest mutiple OIDs. We replace it with GetBulkRequest.
This could significantly reduce the snmp packet number exchanged between the
snmp inspector and snmpd agent. Thus we could improve the performance.


Other deployer impact
---------------------

In order to make snmp inspector working, a snmp agent 'snmpd' needs to be
running on the machine, listening on the network interface which is accessible
to the snmp inspector and the ceilometer agent. However, this is not a new
deploy requirement because we already require a snmpd agent deployed.

Developer impact
----------------

The interface change in ceilometer.hardware.inspector.base.Inspector might
requires other hardware inspector implementations to be changed too. But the
only known implementation now is the snmp inspector, so there should be no
other significant impact.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <lianhao-lu>

Other contributors:
  <None>

Ongoing maintainer:
  <lianhao-lu>

Work Items
----------

* adding new inspect_generic interface and the snmp implementation
* change the pollster to new interface and remove old inspector interface


Future lifecycle
================

None


Dependencies
============

* https://review.openstack.org/#/c/92370


Testing
=======

None


Documentation Impact
====================

None


References
==========


* Juno summit session log

https://etherpad.openstack.org/p/ceilometer-snmp-inspector

* pysnmp library

http://pysnmp.sourceforge.net
