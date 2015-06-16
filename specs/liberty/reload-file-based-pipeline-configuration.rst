..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================================================
Dynamic pipeline configuration using File Reloading
===================================================

https://blueprints.launchpad.net/ceilometer/+spec/reload-file-based-pipeline-configuration

Currently there is no way to enable or disable meters without restarting ceilometer.
There are cases where operators do not want to run all the meters continuously.

By enhancing Ceilometer to monitor pipeline file configuration, the application
could be made to dynamically activate/deactivate meters, collection targets or similar
functions, have distinctly different configurations for multiple nodes in different
environments (i.e. dev/test/prod or HA scenarios) and could ultimately be updated
with new collection targets “on-the-fly”.


Problem description
===================

Currently, Ceilometer relies on pipeline configuration file for determining
run-time parameters used in polling and notification handling.

There is no way to enable or disable meters without restarting ceilometer.
There are cases where operators do not want to run all the meters
continuously. In these cases, there should be a way to disable or enable them
dynamically (without restarting ceilometer).

Meters in ceilometer are enabled by adding them in “setup.cfg:entry_points”
configuration file and restarting ceilometer agent.

When ceilometer agent restarts, during initialization it reads the
entry_points and creates and loads all pollster objects corresponding to
meters present in setup.cfg.

There are disadvantages with this implementation:

Ceilometer might be running several hundreds of meters continuously and
restarting it might impact other meters as it involves the deletion and
re-creation of the hundreds of pollster objects, when the need is to poll
a few meters or avoid polling a few meters.

The subsequent sections will cover an approach to dynamically reload and
update agent pipeline configuration. (pipeline.yaml)


Proposed change
===============

Summary:

File polling by agent - Each agent polls its local pipeline.yaml file for
changes and re-configures pollsters/listeners on detecting a change.

By default, this will be turned off. Its an optional feature and can be
turned on by specifying reload_pipeline_config = True in ceilometer.conf
under the default section.


The proposed high-level flow:

1. Each agent daemon polls local pipeline configuration file
   at configurable interval.
2. The agents will reload and validate the configuration.
3. If validation succeeds, the new configuration is used.

Next steps for central-agent, compute-agent and ipmi-agent:

3. For polling agent, determine which pollsters are needed based on new
   configuration and need to run at what frequency (polling interval)
4. For polling-agent, all pollsters in flight are allowed to gracefully
   complete.
5. In HA mode, re-initialize the partition coordinator with new groups
6. Start the new set of pollsters.

Next steps for notification agent:

3. Existing listeners are stopped to allow for graceful termination.
4. Configure new listeners with the changed pipeline configuration.

Next steps for HA notification agent:

3. Pipeline listeners(pipeline sink-based internal queue listener) are
   gracefully terminated.
4. Pipeline listeners are re-configured with new pipeline.


Pros:
   Preferred and Easiest approach
Cons:
   It doesnt centralize the pipeline definition and runs the risk of agents
   diverging on their pipeline definitions

   This means we're allowing any kind of error levels due to the fact file
   might be changed for one agent, and not changed for another one in the
   same coordination group.


Alternatives
------------

1. File polling by separate daemon - Each agent uses a separate daemon
   that polls the pipeline.yaml for changes. On detecting a change, it signals
   (HUP) the agents to re-load the pipeline configuration.

   Pros:
   Allows for future extensibility while keeping the agent to pipeline file
   contract unchanged.
   Cons:
   Same as Previous approach
   HA for the new daemon needed
   Using SIGHUP a potential loophole? (since anyone can send a SIGHUP and have
   the agent reload its configuration)
   More work for less gain

2. HUP-based pipeline reload by agent - The agent uses signal handler to handle
   the HUP signal and reloads pipeline from the file in response to the signal.
   The assumption on receiving a signal is that the pipeline file has changed,
   so the pipeline file can be changed manually or using config management tools
   - Chef/Puppet.

   Pros:
   No API.
   Cons:
   File synchronization within coordination group is deployer responsibility
   Same as Approach 1
   Using SIGHUP a potential loophole? (since anyone can send a SIGHUP and have
   the agent reload its configuration)

3. Use automated deployment tools - Puppet, Chef, Ansible to change pipeline
   definitions. While this automates changing pipeline definitions across
   multiple agents, it doesnt bring the value-add of on-the-fly updates to the
   agent, without incurring a restart of the daemons.

   Pros:
   No API.
   Cons:
   Is looking like the only pure admins approach. Will be more tricky for everyone
   not familiar with these tools (devs, DevOps, etc.)
   Not in ceilometer scope.

4. Use a combination of 2 and 3 - Use configuration management tool to automate
   the change in pipeline configuration across multiple remote agent nodes. Do
   not restart any processes though. Agents on each node poll the pipeline file
   and on detecting a change in the next polling, will update the pollsters and
   listeners. This approach has a dependency with ops on the config mgmt tools.

   Pros:
   No API.
   Cons:
   File synchronization within coordination group is on deployer responsibility


Data model impact
-----------------

REST API impact
---------------

Security impact
---------------

Pipeline impact
---------------

The use of the pipeline configuration will change from static to dynamic.
This will affect the supported meters and their datapoint collection.

The impact to the system is expected to be minimal since changes to pipeline
are expected to be low in frequency.


Other end user impact
---------------------


Performance/Scalability Impacts
-------------------------------

There could be a small window where message build-up occurs in oslo bus/
internal queues when the notification listeners are restarted.

Other deployer impact
---------------------

Depending on the approach chosen, the deployer will have to synchronize
pipeline file updates across multiple agent instances and then signal the
agents to reload the configuration.


Developer impact
----------------

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  rjaiswal

Other contributors:
  TBD

Ongoing maintainer:
  rjaiswal

Work Items
----------

- Implement timer to poll pipeline configuration file
- Implement graceful termination of pollsters and listeners
- Implement reconfiguring and reloading of pollsters in polling-agent
- Implement reconfiguring and reloading of listeners in notification-agent
- Unit Tests
- Doc updates

Future lifecycle
================

We'll want to continue to iterate on this for future cycles to gain additional
functionality:

Use Tooz with publish-subscribe functionality to enable synchronization of
pipeline configuration across multiple agent instances in a single coordination
group.

Migration of event traits configuration


Dependencies
============

Relates to:
https://blueprints.launchpad.net/ceilometer/+spec/dedicated-event-db
https://review.openstack.org/#/c/119077/ (central and compute agents merge)

Testing
=======

Add unit tests to exercise reloading of pipeline in agents

Documentation Impact
====================

Update the relevant documentation about this new feature of reloading pipeline configuration

References
==========

https://etherpad.openstack.org/p/liberty-ceilometer-pipeline-config

https://etherpad.openstack.org/p/configuration_via_data_store

https://review.openstack.org/#/c/171826/

https://wiki.openstack.org/wiki/Ceilometer/blueprints/Configuration-via-data-store

http://docs.openstack.org/developer/oslo.config/configopts.html#oslo_config.cfg.ConfigOpts.reload_config_files