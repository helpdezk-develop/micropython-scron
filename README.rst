.. role:: bash(code)
   :language: bash

.. role:: python(code)
   :language: python

***************************
Simple CRON for MicroPython
***************************

SimpleCRON is a time-based task scheduling program inspired by the well-known
CRON program for Unix systems.

Before you use this library, you may be interested in a similar MicroCRON_ library that needs 3 times less memory
to operate. It was written based on experience with the current library.

The software was tested under micropython 1.10 (esp32, esp8266) and python 3.5.

Note: The library does not work properly under the  `Loboris MicroPython <https://github.com/loboris/MicroPython_ESP32_psRAM_LoBo/wiki>`_ port,
read this issue `#3 <https://github.com/fizista/micropython-scron/issues/3>`_.


`Project documentation. <https://fizista.github.io/micropython-scron/html/index.html>`_

What you can do with this library:
##################################
* Run any task at precisely defined intervals
* Delete and add tasks while the program is running.
* Run the task a certain number of times and many more.

Requirements:
#############
* The board on which the micropython is installed(v1.10)
* The board must have support for hardware timers.


Install
#######
You can install using the upip:

.. code-block:: python

    import upip
    upip.install("micropython-scron")

or

.. code-block:: bash

    micropython -m upip install -p modules micropython-scron


You can also clone this repository, and install it manually:

.. code-block:: bash

    git clone https://github.com/fizista/micropython-scron.git
    cd ./micropython-scron
    ./flash-src.sh




ESP8266
*******

The library on this processor must be compiled into binary code.

The MicroPython cross compiler is needed for this.

If you already have the mpy-cross command available, then run the bash script:

.. code-block:: bash

    ./compile.sh

and then upload the library to the device, e.g. using the following script:

.. code-block:: bash

    ./flash-byte.sh

**Important!** An even better solution is to integrate this module into the firmware.
The module takes up much less RAM.

To do this, copy the :python:`scron` module to :bash:`micropython/ports/esp8266/modules`.
Then compile the sources, and upload them to your device.

Simple examples
###############

**Important!** You have to add the following lines of code if your application does not periodically run more actions
than the maximum timer time. Calls should be more often than about 5 minutes.

.. code-block:: python

    simple_cron.add('null', lambda *a, **k: None, seconds=0, minutes=range(0,59,5), removable=False)

Simple code to run every second:

.. code-block:: python

    from scron.week import simple_cron
    # OR
    # SimpleCRON single-class library, with minimal recursion depth.
    # This is mainly for ESP8266, because we have very few possible recursions.
    from scron.cweek import simple_cron
    # Depending on the device, you need to add a task that
    # will be started at intervals shorter than the longest
    # time the timer can count.
    simple_cron.add('null', lambda *a, **k: None, seconds=0, minutes=range(0,59,5), removable=False)
    simple_cron.add('helloID', lambda *a,**k: print('hello'))
    simple_cron.run()


Code, which is activated once a Sunday at 12:00.00:

.. code-block:: python

    simple_cron.add(
        'Sunday12.00',
        lambda *a,**k: print('wake-up call'),
        weekdays=6,
        hours=12,
        minutes=0,
        seconds=0
    )


Every second minute:

.. code-block:: python

    simple_cron.add(
        'Every second minute',
        lambda *a,**k: print('second call'),
        minutes=range(0, 59, 2),
        seconds=0
    )


Other usage samples can be found in the 'examples' directory.

How to use it
#############

Somewhere in your code you have to add the following code,
and from then on SimpleCRON is ready to use.

.. code-block:: python

    from scron.week import simple_cron
    simple_cron.run() # You have to run it once. This initiates the SimpleCRON action,
                      # and reserve one timmer.



To add a task you are using:

.. code-block:: python

    simple_cron.add(<callback_id_string>, <callback>, ...)


Callbacks
#########

Example of a callback:

.. code-block:: python

    def some_counter(scorn_instance, callback_name, pointer, memory):
        if 'counter' in memory:
            memory['counter'] += 1
        else:
            memory['counter'] = 1


where:

* :python:`scorn_instance` - SimpleCRON instance, in this case scron.weekend.simple_cron
* :python:`callback_name` - Callback ID
* :python:`pointer` - This is an indicator of the time in which the task was to be run.
  Example: (6, 13, 5, 10).  This is **(** Sunday **,** 1 p.m. **,** minutes 5 **,** seconds 10 **)**
* :python:`memory` - Shared memory for this particular callback, between all calls.
  By default this is a dictionary.

If for some reason a running callback throws an exception,
then it is possible to process this event with special functions.
The default exception processing function is print().

To add new processing functions for callback exceptions, simply add them to the list below:

.. code-block:: python

    simple_cron.callback_exception_processors(processor_function)


where:

    processor_function is function(exception_instance)

Important notes:
################

* If a task takes a very long time, it blocks the execution of other tasks!
* If there are several functions to run at a given time, then they are
  started without a specific order.
* If the time has been changed (time synchronization with the network,
  own time change), run the :python:`simple_cron._sync_time()` function,
  which will set a specific point in time. Without this setting,
  it may happen that some callbacks will not be started.


What has not been tested:
#########################

* SimpleCRON operation during sleep

How to test
###########

First install the following things:

.. code-block:: bash

    git clone https://github.com/fizista/micropython-scron.git
    cd micropython-scron/
    micropython -m upip install -p modules micropython-unittest
    micropython -m upip install -p modules micropython-time


Then run the tests:

.. code-block:: bash

    ./run_tests.sh

Testing directly on the device:

.. code-block:: bash

    pip install pyserial adafruit-ampy

    ./run_tests_esp.sh upload_combined
    ./run_tests_esp.sh run /dev/ttyUSB0

    ./run_tests_esp.sh upload
    ./run_tests_esp.sh run /dev/ttyUSB0


*******************
Support and license
*******************

If you have found a mistake or other problem, write in the issues.

If you need a different license for this library (e.g. commercial),
please contact me: fizista+scron@gmail.com.


.. _MicroCRON: https://github.com/fizista/micropython-mcron