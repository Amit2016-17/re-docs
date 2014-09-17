.. _steps_func_command:

Command
=======

The **command** module allows you to run arbitrary commands on a
remote host. It has one sub-command available, **run**.

command:run
-----------

**Parameters**

* ``cmd`` (type: string)

  * **Required:** True
  * **Description:** The command to run, as it would be typed into a shell prompt

**Example**

.. code-block:: yaml
   :linenos:
   :emphasize-lines: 3-4

   hosts: ['localhost']
   steps:
       - command:run:
           cmd: puppet agent --test --color=false
