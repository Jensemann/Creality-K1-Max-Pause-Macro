# Creality-K1-Max-Pause-Macro
Implementation of M0 / PAUSE / RESUME macros für Creality K1 Max

## Purpose
this repository will describe, how to implement the required macros for the Creality Ka / K1 Max printers, to be able to PAUSE and RESUME a print !

## Requirements
- A Creality K1 / K1 Max
- Firmware with Root enabled. If you dont knoiw how to do it, please check This site was built using [Installation Helper Script for Creality K1 Series](https://github.com/Guilouz/Creality-K1-and-K1-Max).
- Mainsail installed. It might work with Fluidd, however, i did it with Mainsail.

## Installation

Access your printer's [web interface](https://github.com/Guilouz/Creality-K1-and-K1-Max/wiki/Access-to-Web-Interface)
To access to the original Mainsail Web Interface, just use your printer's IP address with port 4409 in your Web browser such as: http://xxx.xxx.xxx.xxx:4409/ (replacing xxx.xxx.xxx.xxx by your local IP address).
Go to the very end of your printer.cfg.

Find this section:
...
#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
#*# [input_shaper]
#*# shaper_type_y = zv
#*# shaper_freq_y = 43.2
#*# shaper_type_x = zv
#*# shaper_freq_x = 47.4
#*#
...

add the following code BEFORE the section 

MACRO code:
```
### ------------------------------------------ PAUSE START -----------------------------------

[gcode_macro M0]
gcode:
  PAUSE


[gcode_macro RESUME_NOW]
gcode:
  RESUME

[gcode_macro PAUSE_NOW]
gcode:
  PAUSE


[idle_timeout] 
timeout: 120
gcode:
    {% if printer.pause_resume.is_paused %}
        M118 Bypassed Timeout
        M117 Bypassed Timeout
    {% else %}
        M118 Timeout Reached - Heaters and Motors Still On!
        M117 Timeout Reached - Heaters and Motors Still On!
        #TURN_OFF_HEATERS
        #M84
    {% endif %}


[gcode_macro PAUSE]
description: Pause the actual running print
rename_existing: PAUSE_BASE
# change this if you need more or less extrusion
variable_extrude: 1.0
gcode:
  ##### read E from pause macro #####
  {% set E = printer["gcode_macro PAUSE"].extrude|float %}
  ##### set park positon for x and y #####
  # default is your max posion from your printer.cfg
  {% set x_park = printer.toolhead.axis_maximum.x|float - 5.0 %}
  {% set y_park = printer.toolhead.axis_maximum.y|float - 5.0 %}
  ##### calculate save lift position #####
  {% set max_z = printer.toolhead.axis_maximum.z|float %}
  {% set act_z = printer.toolhead.position.z|float %}
  {% if act_z < (max_z - 2.0) %}
      {% set z_safe = 2.0 %}
  {% else %}
      {% set z_safe = max_z - act_z %}
  {% endif %}
  ##### end of definitions #####
  PAUSE_BASE
  G91
  {% if printer.extruder.can_extrude|lower == 'true' %}
    G1 E-{E} F2100
  {% else %}
    {action_respond_info("Extruder not hot enough")}
  {% endif %}
  {% if "xyz" in printer.toolhead.homed_axes %}
    G1 Z{z_safe} F900
    G90
    G1 X{x_park} Y{y_park} F6000
  {% else %}
    {action_respond_info("Printer not homed")}
  {% endif %} 

[gcode_macro RESUME]
description: Resume the actual running print
rename_existing: RESUME_BASE
gcode:
  ##### read E from pause macro #####
  {% set E = printer["gcode_macro PAUSE"].extrude|float %}
  #### get VELOCITY parameter if specified ####
  {% if 'VELOCITY' in params|upper %}
    {% set get_params = ('VELOCITY=' + params.VELOCITY)  %}
  {%else %}
    {% set get_params = "" %}
  {% endif %}
  ##### end of definitions #####
  {% if printer.extruder.can_extrude|lower == 'true' %}
    G91
    G1 E{E} F2100
  {% else %}
    {action_respond_info("Extruder not hot enough")}
  {% endif %}  
  RESUME_BASE {get_params}
```


