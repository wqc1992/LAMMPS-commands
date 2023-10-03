# fix temp/rescale command

## Syntax

```
fix ID group-ID temp/rescale N Tstart Tstop window fraction
```

- `N` = perform rescaling every N steps

- `Tstart`, `Tstop` = desired temperature at start/end of run (temperature units)

- `window` = only rescale if temperature is outside this window (temperature units)

- `fraction` = rescale to target temperature by this fraction

## Description

Reset the temperature of a group of atoms by explicitly rescaling their velocities.

The rescaling is applied to only the **translational degrees of freedom** for the particles, which is an important consideration if finite-size particles which have rotational degrees of freedom are being thermostatted with this fix. The translational degrees of freedom can also have a bias velocity removed from them before thermostatting takes place; see the description below.

> Note:
>
> Unlike the `fix nvt` command which performs Nose/Hoover thermostatting **and** time integration, this fix does **not** perform time integration. **It only modifies velocities to effect thermostatting**. Thus you must use a separate time integration fix, like `fix nve` to actually update the positions of atoms using the modified velocities. Likewise, this fix should not normally be used on atoms that also have their temperature controlled by another fix - e.g. by `fix nvt` or `fix langevin` commands.

------

Rescaling is performed every `N` timesteps. The target temperature is a ramped value between the `Tstart` and `Tstop` temperatures at the beginning and end of the run. 

假设 $t_{\rm{start}}$ 和 $t_{\rm{stop}}$ 分别是 `fix temp/rescale` 命令生效和结束的时间, $t$ 和 $T$ 分别是当前时间和温度, 则 `fix temp/rescale` 命令缩放的温度为:
$$
T_{\rm{desired}} =\frac{T_{\rm{stop}}-T_{\rm{start}}}{t_{\rm{stop}}-t_{\rm{start}}} t 
+ \frac{T_{\rm{start}}t_{\rm{stop}} - T_{\rm{stop}}t_{\rm{start}}}{t_{\rm{stop}}-t_{\rm{start}}}
$$

当 $t_{\rm{start}}=0$ 时,
$$
T_{\rm{desired}} = \frac{T_{\rm{stop}}-T_{\rm{start}}}{t_{\rm{stop}}} t + T_{\rm{start}}
$$

> Note:
>
> This thermostat will generate an error if the current temperature is zero at the end of a timestep it is invoked on. It cannot rescale a zero temperature. (即 $T>0$)
>

`Tstart` can be specified as an equal-style `variable`. In this case, the `Tstop` setting is ignored. If the value is a variable, it should be specified as `v_name`, where name is the variable name. In this case, the variable will be evaluated each timestep, and its value used to determine the target temperature.

```
# Example
variable
fix
```

Equal-style variables can specify formulas with various mathematical functions, and include `thermo_style` command keywords for the simulation box parameters and timestep and elapsed time. Thus it is easy to specify a time-dependent temperature.

```
# Example
```

------

Rescaling is only performed if the difference between the current and desired temperatures is greater than the `window` value. `window` 设置了速度缩放的触发条件:
$$
\left | T_{\rm{current}} - T_{\rm{desired}} \right | > \rm{window}
$$
------

The amount of rescaling that is applied is a `fraction` (from 0.0 to 1.0) of the difference between the actual and desired temperature. E.g. if `fraction = 1.0`, the temperature is reset to exactly the desired value. `fraction` 设置了缩放量:
$$
T_{\rm{current}} \gets T_{\rm{current}} + \left( T_{\rm{desired}} - T_{\rm{current}} \right)*\rm{fraction}
$$

------

This fix computes a temperature each timestep. To do this, the fix creates its own compute of style “temp”, as if one of this command had been issued:

```
# 自动创建 compute temp 命令
# Note that the ID of the new compute is the fix-ID + "_temp"
# and the group for the new compute is the same as the fix group
compute fix-ID_temp group-ID temp
```

Note that this is **NOT** the compute used by thermodynamic output with ID = *thermo_temp*. This means you can change the attributes of this fix’s temperature (e.g. its degrees-of-freedom) via the `compute_modify` command or print this temperature during thermodynamic output via the `thermo_style custom` command using the appropriate compute-ID. It also means that changing attributes of *thermo_temp* will have no effect on this fix.

Like other fixes that perform thermostatting, this fix can be used with `compute` commands that remove a "bias" from the atom velocities. E.g. to apply the thermostat only to atoms within a spatial `region`, or to remove the center-of-mass velocity from a group of atoms, or to remove the x-component of velocity from the calculation.

This is not done by default, but only if the `fix_modify` command is used to assign a temperature compute to this fix that includes such a bias term. In this case, the thermostat works in the following manner: bias is removed from each atom, thermostatting is performed on the remaining thermal degrees of freedom, and the bias is added back in.



