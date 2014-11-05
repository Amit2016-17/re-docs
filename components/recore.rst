.. _re_core:

RE-CORE
-------

The core is essentially a finite state machine (**FSM**) hooked into a
message bus and a database.

The core oversees the execution of all *release steps* for any given
project. The core is separate from the actual execution of each
release step. Execution is delegated to the **worker** component.

.. contents::
   :depth: 2
   :local:


Running From Source
~~~~~~~~~~~~~~~~~~~

.. code-block:: bash

   $ . ./hacking/setup-env
   $ re-core -c ./examples/settings-example.json


.. _recore-conf:

RE-CORE Configuration
~~~~~~~~~~~~~~~~~~~~~
Configuration of the server is done in JSON. You can find an example
configuration file in the `examples/
<https://github.com/RHInception/re-core/tree/master/examples>`_
directory.

You must point to a specific configuration file using the ``-c``
command-line option to start the FSM:

.. code-block:: bash

   $ re-core -c settings.json

Descriptions of all settings directives:

================== ====== ================== ===========================================
Name               Type   Parent             Value
================== ====== ================== ===========================================
LOGFILE            str    None               File name for the application level log
RELEASE_LOG_DIR    str    None               Directory for per-release logging (default: ``None``)
MQ                 dict   None               Where all of the MQ connection settings are
SERVER             str    MQ                 Hostname or IP of the server
NAME               str    MQ                 Username to connect with
PASSWORD           str    MQ                 Password to authenticate with
QUEUE              str    MQ                 Queue on the server to bind
VHOST              str    MQ                 VHost to connect to on the server
PORT               int    MQ                 AMQP connection port (default ``5672`` for non-ssl, ``5671`` for ssl)
SSL                bool   MQ                 If the AMQP connection should be SSL/TLS secured (default ``false``)
DB                 dict   None               Where all the DB connection settings are
SERVERS            list   DB                 List of all of the MongoDB hostname/IPs
DATABASE           str    DB                 Name of the MongoDB database
NAME               str    DB                 Username to connect with
PASSWORD           str    DB                 Password to authenticate with
PHASE_NOTIFICATION dict   None               Notifications that will always happen in a phase
TABOOT_URL         str    PHASE_NOTIFICATION URL with ``%s`` to taboot tailer EX: http://example.com/taboot/%s/
TOPIC              str    PHASE_NOTIFICATION The topic (`routing key`) to send notification on. EX: ``notify.irc``.
TARGET             list   PHASE_NOTIFICATION The targets to send the notification to. EX: ``["#mychannel", "auser"]``
PRE_DEPLOY_CHECK   list   None               The yes/no checks to make prior to deployment (:ref:`see below <components_recore_predeployment_checks>`  for more information)
POST_DEPLOY_ACTION list   None               Steps which are ran after the playbook has been ran (:ref:`see below <components_recore_postdeployment_action>`  for more information)
================== ====== ================== ===========================================

For a full example see `example-config.json <https://github.com/RHInception/re-core/blob/master/examples/settings-example.json>`_.

MQ Configuration Notes
++++++++++++++++++++++

There are two optional parameters in the MQ configuration section:
``PORT``, and ``SSL``. Their defaults are shown below:

* ``PORT`` - 5672 (*rabbitmq no-ssl*)
* ``SSL`` - ``false``

If ``SSL`` is not set (or is ``false``) then **re-core** uses the
default rabbit MQ port (5672), unless a port has been specified in the
config file. If ``SSL`` is set to ``true`` then **re-core** uses the
RabbitMQ SSL port (5671), unless a port has been specified in the
configuration file.

Simply put, you don't need to set ``SSL`` or ``PORT`` unless:

* You want to enable SSL (in which case, set ``SSL`` to ``true`` in
  the config file)
* You are running RabbitMQ on non-standard ports.

Here's a bare-minimum MQ configuration section:

