# Tools for klipper

This are python extras, macros and example config for the
[Klipper 3D printer firmware](https://github.com/Klipper3d/klipper). I
originally created this as macros when converting my Jubilee based
toolchangerfrom RRF and Duet3 to Klipper.

I welcome all contribution!

This is still a work in progress. Treat it as a late alpha to early Beta.

## Features

* **Each Tool is treated as an object and has it's own configuration** -
having configurable coordinates for parking, zoning, tool offset, 
meltzonelength, extruder, fan, etc.
*  **Multiple tools can be grouped in ToolGroup.** -Most configuration can
be inherited from the group if not specified in the tool config section.
*  **Virtual tools** - A tool can be virtual andhave a physical parent,
inheriting all nonspecified configuration from parent, parent group and
then toolgroup. Use case example of an ERCF on a PLA tool,a ERCF on a 
PETG tool, one tool without virtual tools for abrasive and yet another
tool with 3 markers that can switch between 3 markers by rotation.
* **Tools don't need to be extruders/hotends**, can be anything.
* **User defineable macro to Lock / Unlock** - Uses custom gcodes in config 
like the gcode_button. This can call a macro or multiple lines. 
* **User defineable macro to Pickup / Dropoff tools** - Can be inherited.
In the macro, `myself` refers to the calling toolobject so you can get 
myself.id for tool number or myself.offset[0] for X offset in the macro.
Ex. having same pickup gcode macro inherited for all tools from a group 
except for one that uses another type of toolwipe and has it's own pickup_gcode.
* **Global ToolLock parameters** - example purge_on_toolchange can be set 
to false when aligning tools with TAMV/ZTATP.
* **Fan speed** is carried over on toolchange if the tool has a fan. Also
`M106`/`M107` defaults to current_tool to set fan speed but can also use a Pnnn 
parameter to specify another tool.
* **Extruder temperature control** - 
  - Definable for any tool with a Pnnn parameter or defaults to current_tool
  - Has diffrent Active and Standby temperatures.
  - Can be set in Active, Standby or Off mode.
  - Can have delayed Standby and Off. Configured and/or customized with N and O parameters at runtime.
    - Example. Set Standby temperature 30 sec after putting the tool in standby and in Off after 30 minutes of not being activated.
    - Or set to Time to standby to 0.1 for instant standby and Time to Powerdown to 604800 for a having it powered for a week.
    - Usefull when having sporadic toolchanges in a large print.
  - Wait to reach temperature with tolerance. Set temperature +/- configurable tolerance.

## To do:
* Save pressure avance per tool
* Change virtual tools code.
* Mean Layer time Standby mode. - Save time at every layerchange and at toolchange set to mean time of last 3 layers *2 or at last layer *1.5 with a Maximum and a minimum time. Needs to be analyzed further.
* Implement Fan Scale. Can change fan scale for diffrent materials or tools from slicer at toolchange. Maybe max and min too?

## G-Code commands:
* `TOOL_LOCK` - Lock command
* `TOOL_UNLOCK` - Unlock command
* `T_1` - Unload and park all tools.
* `SET_AND_SAVE_FAN_SPEED` - Set the fan speed of selected or current tool if no `TOOL` supplied. Then save to be recovered at ToolChange.
  * `S` - Fan speed 0-255 or 0-1, default is 1, full speed.
  * `P` - Fan of this tool. Default current tool.
* `TEMPERATURE_WAIT_WITH_TOLERANCE` - Waits for all temperatures, or a specified tool or heater's temperature.
This command can be used without any additional parameters. Without parameters it waits for bed and current extruder. Only one of either TOOL or HEATER may be used.
  - `TOOL` Tool number.
  - `HEATER` Heater number. 0="heater_bed", 1="extruder", 2="extruder1", 3="extruder2", etc. Only works if named as default, this way.
  - `TOLERANCE` Tolerance in degC. Defaults to 1*C. Wait will wait until heater is between set temperature +/- tolerance.
* `SET_TOOL_TEMPERATURE` - Set tool temperature.
  * `TOOL`= Tool number, optional. If this parameter is not provided, the current tool is used.
  * `STDB_TMP`= Standby temperature(s), optional
  * `ACTV_TMP`= Active temperature(s), optional
  * `CHNG_STATE` = Change Heater State, optional: 0 = off, 1 = standby temperature(s), 2 = active temperature(s).
  * `STDB_TIMEOUT` = Time in seconds to wait between changing heater state to standby and setting heater target temperature to standby temperature when standby temperature is lower than tool temperature.
    * Use for example 0.1 to change immediately to standby temperature.
  * SHTDWN_TIMEOUT = Time in seconds to wait from docking tool to shutting off the heater, optional.
    * Use for example 86400 to wait 24h if you want to disable shutdown timer.
* `Tn` - T0, T1, T2, etc... A select command is created for each tool.

## Values accesible from Macro for each object
- **Toollock**
  - `tool_current`
  - `saved_fan_speed`
  - `purge_on_toolchange`
- **Tool**
  - `name`
  - `is_virtual`
  - `physical_parent_id`
  - `extruder`
  - `fan`
  - `lazy_home_when_parking`
  - `meltzonelength`
  - `zone`
  - `park`
  - `offset`
  - `heater_state`
  - `heater_active_temp`
  - `heater_standby_temp`
  - `placeholder_standby_temp`
  - `idle_to_standby_time`
  - `idle_to_powerdown_time`
- **ToolGroup**
  - `is_virtual`
  - `physical_parent_id`
  - `lazy_home_when_parking`
- **ToolStandbyTempTimer**
  - `tool`
  - `temp_type`
  - `duration`
