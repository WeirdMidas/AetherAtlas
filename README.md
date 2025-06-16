# Perfd opt
![1000006461](https://github.com/user-attachments/assets/bd9a1384-dde5-4654-a80d-6687fc9e714a)

# Modern Scheduler Optimization with a Focus on Energy Efficiency

The old Project WIPE automatically adjusted interactive parameters based on system load. However, with the arrival of CFS and EAS scheduler, interactive was abandoned by the devs to give full access to schedutil. And as a way to try to optimize these devices even more, Matt Yang created Perfd opt, which focused on saving energy and maintaining fluidity at the same time. However, perfd ​​opt was full of flaws and inconsistencies with the task management of current devices, such as aggressive ramping and the need for immediate performance that was terrible in profiles like powersave, for example. However, with current needs arising, with more devices needing more energy efficiency instead of power, like old devices. Perfd opt returned, with a fork made by Weird Midas.

The proposal of this fork is to expand the central idea of ​​Matt Yang's perfd ​​opt, but with very subtle changes. Instead of focusing on responding to demand aggressively and being conservative in idle, Perfd opt is now more efficient in both situations. It now focuses on adjusting the CFS, EAS and even WALT schedulers to be more energy efficient, which instead of being aggressive in terms of performance needs: It focuses on finishing the task quickly and then resting. With this, the module does not favor the underutilization of small cores, etc. In fact, it favors the behavior of a scheduler that takes aspects of EAS and implements them in the CFS and WALT architectures. This means that the scheduler itself is optimized for on-demand, favoring the correct allocation of cores. The module uses everything the device has to finish tasks quickly and with the lowest possible expenditure so that it can rest and spend less.

In general, this means that the module imitates the behavior of the EAS scheduler, favoring a scheduler that can be implemented in all devices, be it CFS, HMP and WALT.

## Features

- Pure CPU optimization and scheduler module, does not contain any placebo and is exclusive to Snapdragon platforms, see if your processor is on the list of compatible SOCs. The objective of the module is to make the behavior of the device scheduler used closer to the dynamic workload of Android, adapting to the system's performance needs with reduced energy costs as the system's demand is satisfied.
- Choose the governor that best suits the architecture and era of the device. Schedutil for the newer ones and Interactive for the older ones, and of course, optimize them for less throttling and energy consumption with the same or better performance, allowing you to significantly increase battery life.
- Implemented the Project WIPE! For devices with interactive. A font created by Matt Yang that proves to be extremely competent in terms of optimizing Interactive from start to finish.
- Prioritize a scheduling method that respects the way Android works. Having the style of: input (feed subsequent tasks) > scheduler (with the energy that the input designed it for, organizes and reorganizes tasks quickly) > governor (maintains or increases the frequency), reducing the task start power noticeably. In addition, there are some extras in this scheduling that are explained below:
  - If the user has uclamp or schedtune. A scalable style is favored for foreground and top-app tasks, allowing top-app and foreground tasks to take maximum advantage of the cores, improving multithreading performance. And still without distractions like background, for example.
  - Have an intermediate frequency, which will be used as a frequency that will try to satisfy the performance needs of the system if it is sufficient.
  - Know the limits of the small cores of each SOC. Use the small cores differently for each SOC. Favoring a better balance instead of a standardization that may not exploit the device architecture.
- Use the CPUset more intelligently and respectfully with Android's dynamic workload. Reserve one (or two if the device is a 6x2) big cores for top-app and foreground tasks, along with reserving two small cores for background and top-app tasks. Based on this, fix the launcher&home on all cores and make the foreground use the small cores when the user is scrolling and only in these situations, in addition to allowing the foreground to use the big cores if the user is doing other tasks. Allowing multithreading focused more on efficiency than raw performance, saving energy when using messenger or social media apps and maintaining foreground performance in apps that need it.
- Equip different SOCs with different boosts. Favoring compatibility between different devices with different performance needs, if necessary more performance in certain situations (such as opening apps, frame stability, etc.).
- Even though the module is not purely for performance, with less total FPS. The module was optimized for maximum stability, this means that even with less FPS in games, the module favors the maximum possible stability, being a worthy trade-off for the user, exchanging raw performance (FPS in games) for FPS stability, UI responsiveness and battery savings.

## Compatible SOCs and profiles

- powersave: based on balance mode, but with lower max frequency
- balance: smoother than the stock config with lower power consumption
- performance: without frequency limitation and with frequency sustainability optimizations
- fast: providing stable performance capacity considering the TDP limitation of device chassis

```plain
How it works:
Compatible SOC (Governor that it will use + if it has the boost mechanics available)
Profiles = Profiles such as powersave, balance, performance and fast will have their respective minimum and maximum frequencies, in addition to, of course, the frequency of the "boost" value if supported
Run Freq = Frequency at which the CPU will immediately jump to the input, being a quick run to allow the processor to follow the flow of input > scheduler > governor
Intel Freq = Intermediate frequency below the two maximum frequency steps, favors energy consumption by allowing the system to satisfy the performance needs in high load situations with a slightly lower frequency
Equipped = Means which additional boosts it comes with, which are the following:
- LB (Launch Boost): Launch Boost, used to start apps by giving them an initial boost when opening
- DP (Disable Packing): Disable packing, spreads threads when starting apps, further reducing startup time
- LBS (Launch Boost Sustained): Maintains the performance gained by previous launches to maintain fixed frequencies
- LBR (Launch Boost Resume): Resumes an app that is in RAM (such as in the recents tab), reducing possible errors such as the app flashing after returning from the recents tab
- ALB (Activity Lauch Boost): Boost in the startup of apps that are "cold". Favoring cold start.
- FS (FPS Stability): A boost that improves frame rate stability in games that use multiple frame rates. Only used on processors capable of handling multiple refresh rates (such as 60 and 90).

List of compatible SOCs:

sdm865 (schedutil + boost available)
- powersave:    1.8+1.6+2.4g, boost 1.8+2.0+2.6g, min 0.3+0.7+1.1
- balance:      1.8+2.0+2.6g, boost 1.8+2.4+2.7g, min 0.7+0.7+1.1
- performance:  1.8+2.4+2.8g, boost 1.8+2.4+2.8g, min 0.7+0.7+1.1
- fast:         1.8+2.0+2.7g, boost 1.8+2.4+2.8g, min 0.7+1.2+1.2
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with LB, LBS and LBR

sdm855/sdm855+ (schedutil + boost available)
- powersave:    1.7+1.6+2.4g, boost 1.7+2.0+2.6g, min 0.3+0.7+0.8
- balance:      1.7+2.0+2.6g, boost 1.7+2.4+2.7g, min 0.5+0.7+0.8
- performance:  1.7+2.4+2.8g, boost 1.7+2.4+2.8/2.9g, min 0.5+0.7+0.8
- fast:         1.7+2.0+2.7g, boost 1.7+2.4+2.8/2.9g, min 0.5+1.2+1.2
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with LB, LBS and LBR

sdm845 (schedutil + boost available)
- powersave:    1.7+2.0g, boost 1.7+2.4g, min 0.3+0.3
- balance:      1.7+2.4g, boost 1.7+2.7g, min 0.5+0.8
- performance:  1.7+2.8g, boost 1.7+2.8g, min 0.5+0.8
- fast:         1.7+2.4g, boost 1.7+2.8g, min 0.5+1.6
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with LB, LBS and LBR

sdm765/sdm765g (schedutil + boost available)
- powersave:    1.8+1.7+2.0g, boost 1.8+2.0+2.2g, min 0.3+0.6+0.8
- balance:      1.8+2.0+2.2g, boost 1.8+2.2+2.3/2.4g, min 0.5+0.6+0.6
- performance:  1.8+2.2+2.3g, boost 1.8+2.2+2.3/2.4g, min 0.5+0.6+0.8
- fast:         1.8+2.0+2.2g, boost 1.8+2.2+2.3/2.4g, min 0.5+1.1+1.4
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with LB, LBS and LBR

sdm730/sdm730g (schedutil + boost available)
- powersave:    1.7+1.5g, boost 1.7+1.9g, min 0.3+0.3
- balance:      1.7+1.9g, boost 1.7+2.1g, min 0.5+0.6
- performance:  1.8+2.2g, boost 1.8+2.2g, min 0.5+0.6
- fast:         1.8+1.9g, boost 1.8+2.2g, min 0.5+1.2
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with LB, LBS and LBR

sdm710/sdm712 (schedutil + boost available)
- powersave:    1.7+1.8g, boost 1.7+2.0g, min 0.3+0.3
- balance:      1.7+2.0g, boost 1.7+2.2/2.3g, min 0.5+0.6
- performance:  1.7+2.2g, boost 1.7+2.2/2.3g, min 0.5+0.6
- fast:         1.7+2.0g, boost 1.7+2.2/2.3g, min 0.5+1.5
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with LB, LBS and LBR

sdm695 (schedutil)
- It is still being ported and compatibility is being planned.

sdm680 (schedutil + boost available)
- powersave:    2.2+1.8g, boost 2.4+1.9g, min 0.3+0.3
- balance:      2.2+1.8g, boost 2.4+1.9g, min 0.6+0.8
- performance:  2.4+1.9g, boost 2.4+1.9g, min 0.6+0.8
- fast:         2.2+1.8g, boost 2.4+1.9g, min 0.6+1.3
- run freq: 1.4+1.6g
- intel freq: 1.6g+2.0g
- Equipped with LB, DP, LBS, LBR and FS

sdm675 (schedutil + boost available)
- powersave:    1.7+1.5g, boost 1.7+1.7g, min 0.3+0.3
- balance:      1.7+1.7g, boost 1.7+1.9g, min 0.5+0.6
- performance:  1.8+2.0g, boost 1.8+2.0g, min 0.5+0.6
- fast:         1.8+1.7g, boost 1.8+2.0g, min 0.5+1.2
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with LB, LBS and LBR

sdm665 (schedutil)
- It is still being ported and compatibility is being planned.

sdm660 (interactive)
- It is still being ported and compatibility is being planned.
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

@yinwanxi
libcgroup script as a tool to optimize threads separately
```
