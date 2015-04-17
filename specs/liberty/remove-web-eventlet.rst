..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

===================================
Remove Eventlet from the API Server
===================================

https://blueprints.launchpad.net/ceilometer/+spec/remove-web-eventlet

As of Kilo, the WSGI application which provides the Ceilometer API can be run
in two fundamentally different ways:

  * As a Python command that runs a Werkzeug-based web server that is
    monkeypatched to use eventlet.
  * As a WSGI application hosted by any WSGI server, often Apache + mod wsgi.

Hosting the server application in a dedicate WSGI host has performance,
configuration and scaling advantages. Running the command line service, while
convenient for simple testing, can result in difficult to diagnose bugs and
unusual behaviors when it is monkeypatched to use eventlet. Since the Werkzeug
server can (and as written, does) provide support for multi-process and
multi-threaded interactions, the inclusion of eventlet should be removed.

Problem description
===================

Eventlet monkeypatches the socket module to provide non-blocking network I/O.
This can work well most of the time, but when unusual socket situations arise
(for example the client frequently closes the connection before reading all the
data the server would like to send) the produced tracebacks can be more
difficult to use as debugging aids than those that are produced without
eventlet in place. We want our tooling to be as useful as possible *especially*
when there are situations that require debugging.

When using the Werkzeug service, using eventlet is fairly redundant as the
web server is configured to run multiple processes.

Proposed change
===============

We can resolve this problem relatively easily. At the moment eventlet monkey
patching is turned on for everything that is imported under ``ceilometer.cmd``
(through ``ceilometer/cmd/__init__.py``). This is overkill. We should evaluate
which services benefit from it and turn it on only for those. We can start with
the API server.

Further, for the sake of guiding people to the best solutions, we should update
documentation and guidance to downstreams to indicate that using a WSGI host is
the better way to host the API service in production.

Alternatives
------------

* Do nothing. This is a hygienic change that will benefit users and developers
  but is not strictly required.


Data model impact
-----------------

None

REST API impact
---------------

None. If there is anything we've messed up.

Security impact
---------------

None.

Pipeline impact
---------------

None.

Other end user impact
---------------------

None.

Performance/Scalability Impacts
-------------------------------

For production environments, the guidance to use a WSGI host will provide more
options for adjusting configuration for scaling and performance.

Other deployer impact
---------------------

Deployers who wish to take advantage of the more direct guidance to use a WSGI
host will have to do what it says.

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  chdent

Other contributors:
  None

Ongoing maintainer:
  chdent

Work Items
----------

* Seperate eventlet and non-eventlet commands into two different module
  directories, starting with the api module.
* Compare and contrast performance of the API server with and without eventlet
  paying specific attention to the impact on accessing the storage layer. Mike
  Bayer has pointed out that removing eventlet has little impact on a properly
  configured web server with a properly configured storage layer. We need to
  confirm this across the three styles of presenting the APU: WSGI host,
  Werkzeug with eventlet, Werkzeug without eventlet.
* Update developer and admin documentation.


Future lifecycle
================

This is a core service which will receive ongoing maintenance from the
Ceilometer project. Once the API server is not using eventlet we can
see if any of the other console scripts would benefit from the change.


Dependencies
============

No new dependencies.


Testing
=======

Existing API tests will cover these changes.


Documentation Impact
====================

As mentioned above developer and admin documentation will need to be updated to
reflect the new guidance about the best service configuration.


References
==========

None
