..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===========================================
Support paginate for all resource list APIs
===========================================

https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-events-pagination

This blueprint adds support to the generic resource listing call for limit
marker, and sort_dir(sorted by timestamp) and groupby(especially by event
types) filters, allowing users of the API to retrieve a subset of resources or
provide a grouping return.

Problem description
===================

It is now highly probable that a event-list call could end up attempting to
return hundreds of events. The event-list API with no pagination is not easy
to use, and may return 500 error if the response is too large.

Actually, there is already a blueprint[2] for pagination of ceilometer APIs
proposed by Fengqian Gao<fengqian.gao@intel.com>. The blueprint has finished
some works, but didn't finish all the proposed goals. Since the assignee is no
longer working on OpenStack anymore, I would like to continue the uncompleted
items of the blueprint and add the alarm/resource/meter/sample pagination as
the proposed changes to this spec.

Proposed change
===============

1. We should support event pagination with limit and marker query parameters.
Additionally, we should allow users specifying the sort_dir('asc' or 'desc') of
the return events, and support the groupby filter of listing events.

    - limit: the number of events to list

    - marker: the message_id of the last item in the previous page

    - sort_dir: the direction of the sorting, 'asc' or 'desc', 'desc' as
      default

    - sort_key: the key of the sorting, such as "generated" of event.

2. In storage layer, for alarm and event feature has dedicated storage
   implementation, so this blueprint will propose to construct pagination
   query, marker query, sort query methods base on different storage backends
   (sqlalchemy, mongodb, hbase, db2) and different data model(alarm data, event
   data, metering data), this will continue the implemented parts by
   Fengqian Gao.

3. To continue the works blueprint[2] has proposed, this blueprint will also
   propose implement alarm/resource/meter/sample pagination of in ceilometer,
   like the event pagination mentioned above. There is a mail thread that has
   discussed the implementation of this blueprint[3].

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

A example of Restful API of event pagination::

  GET /v2/events?limit=2&sort_key=generated&sort_dir=dsc

  Response: 200 OK
  RESP BODY:  [{
                  "traits": [{
                      "type": "string",
                      "name": "state",
                      "value": "active"
                  },...],
                  "generated": "2014-10-12T10:00:13.800050",
                  "message_id": "ad39e927-e68e-48ef-9a05-281298dc142a",
                  "event_type": "compute.instance.delete.start"
              },
              {
                  "traits": [{
                      "type": "string",
                      "name": "state",
                      "value": "active"
                  },...],
                  "generated": "2014-10-12T10:00:13.357174",
                  "message_id": "bf13223c-0d61-4759-8e9a-dc754da833eb",
                  "event_type": "compute.instance.update"
              }]

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
  liusheng<liusheng@huawei.com>

Other contributors:
  ZhiQiang Fan<aji.zqfan@gmail.com>
  fengqian-gao<fengqian.gao@intel.com>

Ongoing maintainer:
  liusheng<liusheng@huawei.com>

Work Items
----------

- Implement pagination/marker/sort query base on the implementation in oslo:
  *oslo.db.utils:paginate_query* for alarm/event/metering/resource

- Complete pagination/marker/sort query to continue the completed parts(by
  Fengqian Gao) in impl_mongodb

- Implement the pagination/marker query constructor methods in impl_db2

- Implement pagination/marker/sort query for alarm/event/metering/resource in
  impl_db2 (continue the Fengqian Gao's works)

- Implement the pagination/marker/sort query constructor methods in impl_hbase

- Implement pagination/marker/sort query for alarm/event/metering/resource in
  impl_hbase

- Implement the API pagination/marker/sort query support for
  alarm/event/resource/meter/sample

Future lifecycle
================

None

Dependencies
============

None

Testing
=======

Add tempest api tests and scenario tests to exercise pagination in ceilometer

Documentation Impact
====================

Update the relevant documentation about this api change

References
==========

[1] https://blueprints.launchpad.net/ceilometer/+spec/ceilometer-events-pagination

[2] https://blueprints.launchpad.net/ceilometer/+spec/paginate-db-search

[3] http://lists.openstack.org/pipermail/openstack-dev/2014-January/024687.html

[4] https://review.openstack.org/#/c/128418/
