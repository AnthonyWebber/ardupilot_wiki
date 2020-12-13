.. _adding-a-new-library-to-Waf:

=========
Adding a new library to Waf
=========

The `waf <https://github.com/ArduPilot/ardupilot/blob/master/BUILD.md>`__ build system makes this very simple.

Assuming that your new library code is in folder 'ardupilot/library/MyLibrary', do one of the following:

- If it's specific to a particular vehicle type, add it here:

- If it's applicable to all vehicles, add it here:


Be sure to to do a new `waf configure` and `waf clean` before building.
