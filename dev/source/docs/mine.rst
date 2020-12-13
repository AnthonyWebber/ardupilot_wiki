.. _adding-a-new-library-to-Waf:

=========
Adding a new library to Waf
=========

The `waf <https://github.com/ArduPilot/ardupilot/blob/master/BUILD.md>`__ build system makes this very simple.

Assuming that your new library code is in folder 'ardupilot/libraries/MyLibrary', do one of the following:

- If it's specific to a particular vehicle type, add it here:

::

def build(bld):
    vehicle = bld.path.name
    bld.ap_stlib(
        name=vehicle + '_libs',
        ap_vehicle=vehicle,
        ap_libraries=bld.ap_common_vehicle_libraries() + [
            'MyLibrary',
            'AC_AttitudeControl',
            'AC_InputManager',
            'AC_PrecLand',
            ...
            

- If it's applicable to all vehicles, add it here:

Be sure to ``waf configure`` and ``waf clean`` before building afresh.
