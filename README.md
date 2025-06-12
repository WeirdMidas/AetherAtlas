# Perfd opt

The old Project WIPE automatically adjusted interactive parameters based on system load. However, with the arrival of CFS and EAS scheduler, interactive was abandoned by the devs to give full access to schedutil. And as a way to try to optimize these devices even more, Matt Yang created Perfd opt, which focused on saving energy and maintaining fluidity at the same time.

However, perfd ​​opt was full of flaws and inconsistencies with the task management of current devices, such as aggressive ramping and the need for immediate performance that was terrible in profiles like powersave, for example.

However, with current needs arising, with more devices needing more energy efficiency instead of power, like old devices. Perfd opt returned, with a fork made by Weird Midas.

The proposal of this fork is to expand the central idea of ​​Matt Yang's perfd ​​opt, but with very subtle changes. Instead of focusing on responding to demand aggressively and being conservative in idle, Perfd opt is now more efficient in both situations. It now focuses on adjusting the CFS, EAS and even WALT schedulers to be more energy efficient, which instead of being aggressive in terms of performance needs: It focuses on finishing the task quickly and then resting. With this, the module does not favor the underutilization of small cores, etc. In fact, it favors the behavior of a scheduler that takes aspects of EAS and implements them in the CFS and WALT architectures. This means that the scheduler itself is optimized for on-demand, favoring the correct allocation of cores. The module uses everything the device has to finish tasks quickly and with the lowest possible expenditure so that it can rest and spend less.

In general, this means that the module imitates the behavior of the EAS scheduler, favoring a scheduler that can be implemented in all devices, be it CFS, HMP and WALT.

## Features
- Pure CPU optimization and scheduler module, does not contain any placebo and is exclusive to Snapdragon platforms, see if your processor is on the list of compatible SOCs.
- For recent SOCs (from sdm665 onwards) the governor used will be schedutil. For older SOCs (from sdm660 onwards) interactive will be used. Both are optimized for fast and stable response while consuming the least possible power.
- Optimize the scheduler behavior to be more efficient with each SOC architecture. Reserve one or two cores for foreground and top-app (depending on whether the device is a 4x4 or 6x2, etc.), distribute tasks correctly between cores and allow more efficient utilization between CPUs. Favoring more efficient multithreading for energy savings.
- Pinning of threads that handle scrolling on small cores, maximizing energy savings when scrolling and avoiding using big cores for tasks that small cores can handle efficiently.
- Follow a scheduling strategy that fully respects the scheduler. Following a flow like this: Input boost (starts the CPU at a frequency that serves as a "feed" for subsequent tasks) > scheduler (reorders tasks among cores) > governor/schedutil (decides whether to increase or maintain the frequency). Based on this ramping flow, the system responds to almost most tasks with transition latency close to 0.

## Compatible SOCs and profiles

- powersave+: based on powersave, but with additional power saving settings to save as much battery power as possible
- powersave: based on balance mode, but with lower max frequency
- balance: smoother than the stock config with lower power consumption
- performance: dynamic stune boost = 30 with no frequency limitation
- fast: providing stable performance capacity considering the TDP limitation of device chassis

```plain
sdm865 (schedutil)
sdm855/sdm855+ (schedutil)
sdm845 (schedutil)
sdm765/sdm765g (schedutil)
sdm730/sdm730g (schedutil)
sdm680 (schedutil)
sdm675 (schedutil)
sdm710/sdm712 (schedutil)
```

## Requirements

1. Android 8-15
2. Rooted with Magisk or KSU

## Installation

1. Download zip in [Release Page](https://github.com/yc9559/perfd-opt/releases)
2. Flash in Magisk manager
3. Reboot
4. Check whether `/sdcard/Android/panel_powercfg.txt` exists

## Switch modes

### Switching on boot

1. Open `/sdcard/Android/panel_powercfg.txt`
2. Edit line `default_mode=balance`, where `balance` is the default mode applied at boot
3. Reboot

### Switching after boot

Option 1:  
Exec `sh /data/powercfg.sh balance`, where `balance` is the mode you want to switch.  

Option 2:  
Install [vtools](https://www.coolapk.com/apk/com.omarea.vtools) and bind APPs to power mode.  

## Credit

```plain
@屁屁痒
provide /vendor/etc & sched tunables on Snapdragon 845

@林北蓋唱秋
provide /vendor/etc on Snapdragon 675

@酪安小煸
provide /vendor/etc on Snapdragon 710

@沉迷学习日渐膨胀的小学僧
help testing on Snapdragon 855

@NeonXeon
provide information about dynamic stune

@rfigo
provide information about dynamic stune
```
