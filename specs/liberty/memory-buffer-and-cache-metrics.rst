..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=============================================================
Add hardware.memory.buffer and hardware.memory.cached metrics
=============================================================

https://blueprints.launchpad.net/ceilometer/+spec/hardware-memory-buffer-and-cache-metrics

Add hardware.memory.buffer and hardware.memory.cached metrics to monitor
the memory buffer size and memory cache size of a physical machine through
SNMP.


Problem description
===================

Currently Ceilometer only support 4 memory oid of SNMP:
    _memory_total_oid = "1.3.6.1.4.1.2021.4.5.0"
    _memory_avail_real_oid = "1.3.6.1.4.1.2021.4.6.0"
    _memory_total_swap_oid = "1.3.6.1.4.1.2021.4.3.0"
    _memory_avail_swap_oid = "1.3.6.1.4.1.2021.4.4.0"

But in practice, memory cache and buffer size are also very useful information
to determine the status of a physical machine.


Proposed change
===============

Add two metrics, hardware.memory.buffer and hardware.memory.cached, to
monitor the memory buffer size and memory cache size of a physical machine.

To achieve this, we need add two SNMP oid and two hardware pollsters.

Firstly, add two oid in SNMP inspector:

    _memory_buffer_oid = "1.3.6.1.4.1.2021.4.14.0"
    _memory_cached_oid = "1.3.6.1.4.1.2021.4.15.0"

Secondly, add two Hardware Pollsters in hardware.pollsters.memory:

    * MemoryBufferPollster
    * MemoryCachedPollster


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
    luogangyi

Work Items
----------

* Add two oid in SNMP inspector:

    _memory_buffer_oid = "1.3.6.1.4.1.2021.4.14.0"
    _memory_cached_oid = "1.3.6.1.4.1.2021.4.15.0"

* Add two Hardware Pollsters in hardware.pollsters.memory:

    * MemoryBufferPollster
    * MemoryCachedPollster

Future lifecycle
================

None

Dependencies
============

None


Testing
=======

Need unit test

Documentation Impact
====================

None


References
==========

[1] oid references http://www.net-snmp.org/docs/mibs/ucdavis.html

