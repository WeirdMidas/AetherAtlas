# Perfd opt
![1000006461](https://github.com/user-attachments/assets/bd9a1384-dde5-4654-a80d-6687fc9e714a)

# Modern Scheduler Optimization with a Focus on Energy Efficiency

The old Project WIPE automatically adjusted interactive parameters based on system load. However, with the arrival of CFS and EAS scheduler, interactive was abandoned by the devs to give full access to schedutil. And as a way to try to optimize these devices even more, Matt Yang created Perfd opt, which focused on saving energy and maintaining fluidity at the same time. However, perfd ​​opt was full of flaws and inconsistencies with the task management of current devices, such as aggressive ramping and the need for immediate performance that was terrible in profiles like powersave, for example. However, with current needs arising, with more devices needing more energy efficiency instead of power, like old devices. Perfd opt returned, with a fork made by Weird Midas.

The proposal of this fork is to expand the central idea of ​​Matt Yang's perfd ​​opt, but with very subtle changes. Instead of focusing on responding to demand aggressively and being conservative in idle, Perfd opt is now more efficient in both situations. It now focuses on adjusting the CFS, EAS and even WALT schedulers to be more energy efficient, which instead of being aggressive in terms of performance needs: It focuses on finishing the task quickly and then resting. With this, the module does not favor the underutilization of small cores, etc. In fact, it favors the behavior of a scheduler that takes aspects of EAS and implements them in the CFS and WALT architectures. This means that the scheduler itself is optimized for on-demand, favoring the correct allocation of cores. The module uses everything the device has to finish tasks quickly and with the lowest possible expenditure so that it can rest and spend less.

In general, this means that the module imitates the behavior of the EAS scheduler, favoring a scheduler that can be implemented in all devices, be it CFS, HMP and WALT.

## Features
- Pure CPU optimization and scheduler module, does not contain any placebo and is exclusive to Snapdragon platforms, see if your processor is on the list of compatible SOCs.
- For recent SOCs (like sdm665 and similar) schedutil is used. For older SOCs (like sdm660 and similar) interactive is used. Both are optimized to improve performance with energy costs reduced as much as possible.
- Optimize the scheduler behavior to be more efficient with each SOC architecture. Reserve one or two cores for foreground and top-app (depending on whether the device is a 4x4 or 6x2, etc.), distribute tasks correctly between cores and allow more efficient utilization between CPUs. Favoring more efficient multithreading for energy savings. But we don't forget to allow the launcher to run on all cores, for the purpose of keeping the UX performance up to date.
- Pinning of threads that handle scrolling on small cores, maximizing energy savings when scrolling and avoiding using big cores for tasks that small cores can handle efficiently.
- Follow a scheduling strategy that fully respects the scheduler. Following a flow like this: Input boost (starts the CPU at a frequency that serves as a "feed" for subsequent tasks) > scheduler (reorders tasks among cores) > governor/schedutil (decides whether to increase or maintain the frequency). Based on this ramping flow, the system responds to almost most tasks with transition latency close to 0.
- Prioritize really light or constant tasks on small cores. But don't underuse big cores! Also those for tasks that only they can execute, such as games and others. Both with the goal of improving the multi-core performance of each cluster in their respective specialty.
- Have an intermediate frequency that schedutil/interactive can use if a high load arrives. If this high load can be satisfied with this frequency, this slightly reduces cold start latency and energy consumption.
- Optimizations are also applied to the minimum and maximum frequencies of the device. With the focus of reducing energy consumption proportionally based on the chosen profile.
- Implement the boost idea, which is a frequency that the processor can go above normal to improve the performance of critical tasks, such as opening apps, while maintaining normal power consumption outside of these tasks.
- Even though the module is not purely for performance, with less total FPS. The module was optimized for maximum stability, this means that even with less FPS in games, the module favors the maximum possible stability, being a worthy trade-off for the user, exchanging raw performance (FPS in games) for FPS stability, UI responsiveness and battery savings.

## Compatible SOCs and profiles

- powersave+: based on powersave, but with additional power saving settings to save as much battery power as possible
- powersave: based on balance mode, but with lower max frequency
- balance: smoother than the stock config with lower power consumption
- performance: without frequency limitation and with frequency sustainability optimizations
- fast: providing stable performance capacity considering the TDP limitation of device chassis

```plain
How it works:
Compatible SOC (Governor that it will use + if it has the boost mechanics available)

List of compatible SOCs:

sdm865 (schedutil + boost available)
- powersave:    1.8+1.6+2.4g, boost 1.8+2.0+2.6g, min 0.3+0.7+1.1
- balance:      1.8+2.0+2.6g, boost 1.8+2.4+2.7g, min 0.7+0.7+1.1
- performance:  1.8+2.4+2.8g, boost 1.8+2.4+2.8g, min 0.7+0.7+1.1
- fast:         1.8+2.0+2.7g, boost 1.8+2.4+2.8g, min 0.7+1.2+1.2

sdm855/sdm855+ (schedutil + boost available)
- powersave:    1.7+1.6+2.4g, boost 1.7+2.0+2.6g, min 0.3+0.7+0.8
- balance:      1.7+2.0+2.6g, boost 1.7+2.4+2.7g, min 0.5+0.7+0.8
- performance:  1.7+2.4+2.8g, boost 1.7+2.4+2.8/2.9g, min 0.5+0.7+0.8
- fast:         1.7+2.0+2.7g, boost 1.7+2.4+2.8/2.9g, min 0.5+1.2+1.2

sdm845 (schedutil + boost available)
- powersave:    1.7+2.0g, boost 1.7+2.4g, min 0.3+0.3
- balance:      1.7+2.4g, boost 1.7+2.7g, min 0.5+0.8
- performance:  1.7+2.8g, boost 1.7+2.8g, min 0.5+0.8
- fast:         1.7+2.4g, boost 1.7+2.8g, min 0.5+1.6

sdm765/sdm765g (schedutil + boost available)
- powersave:    1.8+1.7+2.0g, boost 1.8+2.0+2.2g, min 0.3+0.6+0.8
- balance:      1.8+2.0+2.2g, boost 1.8+2.2+2.3/2.4g, min 0.5+0.6+0.6
- performance:  1.8+2.2+2.3g, boost 1.8+2.2+2.3/2.4g, min 0.5+0.6+0.8
- fast:         1.8+2.0+2.2g, boost 1.8+2.2+2.3/2.4g, min 0.5+1.1+1.4

sdm730/sdm730g (schedutil + boost available)
- powersave:    1.7+1.5g, boost 1.7+1.9g, min 0.3+0.3
- balance:      1.7+1.9g, boost 1.7+2.1g, min 0.5+0.6
- performance:  1.8+2.2g, boost 1.8+2.2g, min 0.5+0.6
- fast:         1.8+1.9g, boost 1.8+2.2g, min 0.5+1.2

sdm680 (schedutil)
- powersave:    2.2+1.8g, min 0.3+0.3
- balance:      2.2+1.8g, min 0.6+0.8
- performance:  2.4+1.9g, min 0.6+0.8
- fast:         2.2+1.8g, min 0.6+1.3

sdm675 (schedutil + boost available)
- powersave:    1.7+1.5g, boost 1.7+1.7g, min 0.3+0.3
- balance:      1.7+1.7g, boost 1.7+1.9g, min 0.5+0.6
- performance:  1.8+2.0g, boost 1.8+2.0g, min 0.5+0.6
- fast:         1.8+1.7g, boost 1.8+2.0g, min 0.5+1.2

sdm710/sdm712 (schedutil + boost available)
- powersave:    1.7+1.8g, boost 1.7+2.0g, min 0.3+0.3
- balance:      1.7+2.0g, boost 1.7+2.2/2.3g, min 0.5+0.6
- performance:  1.7+2.2g, boost 1.7+2.2/2.3g, min 0.5+0.6
- fast:         1.7+2.0g, boost 1.7+2.2/2.3g, min 0.5+1.5
```

## Requirements

1. Android 8-15
2. Rooted with the latest version of Magisk or KSU

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