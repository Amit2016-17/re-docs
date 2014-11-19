.. _steps_servicenow:

ServiceNow
**********

.. note:: All checks are ran for each host in ``hosts``

.. note:: All ServiceNow steps are more useful as a :ref:`Pre-Deployment Step <components_recore_predeployment_checks>`


.. _steps_servicenow_doeschangerecordexist:

servicenow:DoesChangeRecordExist
================================

Checks to see if a change record exists. Resulting data will have an exists key with a bool.

**Dynamic Arguments**

* ``change_record`` (type ``str``)

  * **Required:** True
  * **Description:** The change record to look for.

**Example**

To check if a change record exists:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 3,4

   hosts: ['localhost']
   steps:
       - servicenow:DoesChangeRecordExist:
           dynamic:
               - change_record

.. _steps_servicenow_updatestarttime:

servicenow:UpdateStartTime
==========================

Updates the custom start time field for an environment.

**Dynamic Arguments**

* ``change_record`` (type ``str``)

  * **Required:** True
  * **Description:** The change record to look for.

* ``environment`` (type ``str``)

  * **Required:** True
  * **Description:** The target environment.


**Example**


To update the start time:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 3,4

   hosts: ['localhost']
   steps:
       - servicenow:UpdateStartTime:
           dynamic:
               - environment
               - change_record


.. _steps_servicenow_updateendtime:

servicenow:UpdateEndTime
==========================

Updates the custom end time field for an environment.

**Dynamic Arguments**

* ``change_record`` (type ``str``)

  * **Required:** True
  * **Description:** The change record to look for.

* ``environment`` (type ``str``)

  * **Required:** True
  * **Description:** The target environment.


**Example**


To update the start time:

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 3,4

   hosts: ['localhost']
   steps:
       - servicenow:UpdateEndTime:
           dynamic:
               - environment
               - change_record



.. _steps_servicenow_create_change_record:

servicenow:CreateChangeRecord
=============================

Create a change record automatically.

.. note:: Creating change records requires a non-trivial amount of
          integration work. This subcommand will **not** work
          out-of-the-box.

          See the main :ref:`re-worker-servicenow
          <workers_servicenow>` documentation for additional
          information.


**Arguments**

* *This subcommand accepts no arguments*


**Example**


To create a change record

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 3

   hosts: ['localhost']
   steps:
       - servicenow:CreateChangeRecord
