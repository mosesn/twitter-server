HTTP Admin interface
====================

.. _admin_interface:

TwitterServer starts an HTTP server and includes a variety of tools
for diagnostics, profiling, and more.
It binds to the port defined by the :ref:`flag <flags>`
`-admin.port` and uses port 9990 by default.

Visit `/admin` on your service's admin port in a web browser to see what is
available.

.. image:: ../../img/intro.png

TwitterServer defines a series of default endpoints grouped into a variety
of categories.

- :ref:`Process Info <admin_info>`
- :ref:`Utilities <admin_utilities>`
- :ref:`Metrics <admin_metrics>`
- :ref:`Profiling <admin_profile>`
- :ref:`Downstream Clients <admin_clients>`
- :ref:`Listening Servers <admin_servers>`

Process Info
------------

.. _admin_info:

Endpoints that provide insights into your service's configuration and usage.

/admin/lint
~~~~~~~~~~~

Runs and displays the results for all registered linters to check for various
service issues. This can help you identify misuse or abuse of functionality.

.. image:: ../../img/lint.png

/admin/threads
~~~~~~~~~~~~~~

A user interface for capturing the current stacktraces. Includes filtering
of inactive threads as well as deadlock detection. JSON output is also available
through tools like curl via the inspection of the HTTP Accept header.

.. image:: ../../img/threads.png

/admin/registry.json
~~~~~~~~~~~~~~~~~~~~

Displays how the service is currently configured across a variety of dimensions
including the client stack, server stack, flags, service loader values,
system properties, environment variables, build properties and more.

.. image:: ../../img/registry.png

/admin/server_info
~~~~~~~~~~~~~~~~~~

Return build information about this server.
See `/admin/registry.json` for this in addition to other details.

::

  {
    "name" : "myserver",
    "version" : "1.0.0-SNAPSHOT",
    "build" : "20130221-105425",
    "build_revision" : "694299d640d337c58fadf668e44322b17fd0562e",
    "build_branch_name" : "refs/heads/twitter-server!doc",
    "build_last_few_commits" : [
      "694299d (HEAD, origin/twitter-server!doc, twitter-server!doc) Merge branch 'master' into twitter-server!doc",
      "ba1c062 Fix test for sbt + Jeff's comments",
    ],
    "start_time" : "Thu Feb 21 13:43:32 PST 2013",
    "uptime" : 22458
  }

/admin/announcer
~~~~~~~~~~~~~~~~

Returns a set of announcement chains that have run through the
`Announcer`. This allows one to see how a particular target is being
announced.

Utilities
---------

.. _admin_utilities:

Endpoints that cover various utilities that developers can manipulate at runtime.

/admin/events
~~~~~~~~~~~~~

A user interface for collecting and viewing runtime events to make it easier to
diagnose production issues. This includes logging by default, while metrics are included
if you are using ``finagle-stats`` and tracing events are included if you are
using ``finagle-zipkin``. JSON output is also available
through tools like curl via the inspection of the HTTP Accept header.

.. image:: ../../img/events.png

/admin/tracing
~~~~~~~~~~~~~~

Enable (`/admin/tracing?enable=true`) or disable tracing (`/admin/tracing?disable=true`)

See `zipkin <https://github.com/openzipkin/zipkin>`_ documentation for more info
regarding tracing.

/admin/logging
~~~~~~~~~~~~~~

Display the set of loggers and their current log level. The level of
each logger can also be modified on-the-fly.

.. image:: ../../img/logging.png

Metrics
-------

.. _admin_metrics:

Endpoints regarding your service's metrics. The `Finagle User Guide
<https://twitter.github.io/finagle/guide/Metrics.html>`_ explains what
many of them represent.

/admin/metrics
~~~~~~~~~~~~~~

Watch specific metrics in real-time.

.. image:: ../../img/metric_watch.png

/admin/metrics.json
~~~~~~~~~~~~~~~~~~~

Export a snapshot of the current statistics of the program. You can
use the StatsReceiver in your application for add new
counters/gauges/histograms, simply use the `statsReceiver` variable
provided by TwitterServer.

This endpoint is available when you are using the `finagle-stats` library.
See the :ref:`metrics <metrics_label>` section for more information.