.. code-block:: json
   :linenos:

   {
       "MQ": {
           "EXCHANGE": "my_exchange",
           "NAME": "username",
           "PASSWORD": "password",
           "QUEUE": "re",
           "SERVER": "amqp.example.com",
           "VHOST": "/"
       }
   }

Note that ``PORT`` and ``SSL`` are not set. Therefore this will open
an unencrypted connection to Rabbit MQ using the default port (5672).


Here's a bare-minimum MQ configuration file for an encrypted
connection:

.. code-block:: json
   :linenos:
   :emphasize-lines: 8

   {
       "MQ": {
           "EXCHANGE": "my_exchange",
           "NAME": "username",
           "PASSWORD": "password",
           "QUEUE": "re",
           "SERVER": "amqp.example.com",
           "SSL": true,
           "VHOST": "/"
       }
   }

Note on line **8** that we set ``SSL`` to ``true`` (remember, it's
lower-case "true" in JSON files) and we are not setting the port. In
this case the port is automatically set to 5671.

And now a non-standard configuration:

.. code-block:: json
   :linenos:
   :emphasize-lines: 6,9

   {
       "MQ": {
           "EXCHANGE": "my_exchange",
           "NAME": "username",
           "PASSWORD": "password",
           "PORT": 5672,
           "QUEUE": "re",
           "SERVER": "amqp.example.com",
           "SSL": true,
           "VHOST": "/"
       }
   }

In this **confusing** and **non-standard** configuration we are
connecting to an SSL enabled RabbitMQ server which is listening for
SSL connections on port 5672, a port which is normally reserved for
non-SSL connections.


.. _recore-deployment:

RE-CORE Deployment
~~~~~~~~~~~~~~~~~~

From RPM Package
++++++++++++++++

* Install the ``re-core`` RPM package
* Create a ``settings.json`` file (use `example-config.json
  <https://github.com/RHInception/re-core/blob/master/examples/settings-example.json>`_
  as a starting point)
* Start the core:

.. code-block:: bash

   $ re-core -c ./settings.json


From Source
+++++++++++

* Change into the directory you cloned **re-core** into
* Run **screen**
* Update your :ref:`re-core config file <recore-conf>` with appropriate values
* Update your paths by running: ``. ./hacking/setup-env``
* Run ``re-core -c path/to/settings.json``

You should see output similar to the following:

.. code-block:: bash

   [~/release-engine/re-core] $ re-core -c ./real-settings.json
   2014-05-19 13:56:00,179 - __init__:start_logging:43 - DEBUG - initialized stdout logger
   2014-05-19 13:56:00,180 - __init__:parse_config:53 - DEBUG - Parsed configuration file

Additional output will be directed to the log file you configured in
the ``settings.json`` file. The default log file is called
``recore.log`` and will be in your present working directory.


Per-release Logging
~~~~~~~~~~~~~~~~~~~

By default, the FSM will log to the console and a single logfile
(``LOGFILE``).

Optionally, one may log the FSM activity for **each release** to a
separate file. This is done by configuring the re-core
``RELEASE_LOG_DIR`` setting with the path to the log-holding
directory.

If per-release logging is enabled, the log files will be created as:
``RELEASE_LOG_DIR/FSM-STATE_ID.log``

.. warning::

   Be sure the FSM has permission to write the specified
   directory. You won't find out it can't until the first release is
   attempted.


.. code-block:: json
   :linenos:
   :emphasize-lines: 3

   {
       "LOGFILE": "recore.log",
       "RELEASE_LOG_DIR": "/var/log/recore",
       "MQ": {
           "SERVER": "amqp.example.com"
      }
   }


.. _components_recore_predeployment_checks:

Pre-Deployment Checks
~~~~~~~~~~~~~~~~~~~~~

An re-core instance may be configured to run one or more scripts prior
to the deployment of any playbook. Each pre-deployment check defines
the command to run and the expected result from the command. If
expected equals observed, then the check is considered to have
passed. If expected is not equal to observed, then the check has
failed and the entire deployment is marked as failed.

