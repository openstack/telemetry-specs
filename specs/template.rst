..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Example Spec - The title of your blueprint
==========================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/ceilometer/+spec/example
https://blueprints.launchpad.net/python-ceilometerclient/+spec/example

Introduction paragraph -- why are we doing anything? A single paragraph of
free-form text that other developers and operators can understand.

Some notes about using this template:

* Your spec should be in ReSTructured text, like this template.

* Please wrap text at 79 columns.

* The filename in the git repository should match the launchpad URL, for
  example a URL of:

    https://blueprints.launchpad.net/ceilometer/+spec/awesome-thing

  should be named awesome-thing.rst

* Please do not delete any of the sections in this template.  If you have
  nothing to say for a whole section, just write: None

* For help with syntax, see http://sphinx-doc.org/rest.html

* To test out your formatting, build the docs using tox, or see:
  http://rst.ninjs.org

* If you would like to provide a diagram with your spec, ascii diagrams are
  required.  http://asciiflow.com/ is a very nice tool to assist with making
  ascii diagrams.  The reason for this is that the tool used to review specs is
  based purely on plain text.  Plain text will allow review to proceed without
  having to look at additional files which can not be viewed in Gerrit.  It
  will also allow inline feedback on the diagram itself.

* If your specification proposes any changes to the Ceilometer REST API such
  as changing parameters which can be returned or accepted, or even
  the semantics of what happens when a client calls into the API, then
  you should add the APIImpact flag to the commit message. Specifications with
  the APIImpact flag can be found with the following query::

    https://review.openstack.org/#/q/status:open+project:openstack/ceilometer-specs+message:apiimpact,n,z


Problem description
===================

A detailed description of the problem:

* For a new feature this might be use cases. Ensure you are clear about the
  actors in each use case: End User vs Cloud Operator

* For a major reworking of something existing it would describe the
  problems in that feature that are being addressed. In this case, any
  potential migration issues must be called out upfront.

* For a major functional area not currently addressed within the
  OpenStack Telemetry program, ensure you describe why you think
  this is appropriate given our project mandate and mission statement.

Proposed change
===============

Here is where you cover the change you propose to make in detail. How do you
propose to solve this problem?

If this is one part of a larger effort make it clear where this piece ends. In
other words, what's the scope of this effort? If this larger effort may span
several release cycles, state this explicitly.

Alternatives
------------

What other ways could we do this thing? Why aren't we using those? This doesn't
have to be a full literature review, but it should demonstrate that thought has
been put into why the proposed solution is an appropriate one. Especially if
there's a history in the community of divided opinion on this issue.

Data model impact
-----------------

Changes which require modifications to the data model often have a wider impact
on the system and its performance, or lack thereof.  The community often has
strong opinions on how the data model should be evolved, from both a functional
and performance perspective. It is therefore important to capture and gain
agreement as early as possible on any proposed changes to the data model.

Questions which need to be addressed by this section include:

* What new data objects and/or database schema changes is this going to
  require?

* What database migrations will accompany this change, treating the SQLAlchemy
  and NoSQL cases separately.

* Will this add to the on-the-fly data massaging cruft that we've accreted
  over time?

* How will the initial set of new data objects be generated, for example if you
  need to take into account existing instances, or modify other existing data
  describe how that will work.

REST API impact
---------------

Each API method which is either added or changed should have the following

* Specification for the method

  * A description of what the method does suitable for use in
    user documentation

  * Method type (POST/PUT/GET/DELETE)

  * Normal http response code(s)

  * Expected error http response code(s)

    * A description for each possible error code should be included
      describing semantic errors which can cause it such as inconsistent
      parameters supplied to the method, or when an instance is not in an
      appropriate state for the request to succeed. Errors caused by
      syntactic problems covered by the JSON schema definition do not need
      to be included.

  * URL for the resource

  * Parameters which can be passed via the url

  * example JSON fragments for the body data if appropriate

  * example JSON fragments for the response data if any

* Example use case including typical API samples for both data supplied
  by the caller and the response.

* Discuss any policy changes, and discuss what things a deployer needs to
  think about when defining their policy.

* Discuss whether this change should be backported to any currently supported
  API versions (e.g. to v2 when this is put on the deprecation path in favor
  of a new v3 API)

Security impact
---------------

Describe any potential security impact on the system.  The principal issue
to consider is:

* Does this change impact on the direct or indirect visibility of data
  in the metering store in a way that doesn't respect full segregation
  between non-admin tenants.

An example of such a concern would the on_behalf_of mechanism in the
alarm evaluation logic.

