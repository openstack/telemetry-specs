..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Declarative snmp metrics
==========================================

https://blueprints.launchpad.net/ceilometer/+spec/declarative-snmp-metrics

This blueprint wants to add a mechanism so we can add new snmp metrics in a
declarative way instead of writing new source code for new pollsters.


Problem description
===================

Currently, we use a generic snmp hardware inspector and a python dict which
defines the mapping between snmp oid/operations and the metrics value the
caller wants(See the following link for the description of that mapping).

https://github.com/openstack/ceilometer/blob/stable/kilo/ceilometer/hardware/inspector/snmp.py#L112

On top of that generic snmp inspector, we still have separate hardware
pollsters doing things like setting multiple fields in the Sample object.
It's ok to add a dozen of new metrics. But if want to add hundreds of new snmp
metrics, we need to do it in a declarative way, instead of writing new
pollster code for every new snmp metrics added.


Proposed change
===============

While keeping the current snmp related hardware.* pollsters intact for
backward compatibility, we'll add a new generic snmp pollster, which
can return different types of snmp related metrics based on user request.

This generic snmp pollster will load a definition from a yaml definition
file which defines the parameters used to call into the generic snmp
inspector, and also other fields in the final Sample object the pollster
returns, like name/unit/type. However, the resource_id/user_id/project_id
will still be read from the resources returned by the discover just
like what we do today.


The yaml definition would be something like the followings::

  ---
  - counter_name: hardware.cpu.load.15min
    unit: process
    type: gauge
    snmp_inspector:
      matching_type:  type_exact,
      oid:  "1.3.6.1.4.1.2021.10.1.3.1",
      type: float
      metadata: { }
      post_op:

  - counter_name: hardware.network.incoming.bytes
    unit: B
    type: cumulative
    snmp_inspector:
      matching_type: type_prefix
      metric_oid:  "1.3.6.1.2.1.2.2.1.10"
      metadata: {
              name: {
                       oid: "1.3.6.1.2.1.2.2.1.2",
                       type: "str",
              },
              speed: {
                       oid: "1.3.6.1.2.1.2.2.1.5",
                       type: "lambda x: int(x) / 8",
              },
        }
      post_op: "_post_op_net"

Please see the following link for the detailed explanation of the
snmp_inspector dict in the definition:

https://github.com/openstack/ceilometer/blob/stable/kilo/ceilometer/hardware/inspector/snmp.py#L112

When adding new snmp metrics, we only needs to add the new definitions into
the yaml definition. No need to writing new code.

Because the new generic snmp pollster could return multiple different types
of meters, it would bring the following changes to the current polling
agent implementation:

1. Currently, because of the mapping of meter pollster plugin one meter,
when deciding whether a pollster is needed for a given pipeline, we
use the pollster plugin's entry point name. This will be changed. We'll
add a new helper function which could return all the meters that pollster
could return, and use that helper function in the above process to decide
whether a pollster is needed for a given pipeline.

2. When calling the pollster plugin to get the samples, we'll pass
the pipeline definition as a new parameter to get_samples() call. So
instead of return all the samples, the pollster can only return what
the pipeline source asks for.

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

No changes for the admin in how to configure the pipeline yaml file.

Other end user impact
---------------------

None

Performance/Scalability Impacts
-------------------------------

None

Other deployer impact
---------------------

Need to deploy a new yaml definition file on the system running polling agent.

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------


Primary assignee:
  <lianhao-lu>

Ongoing maintainer:
  <lianhao-lu>

Work Items
----------

* enhance the pipeline to support new source definition
* implement the generic snmp pollster and its yaml definition file


Future lifecycle
================

To be maintained by <lianhao-lu>


Dependencies
============

N/A


Testing
=======

The functionality test should be covered in the 3rd party CI of
Intel Hradware-Meters CI.


Documentation Impact
====================

Need to update the openstack-manual admin manual about the change to the pipeline
definition.


References
==========

N/A
