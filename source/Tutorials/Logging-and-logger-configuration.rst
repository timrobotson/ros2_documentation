.. redirect-from::

    Logging-and-logger-configuration

Logging and logger configuration demo
=====================================

See `the logging page <../Concepts/Logging>` for details on available functionality.

In this demo, different types of log calls are shown and the severity level of different loggers is configured locally and externally.

Start the demo with:

.. code-block:: bash

   ros2 run logging_demo logging_demo_main

Over time you will see output from various log calls with different properties.
To start with you will only see output from log calls with severity ``INFO`` and above (``WARN``, ``ERROR``, ``FATAL``).
Note that the first message will only be logged once, though the line is reached on each iteration, as that is a property of the log call used for that message.

Logger level configuration: programmatically
--------------------------------------------

After 10 iterations the level of the logger will be set to ``DEBUG``, which will cause additional messages to be logged.

Some of these debug messages cause additional functions/expressions to be evaluated, which were previously skipped as ``DEBUG`` log calls were not enabled.
See `the source code <https://github.com/ros2/demos/blob/master/logging_demo/src/logger_usage_component.cpp>`__ of the demo for further explanation of the calls used, and see the rclcpp logging documentation for a full list of supported logging calls.

Logger level configuration: externally
--------------------------------------

In the future there will be a generalized approach to external configuration of loggers at runtime (similar to how `rqt_logger_level <http://wiki.ros.org/rqt_logger_level>`__ in ROS 1 allows logger configuration via remote procedural calls).
**This concept is not yet officially supported in ROS 2.**
In the meantime, this demo provides an **example** service that can be called externally to request configuration of logger levels for known names of loggers in the process.

The demo previously started is already running this example service.
To set the level of the demo's logger back to ``INFO``\ , call the service with:

.. code-block:: bash

   ros2 service call /config_logger logging_demo/srv/ConfigLogger "{logger_name: 'logger_usage_demo', level: INFO}"

This service call will work on any logger that is running in the process provided that you know its name.
This includes the loggers in the ROS 2 core, such as ``rcl`` (the common client library package).
To enable debug logging for ``rcl``, call:

.. code-block:: bash

   ros2 service call /config_logger logging_demo/srv/ConfigLogger "{logger_name: 'rcl', level: DEBUG}"

You should see debug output from ``rcl`` start to show.

Using the logger config component
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The server that responds to the logger configuration requests has been developed as a component so that it may be added to an existing composition-based system.
For example, if you are using `a container to run your nodes <Composition>`, to be able to configure your loggers you only need to request that it additionally load the ``logging_demo::LoggerConfig`` component into the container.

As an example, if you want to debug the ``composition::Talker`` demo, you can start the talker as normal with:

Shell 1:

.. code-block:: bash

   ros2 run rclcpp_components component_container

Shell 2:

.. code-block:: bash

   ros2 component load /ComponentManager composition composition::Talker

And then when you want to enable debug logging, load the ``LoggerConfig`` component with:

Shell 2

.. code-block:: bash

   ros2 component load /ComponentManager logging_demo logging_demo::LoggerConfig

And finally, configure all unset loggers to the debug severity by addressing the empty-named logger.
Note that loggers that have been specifically configured to use a particular severity will not be affected by this call.

Shell 2:

.. code-block:: bash

   ros2 service call /config_logger logging_demo/srv/ConfigLogger "{logger_name: '', level: DEBUG}"

You should see debug output from any previously unset loggers in the process start to appear, including from the ROS 2 core.

Logger level configuration: command line
----------------------------------------

As of the Bouncy ROS 2 release, the severity level for loggers that have not had their severity set explicitly can be configured from the command line.
Restart the demo including the following command line argument:

.. code-block:: bash

   ros2 run logging_demo logging_demo_main __log_level:=debug

This configures the default severity for any unset logger to the debug severity level.
You should see debug output from loggers from the demo itself and from the ROS 2 core.

The ability to configure specific loggers from the command line is forthcoming.

Console output formatting
^^^^^^^^^^^^^^^^^^^^^^^^^

If you would like more or less verbose formatting, you can use `the RCUTILS_CONSOLE_OUTPUT_FORMAT environment variable <logging-console-output-configuration>`.
For example, to additionally get the timestamp and location of the log calls, stop the demo and restart it with the environment variable set:

.. code-block:: bash

   export RCUTILS_CONSOLE_OUTPUT_FORMAT="[{severity} {time}] [{name}]: {message} ({function_name}() at {file_name}:{line_number})"
   # Or, on Windows:
   # set "RCUTILS_CONSOLE_OUTPUT_FORMAT=[{severity} {time}] [{name}]: {message} ({function_name}() at {file_name}:{line_number})"
   ros2 run logging_demo logging_demo_main

You should see the timestamp in seconds and the function name, filename and line number additionally printed with each message.
*The ``time`` option is only supported as of the ROS 2 Bouncy release.*

Console output colorizing
^^^^^^^^^^^^^^^^^^^^^^^^^

By default, the output is colorized when it's targeting a terminal.
If you would like to force enabling or disabling it, you can use the ``RCUTILS_COLORIZED_OUTPUT`` environment variable.
For example:

.. code-block:: bash

   export RCUTILS_COLORIZED_OUTPUT=0  # 1 for forcing it
   # Or, on Windows:
   # set "RCUTILS_COLORIZED_OUTPUT=0"
   ros2 run logging_demo logging_demo_main

You should see that debug, warn, error and fatal logs aren't colorized now.

.. note::

   In Linux and MacOS forcing colorized output means that if you redirect the output to a file, the ansi escape color codes will appear on it.
   In windows the colorization method relies on console APIs.
   If it is forced you will get a new warning saying that colorization failed.
   The default behavior already checks if the output is a console or not, so forcing colorization is not recommended.

Line buffered console output
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default, info and debug log calls aren't line buffered.
You can force it using ``RCUTILS_CONSOLE_STDOUT_LINE_BUFFERED`` environment variable.
For example:

.. code-block:: bash

   export RCUTILS_CONSOLE_STDOUT_LINE_BUFFERED=1
   # Or, on Windows:
   # set "RCUTILS_CONSOLE_STDOUT_LINE_BUFFERED=1"
   ros2 run logging_demo logging_demo_main

The output should look as before.
For details about I/O buffering, see `buffering concepts <https://www.gnu.org/software/libc/manual/html_node/Buffering-Concepts.html>`_.