.. important:: These checks apply to *all* deployments

Configuration of pre-deployment checks takes place in the re-core
``setting.json`` file.

Example settings

.. code-block:: json
   :linenos:
   :emphasize-lines: 6-16

   {
       "LOGFILE": "recore.log",
       "RELEASE_LOG_DIR": null,

       "PRE_DEPLOY_CHECK": [{
           "Require Change Record": {
               "COMMAND": "servicenow",
               "SUBCOMMAND": "getchangerecord",
               "PARAMETERS": {},
               "EXPECTATION": {
                   "status": "completed",
                   "data": {
                       "exists": true
                   }
               }
           }
       }]
   }


Here we see a new directive, ``PRE_DEPLOY_CHECK`` (line **5**), this
key holds a list whose members are nested dictionaries (lines **6** →
**16**). This example has one nested-dictionary. It has one key, that
is the name of the check, **Require Change Record**. You can give any
name you want to keys as long as it is JSON parsable.

Now let's look at this nested-dictionary closer:

.. code-block:: python

   {
       "COMMAND": "servicenow",
       "SUBCOMMAND": "getchangerecord",
       "PARAMETERS": {},
       "EXPECTATION": {
           "status": "completed",
           "data": {
               "exists": true
           }
       }
   }


* ``COMMAND`` - Name of the worker to run the check with,
  :ref:`re-worker-servicenow <steps_servicenow>` in this example
* ``SUBCOMMAND`` - The specific sub-command to run on that worker
* ``PARAMETERS`` - Dictionary with variable keys depending on what your worker requires
* ``EXPECTATION`` - The result we expected to get back from the check.

**Pass or fail** is determined by comparing the *actual* response
against ``EXPECTATION``. If they are the same then the check
passes. If they differ then the check fails and the deployment is
marked as *failed* and aborted.

.. _components_recore_postdeployment_action:

Post-Deployment Action
~~~~~~~~~~~~~~~~~~~~~~

Similar in functionality to the :ref:`pre-deployment check
<components_recore_predeployment_checks>`, re-core allows us to run a
set of worker steps **after** each deployment as well. What makes
**POST_DEPLOY_ACTION** different from **PRE_DEPLOY_CHECK** is that
pre-deployment checks allow you to specify expected results, whereas
post-deploy actions only check for ``completed`` or ``failed``
return-statuses.

If a post-deploy action fails (by returning a ``status`` other than
``completed``), then the deployment is recorded as a failure.

.. important:: These actions apply to *all* deployments

Configuration of pre-deployment checks takes place in the re-core
``setting.json`` file.

Let's take a look at example settings for a post-deploy action which
records the time the deployment finished in a Service Now change
record.

.. code-block:: json
   :linenos:

   {
       "POST_DEPLOY_ACTION": [{
           "Update Change Record": {
               "COMMAND": "servicenow",
               "SUBCOMMAND": "UpdateEndTime",
               "PARAMETERS": {}
           }
       }]
   }

In this example we're defining an empty ``PARAMETERS`` key. This is
because the :ref:`ServiceNow <steps_servicenow>` worker's
:ref:`UpdateEndTime <steps_servicenow_updateendtime>` sub-command only
requires dynamic arguments. The FSM (*re-core*) will send dynamic
arguments to the worker automatically.


RE-CORE Disconnection
~~~~~~~~~~~~~~~~~~~~~

Much like the worker component **re-core** will attempt to reconnect to the
bus if it gets disconnected. However, a disconnect of **re-core** is likely
to be disruptive due to the use of temporary queues. Do not expect a flaky
**re-core** connection to be sufficient. A disconnection of **re-core**
should always stop a deployment dead in it's tracks and not restart it
if able to reconnect.

.. warning::

   If a deployment is taking place while **re-core** is disconnected it's highly unlikely that a notification will be sent out showing a failure.
