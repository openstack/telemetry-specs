..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

====================================
Network Statistics From OpenDaylight
====================================

https://blueprints.launchpad.net/ceilometer/+spec/network-statistics-from-opendaylight

This feature proposes to add a ceilometer driver to collect network
statistics information using REST APIs exposed by network-statistics module
in OpenDaylight.

Problem description
===================

OpenDaylight collects and stores network statistics information and exposes
the same via its northbound REST APIs. An example of such statistics is
number of packets received or transmitted by a port which may be created
through neutron service.

A large portion of the network configuration received by OpenDaylight is
from OpenStack. It will be good to have the network statistics readable from
OpenDaylight via ceilometer APIs in a OpenStack-OpenDaylight deployment.

A driver already exists in ceilometer which collects statistics from
OpenDaylight. But that ODL REST APIs used by the driver have been
unfortunately removed or less maintained in current release of OpenDaylight
which is Carbon.

We propose a v2 version of the driver which will utilize a new statistics
module in OpenDaylight. To start with the v2 version of the driver will
currently support only a subset of counters that were supported by old
driver. Eventually this driver can be enhanced to support all counters.
This v2 OpenDaylight ceilometer driver will enable counters to be stored with
either MongoDB (or) with the Gnochhi collector thereby supporting both
backends for ceilometer.

Proposed change
===============

We propose to create a new driver in networking-odl in
networking_odl/ceilometer/network/statistics/opendaylight_v2/

The driver will communicate with OpenDaylight to fetch statistics through
REST APIs.

We wil support following meters::

* switch
* switch.ports
* switch.port
* switch.port.uptime
* switch.port.receive.drops
* switch.port.receive.errors
* switch.port.transmit.packets
* switch.port.receive.packets
* switch.port.transmit.bytes
* switch.port.receive.bytes
* port
* port.uptime
* port.receive.drops
* port.receive.errors
* port.transmit.packets
* port.receive.packets
* port.transmit.bytes
* port.receive.bytes
* switch.table.active.entries

New resource types will be added to /etc/ceilometer/gnocchi_resources.yaml::

  - resource_type: switch
    metrics:
      - 'switch'
      - 'switch.ports'
    attributes:
      controller: resource_metadata.controller

  - resource_type: switch_port
    metrics:
      - 'switch.port'
      - 'switch.port.uptime'
      - 'switch.port.receive.drops'
      - 'switch.port.receive.errors'
      - 'switch.port.transmit.packets'
      - 'switch.port.receive.packets'
      - 'switch.port.transmit.bytes'
      - 'switch.port.receive.bytes'
    attributes:
      switch: resource_metadata.switch
      port_number_on_switch: resource_metadata.port_number_on_switch
      neutron_port_id: resource_metadata.neutron_port_id
      controller: resource_metadata.controller

  - resource_type: port
    metrics:
      - 'port'
      - 'port.uptime'
      - 'port.receive.drops'
      - 'port.receive.errors'
      - 'port.transmit.packets'
      - 'port.receive.packets'
      - 'port.transmit.bytes'
      - 'port.receive.bytes'
    attributes:
      controller: resource_metadata.controller

  - resource_type: switch_table
    metrics:
      - 'switch.table.active.entries'
    attributes:
      controller: resource_metadata.controller
      switch: resource_metadata.switch

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

Users will have to configure OpenDaylight REST API endpoint information in
resources attribute in configuration file of ceilometer ie
/etc/ceilometer/polling.yaml. For eg::

    sources:
      - name: odl_source
        interval: 600
        resources:
          - opendaylight.v2://127.0.0.1:8080/controller/statistics?
              auth=basic&user=admin&password=admin&scheme=http
        meters:
          - "switch"
          - "switch.ports"
          - "switch.port"
          - "switch.port.uptime"
          - "switch.port.receive.drops"
          - "switch.port.receive.errors"
          - "switch.port.transmit.packets"
          - "switch.port.receive.packets"
          - "switch.port.transmit.bytes"
          - "switch.port.receive.bytes"
          - "port"
          - "port.uptime"
          - "port.receive.drops"
          - "port.receive.errors"
          - "port.transmit.packets"
          - "port.receive.packets"
          - "port.transmit.bytes"
          - "port.receive.bytes"
          - "switch.table.active.entries"

Other end user impact
---------------------

None.

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

Primary assignee:
  Deepthi V V

Ongoing maintainer:
  Deepthi V V

Work Items
----------

* Add new ceilometer driver for OpenDaylight in networking-odl

* Add support for new driver in devstack.

* Provide support for switch.* meters for gnocchi as backend in ceilometer.

* Unit tests for new driver

Future lifecycle
================

None.

Dependencies
============

Statistics module should be available in OpenDaylight Nitrogen release.

Testing
=======

Unit Tests will be added to test the new driver.

Documentation Impact
====================

The added metrics will need to be documented in the `measurements section`_.

.. _measurements section: http://docs.openstack.org/admin-guide-cloud/telemetry-measurements.html

References
==========

https://blueprints.launchpad.net/ceilometer/+spec/network-statistics-from-opendaylight

https://git.opendaylight.org/gerrit/#/c/59283/
