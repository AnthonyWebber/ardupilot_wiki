.. _adding-a-new-library-to-Waf:

=========
Adding a new library to Waf
=========

The `waf <https://github.com/ArduPilot/ardupilot/blob/master/BUILD.md>`__ build system makes this very simple.

Assuming that your new library code is in folder 'ardupilot/library/MyLibrary', do one of the following:

- If it's specific to a particular vehicle type, add it here:

- If it's applicable to all vehicles, add it here:



The threading approach in ArduPilot depends on the board it is built
for. Some boards (such as the APM1 and APM2) don't support threads, so
make do with a simple timer and callbacks. Some boards (PX4 and Linux)
support a rich Posix threading model with realtime priorities, and these
are used extensively by ArduPilot.

There are a number of key concepts related to threading that you need to
understand in ArduPilot:

-  The timer callbacks
-  HAL specific threads
-  driver specific threads
-  ardupilot drivers versus platform drivers
-  platform specific threads and tasks
-  the AP_Scheduler system
-  semaphores
-  lockless data structures

The timer callbacks
===================

Every platform provides a 1kHz timer in the AP_HAL. Any code in
ArduPilot can register a timer function which is then called at 1kHz.
All registered timer functions are called sequentially. This very
primitive mechanism is used as it is extremely portable, and yet very
useful. You register a timer callback by calling the
hal.scheduler->register_timer_process() like this:

::

      hal.scheduler->register_timer_process(AP_HAL_MEMBERPROC(&AP_Baro_MS5611::_update));

that particular example is from the MS5611 barometer driver. The
AP_HAL_MEMBERPROC() macro provides a way to encapsulate a C++ member
function as a callback argument (bundling up the object context with the
function pointer).

When a piece of code wants something to happen at less than 1kHz then it
should maintain its own "last_called" variable and return immediately
if not enough time has passed. You can use the hal.scheduler->millis()
and hal.scheduler->micros() functions to get the time since boot in
milliseconds and microseconds to support this.

You should now go and modify an existing example sketch (or create a new
one) and add a timer callback. Make the timer increment a counter then
print the value of the counter every second in the loop() function.
Modify your function so that it increments the counter  every 25
milliseconds.

HAL specific threads
====================

On platforms that support real threads the AP_HAL for that platform
will create a number of threads to support basic operations. For
example, on Pixhawk the following HAL specific threads are created:

-  The UART thread, for reading and writing UARTs (and USB)
-  The timer thread, which supports the 1kHz timer functionality
   described above
-  The IO thread, which supports writing to the microSD card, EEPROM and
   FRAM

Have a look in Scheduler.cpp inside each AP_HAL implementation to see
what threads are created and what the realtime priority of each thread
is.

If you have a Pixhawk then you should also now setup a debug console
cable and attach to the nsh console (the serial5 port). Connect at
57600. When you have connected, try the "ps" command ad you will get
something like this:

::

    PID PRI SCHD TYPE NP STATE NAME
     0 0 FIFO TASK READY Idle Task()
     1 192 FIFO KTHREAD WAITSIG hpwork()
     2 50 FIFO KTHREAD WAITSIG lpwork()
     3 100 FIFO TASK RUNNING init()
     37 180 FIFO TASK WAITSEM AHRS_Test()
     38 181 FIFO PTHREAD WAITSEM <pthread>(20005400)
     39 60 FIFO PTHREAD READY <pthread>(20005400)
     40 59 FIFO PTHREAD WAITSEM <pthread>(20005400)
     10 240 FIFO TASK WAITSEM px4io()
     13 100 FIFO TASK WAITSEM fmuservo()
     30 240 FIFO TASK WAITSEM uavcan()

In this example you can see the "AHRS_Test" thread, which is running
the example sketch from libraries/AP_AHRS/examples/AHRS_Test. You can
