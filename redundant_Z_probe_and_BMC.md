# The Problem
Most people using the klicky probe and adaptive bed mesh calibrate will run into the following:

- ATTACH PROBE (KLICK!)
- PROBE Z
- DETACH PROBE (KLICK!) (REDUNDANT!)
- ATTACH PROBE (KLICK!) (REDUNDANT!)
- BED_MESH_CALIBRATE
- DETACH PROBE (KLICK!)

See the problem? There was a redundant DETACH and ATTACH probe in the middle.
This happens because the default Klicky macro overrides the homing and forces the attach and detach. The BED_MESH_CALIBRATE also forces an attach and detatch. Because we call these two commands separately, we get redundant probe attach+detatch.

# The Workaround
1. Let the BED_MESH_CALIBRATE be handled by the Z_Probe.
2. Obviously you don't want to run this every time you probe so we create a macro that enables the BED_MESH_CALIBRATE feature during the Z_Probe.

What are the consequences of this? I don't know yet. But it definitely works.


# The Solution:
1. In [klicky-macros.cfg], insert the following snippet after ``_Home_Z_``
```
# START INSERT - DELTA'S BED_MESH_CALIBRATE Fix
{% set bmc_on_z_probe = printer["gcode_macro _bed_mesh_calibrate_Variables"].bmc_on_z_probe %}
{% if bmc_on_z_probe %}
    _KLICKY_STATUS_MESHING ; This part is completely optional
    _BED_MESH_CALIBRATE ADAPTIVE=1
    SET_GCODE_VARIABLE MACRO=_bed_mesh_calibrate_Variables VARIABLE=bmc_on_z_probe VALUE={ false }
{% endif %}
# END INSERT
```

2. In [klicky-bed-mesh-calibrate.cfg], insert the following snippet at the end:
```
[gcode_macro _bed_mesh_calibrate_Variables]
    variable_bmc_on_z_probe:            0
    gcode:

[gcode_macro HOME_Z_AND_BED_MESH_CALIBRATE]
description: Enables a variable to automatically z_probe and bed mesh calibrate at the same time. This is a useful workaround to skip having to individually attach and detach for a z_probe and BED_MESH_CALIBRATE.
gcode:
    {% set enable = params.ENABLE|default(0)|int %}
    M118 HOME_Z_AND_BED_MESH_CALIBRATE set to { enable }
    SET_GCODE_VARIABLE MACRO=_bed_mesh_calibrate_Variables VARIABLE=bmc_on_z_probe VALUE={ enable }
```

3. In your slicer's Machine Start GCODE, remove your old ``BED_MESH_CALIBRATE`` and include
``HOME_Z_AND_BED_MESH_CALIBRATE ENABLE=1`` BEFORE your G28 (Home All Axis)
```
HOME_Z_AND_BED_MESH_CALIBRATE ENABLE=1 ; Calibrate Bed Mesh
G28             ; home all axis
```

Now, your probe Z probe will do:
- ATTACH PROBE (KLICK!)
- PROBE Z
- BED_MESH_CALIBRATE
- DETACH PROBE (KLICK!)
