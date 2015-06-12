..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================
Declarative Notification Handling
=================================

https://blueprints.launchpad.net/ceilometer/+spec/declarative-notifications

The goal of this spec is to propose a more declarative approach to defining
notification events.

Problem description
===================

The existing implementation of notification handling has various limitations.
The issues this proposal addresses include:

* There is no way to selectively listen to specific event topics of interest.
* Capturing notifications of interest should not require code changes.
* Adding new event exchanges involves same routine code changes. Too much code duplication.
* Requires more insight into the code base than needed
* more elegant exchange control

This proposal takes a more declarative approach in addressing these concerns.

Proposed change
===============

Firstly, we define a notification event template where the event signature and attributes are set.
This is simple yaml file of the following format::

  ---
  resources:
    - name: image
       type: "gauge"
       topics:
            - "image.size"
       volume: payload.size
       unit: B
       counter_name: image.size
       tenant_id: payload.tenant_id
       user_id: payload.user_id
       instance_id: payload.instance_id

This essentially depicts the boiler plate code we add in notifications.py. A base
notification handler loads the event definition and does the rudimentary steps of
calling the process_notifications and yielding samples. The advantage of this is that
in future when new events need to be added, the end user only need to update the yaml.

Also note that we're in the process of deprecating existence meters. All the current
existence meters will be put at the bottom of the template file with a note saying they
will be removed in 'M' release.

Secondly, we need to elegantly handle exchange control as part of this definition.
The exchanges are specified in ceilometer.conf

We already have exchange definitions as part of ceilometer.conf. We could leverage these
as it is and assume that if these are defined then we will enable them. We could also
add a new option to specify all the exchanges we like to allow with same exchange name
as defined. These exchanges will be read by the base notification handler and get_targets
logic will loop through the allowed exchanges and sets up the exchanges. If an event name
is part of the exchange names, we'll skip processing these notifications and move on.

Alternatives
------------

We could leverage the existing event_definition.yaml used for conversion.
But i'm not sure how free flowing and flexible that would be. I'm open to
this idea.

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
  Pradeep Kilambi <pkilambi@redhat.com>

Other contributors:
  Pradeep Kilambi <pkilambi@redhat.com>

Work Items
----------

Work items:
* Define Event notification template
* Implement generic notification handler to process the template and generate samples
* Add support for handling exchange control
* Clean up existing handlers
* Add and update test coverage

Future lifecycle
================

None


Dependencies
============

None


Testing
=======

Add unit test coverage

Documentation Impact
====================

Document new ways of enabling notification events and exchanges via the template file with examples.

References
==========

* https://etherpad.openstack.org/p/ceilo-declarative-notifications