Profiling
---------

Endpoints for profiling your process.

.. _admin_profile:

/admin/pprof/contention
~~~~~~~~~~~~~~~~~~~~~~~

Returns a CPU contention profile which identifies blocked threads
(`Thread.State.BLOCKED`).
The output is in `pprof <https://github.com/gperftools/gperftools>`_ format.
The process will be profiled for 10 seconds at a frequency of 100 hz. These
values can be controlled via HTTP request parameters `seconds` and `hz`
respectively.

/admin/contention
~~~~~~~~~~~~~~~~~

Show call stacks of blocked and waiting threads.

::

  $ curl localhost:9990/admin/contention
  Blocked:
  "util-jvm-timer-1" Id=11 TIMED_WAITING on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@33aac3c
    at sun.misc.Unsafe.park(Native Method)
    -  waiting on java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject@33aac3c
    at java.util.concurrent.locks.LockSupport.parkNanos(LockSupport.java:226)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.awaitNanos(AbstractQueuedSynchronizer.java:2082)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:1090)
    at java.util.concurrent.ScheduledThreadPoolExecutor$DelayedWorkQueue.take(ScheduledThreadPoolExecutor.java:807)
    at java.util.concurrent.ThreadPoolExecutor.getTask(ThreadPoolExecutor.java:1043)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1103)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:603)
    ...

/admin/pprof/profile
~~~~~~~~~~~~~~~~~~~~

Returns a CPU usage profile. The output is in `pprof
<https://github.com/gperftools/gperftools>`_ format.
The process will be profiled for 10 seconds at a frequency of 100 hz. These
values can be controlled via HTTP request parameters `seconds` and `hz`
respectively.

::

  $ curl -s localhost:9990/admin/pprof/profile > /tmp/cpu_profiling
  $ pprof --text /tmp/cpu_profiling
  Using local file /tmp/cpu_profiling.
  Using local file /tmp/cpu_profiling.
  Total: 83 samples
        17  20.5%  20.5%       24  28.9% com.twitter.finagle.ProxyServiceFactory$class.status
         8   9.6%  30.1%       10  12.0% scala.collection.immutable.HashMap$HashTrieMap.updated0
         5   6.0%  36.1%       70  84.3% scala.collection.Iterator$class.foreach
         5   6.0%  42.2%        5   6.0% scala.runtime.ScalaRunTime$.hash
         4   4.8%  47.0%        4   4.8% com.twitter.finagle.transport.Transport$$anon$2.status
         4   4.8%  51.8%        4   4.8% sun.management.OperatingSystemImpl.getOpenFileDescriptorCount
         3   3.6%  55.4%        9  10.8% com.twitter.finagle.Filter$$anon$2.status
         ...

/admin/pprof/heap
~~~~~~~~~~~~~~~~~

Returns a heap profile computed by the `heapster agent
<https://github.com/mariusae/heapster>`_.  The output is in
`pprof <https://github.com/gperftools/gperftools>`_ format.

::

  $ java -agentlib:heapster -jar target/myserver-1.0.0-SNAPSHOT.jar
  $ pprof /tmp/heapster_profile
  Welcome to pprof!  For help, type 'help'.
  (pprof) top
  Total: 2001520 samples
   2000024  99.9%  99.9%  2000048  99.9% LTest;main
      1056   0.1% 100.0%     1056   0.1% Ljava/lang/Object;
       296   0.0% 100.0%      296   0.0% Ljava/lang/String;toCharArray
       104   0.0% 100.0%      136   0.0% Ljava/lang/Shutdown;



Downstream Clients
------------------

.. _admin_clients:

The `Finagle clients <https://twitter.github.io/finagle/guide/Clients.html>`_
that your service depends on.

/admin/clients
~~~~~~~~~~~~~~

Surface client information exposed by Finagle. Per-client configuration parameters and
values for each module are available at `/admin/clients/<client name>`.

Listening Servers
-----------------

.. _admin_servers:

The `Finagle servers <https://twitter.github.io/finagle/guide/Servers.html>`_
that your service is running.

/admin/servers
~~~~~~~~~~~~~~

Surface server information exposed by Finagle. Per-server configuration parameters and
values for each module are available at `/admin/clients/<client name>`.