For more detailed guidance, please see the OpenStack Security Guidelines as
a reference (https://wiki.openstack.org/wiki/Security/Guidelines).  These
guidelines are a work in progress and are designed to help you identify
security best practices.  For further information, feel free to reach out
to the OpenStack Security Group at openstack-security@lists.openstack.org.

Pipeline impact
---------------

Please specify any changes to the metering pipeline, from the data acquisition
agents, via the publication conduit(s), through to the database dispatch layer.
For example:

* Is yet another agent required to host the data acquisition pollsters or
  notification handlers?

* If accommodated in an existing agent, is the scaling of that agent impacted?

* Is explicit configuration of the source and/or transformations required
  in the pipeline.yaml?

* Is the typical cadence of data acquisition likely in practice to be unusually
  frequent or infrequent?

* Is an explicit resource discovery extension required to retrieve target
  resources?

* Is AMQP the appropriate publication conduit for these data?

* Is any change required to the metering message signature verification
  used by the collector?

Other end user impact
---------------------

Aside from the API, are there other ways a user will interact with this
feature?

* If a service-side feature, does this change also have an impact on
  python-ceilometerclient? What does the user interface there look like?

* Should this feature be exposed via the Horizon metering dashboard?

Performance/Scalability Impacts
-------------------------------

Describe any potential performance or scaling impact on the system, considering
for example:

* The volume of new metering data generated, and the knock-on impact
  of this on the latency of the publication conduit and database dispatch
  layer.

* Whether any new data retention policies are required.

* How any new APIs and/or storage driver methods will perform when scaled
  over very large datasets.

* Whether any explicit performance testing would be advisable to validate
  the new feature, either at the PoC stage, and/or in its final form.


Other deployer impact
---------------------

Discuss things that will affect how you deploy and configure OpenStack that
have not already been mentioned, such as:

* What config options are being added?

* How is the storage driver feature parity matrix impacted? Traditionally
  new features were often only supported initially in the MongoDB and
  SQLAlchemy drivers, leaving the more niche drivers to catch up later.
  Though this is established custom and practice, you must explicitly
  state which drivers you intend to address in the first cut.

* Is this a change that takes immediate effect after its merged, or is it
  something that has to be explicitly enabled?

* If this change is a new binary, how would it be deployed? Will the puppet
  or chef recipes in wide use require extension to accommodate this feature.

* Please state anything that those doing continuous deployment, or those
  upgrading from the previous release, need to be aware of. Also describe
  any plans to deprecate configuration values or features.  For example, if we
  change the pipeline.yaml format, how do we handle pipelines created before
  the change landed?  Do we transform them?  Do we continue to support the
  old format in a deprecated form?

* Please state anything that those doing downstream distro-oriented
  packaging need to be aware of. For example, is a new service being added,
  or many new transitive dependencies pulled in, or a new feature that is
  effectively optional and hence suited to separate packaging.

Developer impact
----------------

Discuss things that will affect other developers working on OpenStack,
such as:

* If the blueprint proposes a change to the internal storage driver or
  hypervisor inspector APIs, discussion of how existing implementations
  of these APIs would implement the feature is required.


Implementation
==============

Assignee(s)
-----------

Who is leading the writing of the code? Or is this a blueprint where you're
throwing it out there to see who picks it up?

If more than one person is working on the implementation, please designate the
primary author and contact.

Primary assignee:
  <launchpad-id or None>

Other contributors:
  <launchpad-id or None>

Ongoing maintainer:
  <launchpad-id or None>

Work Items
----------

Work items or tasks -- break the feature up into the things that need to be
done to implement it. Those parts might end up being done by different people,
but we're mostly trying to understand the timeline for implementation.


Future lifecycle
================

The Telemetry program is explicitly not interested in "code drops", where
some new niche feature is landed, but then ongoing active maintainership
is not provided by either the original author and/or an obviously sustainable
user community. You must address how you envisage the ongoing maintenance
of the feature being handled through the next two release cycles.


Dependencies
============

* Include specific references to specs and/or blueprints under the Telemetry
  program, or in other programs, that the current blueprint one either depends
  on or is related to.

* If this requires functionality of another program that is not currently
  used by Telemetry (such as a new or extended library provided by the Oslo
  program), document that fact.

* Does this feature require any new external dependencies or code otherwise not
  included in OpenStack? Or does it depend on a specific version of library? Is
  this library already packaged for the major distros (i.e. derivatives of
  Debian and Fedora).


Testing
=======

Please discuss how the change will be tested. We especially want to know what
Tempest tests will be added. It is assumed that unit and scenario test coverage
will be added so that doesn't need to be mentioned explicitly, but discussion
of why you think unit/scenario tests are sufficient and we don't need to add
more tempest testcases would need to be included.

Is this untestable in the upstream gate given current limitations (specific
hardware / software configurations available)? If so, are there mitigation
plans (3rd party testing, gate enhancements, etc.).


Documentation Impact
====================

What is the impact on the docs team of this change? Some changes might require
donating resources to the docs team to have the documentation updated. Don't
repeat details discussed above, but please reference them here.


References
==========

Please add any useful references here. You are not required to have any
reference. Moreover, this specification should still make sense when your
references are unavailable. Examples of what you could include are:

* Links to mailing list or IRC discussions

* Links to notes from a summit session

* Links to relevant research, appropriately distilled or summarized

* Related specifications as appropriate (e.g.  if it's calling out to a REST
  API exposed by another OpenStack service, link to that API definition)

* Anything else you feel it is worthwhile to refer to

