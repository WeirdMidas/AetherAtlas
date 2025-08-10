# Aether Atlas
![1000002543](https://github.com/user-attachments/assets/9516ac1e-648b-4896-8523-cdc2bc67e519)

# Review

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, when the HMP scheduler was removed from mainline and in sequence the interactive was also removed. The new scheduler was integrated into recent Android devices (from 2020 onwards) and replaced the mainstream: EAS (Energy-Aware Scheduling), a scheduler that focuses on energy efficiency rather than raw performance, abandoning very old solutions like CFS (Completely Fair Scheduler) and even HMP (Heterogeneous Multi-Processing). But that's not all; trackers like WALT (Window-Assisted Load Tracking) and PELT (Per-Entity Load Tracking) were also integrated, allowing load tracking to be as stable or as fast as possible, depending on the device used. The way to manage the task of each scheduler and tracker became complicated, causing many users to have to specialize in each scheduler to know when and where to optimize to reduce energy consumption, without penalty of performance losses or latency. Causing information to be lost and everything to be obfuscated, making it difficult for users who would like to do this to optimize. This caused even schedutil to be affected, reducing the accuracy of tests because schedutil often works differently in each scheduler.

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. Based on this old project, Perfd opt tries to find a way to, in turn: make the EAS and the device's Tracker more "united", this means that the WALT, or even the PELT, can synchronize better with the EAS, allowing them to track loads, help the EAS to perform better, both energetically and in high performance, allowing the Scheduler and the device's Tracker to work together, improving the consistency of the WALT and the response of the PELT. Where the Scheduler and Tracker will follow the "**Opportunistic** Rice-to-idle" strategy, this means that the way the Scheduler and Tracker work is completely opportunistic with the bias in saving energy by having an immediate "to-idle" entry if the demand ends, of course, based on the performance demand, a more aggressive ramping (or better, "rice") is preferred, always looking for efficient frequencies that can feed the need for immediate performance of the situation. However, as the power profile currently being used becomes closer to high performance, more preference is given to powerful cores, and then, upon completion of the interaction, the device runs directly to idle, using much lower frequencies compared to the standard ones. 

See details of the original project created by Matt Yang [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features

- A CPU/GPU, DevFreq and Scheduler optimization module. It's placebo-free and focuses entirely on improving the Android's dynamic behavior.
- Integrates the Tracker optimization called Rice-to-idle (Race to idle whenever possible). This is a way to optimize WALT/PELT that allows the CPU to scale frequencies according to immediate performance needs, always preferring to resolve the task quickly even if this requires a higher frequency. Based on this, a touch of "opportunism" was also introduced, where the CPU scales as needed but always one step back, avoiding unnecessarily high frequencies.
- Integrates the EAS scheduler optimization called "EAO" (Energy-Aware Opportunism). This is a way to optimize the EAS scheduler and the schedutil/interactive governor so that they prefer more energy-efficient placement, frequencies, and scheduling even in high-performance profiles. Always preferring to save energy by already knowing the immediate performance needs.
- Improve and optimize the behavior of subsystems that directly impact the user experience, such as audio, encoder, and others. This allows you to significantly reduce the power consumption of these subsystems, improving the user experience.

## Profiles

- powersave: based on balance mode, but with more aggressive entry to idle and bias more preferential to small cores
- balance: smoother than the stock config with lower power consumption
- performance: without limitations, seeks maximum performance to the detriment of an efficient "to-idle"
- fast: providing stable performance capacity considering the TDP limitation of device chassis

```plain
sdm865/870 (Schedutil)
- powersave:    min 1.3+0.7+1.1, idle 0.3+0.7+1.1
- balance:      min 1.1+1.0+1.1, idle 0.6+0.7+1.1
- performance:  min 1.1+1.2+1.1, idle 0.6+0.7+1.1
- fast:         min 1.1+1.5+1.7, idle 0.6+1.2+1.2
- Groups Tasks that use up to 30% of the little cluster into a single small core

sdm855/855+/860 (Schedutil)
- powersave:    min 1.3+0.7+0.8, idle 0.3+0.7+0.8
- balance:      min 1.1+1.0+0.8, idle 0.5+0.7+1.1
- performance:  min 1.1+1.2+0.8, idle 0.5+0.7+1.1
- fast:         min 1.1+1.6+1.6, idle 0.5+1.2+1.2
- Groups Tasks that use up to 30% of the little cluster into a single small core

sdm845 (Schedutil)
- powersave:    min 1.3+0.3, idle 0.3+0.3
- balance:      min 1.1+1.2, idle 0.5+0.8 
- performance:  min 1.1+1.2, idle 0.5+0.8 
- fast:         min 1.1+1.8, idle 0.5+1.6
- Groups Tasks that use up to 30% of the little cluster into a single small core

sdm765/sdm765g (Schedutil)
- powersave:    min 1.3+0.6+0.8, idle 0.3+0.6+0.8
- balance:      min 0.6+1.0+0.8, idle 0.3+0.6+0.6
- performance:  min 0.6+1.2+0.8, idle 0.3+0.6+0.8
- fast:         min 0.6+1.4+1.7, idle 0.3+1.1+1.4
- Groups Tasks that use up to 25% of the little cluster into a single small core

sdm730/sdm730g (Schedutil)
- powersave:    min 1.3+0.6, idle 0.3+0.6
- balance:      min 0.5+1.0, idle 0.3+0.6 
- performance:  min 0.5+1.2, idle 0.3+0.6  
- fast:         min 0.5+1.4, idle 0.3+1.2 
- Groups Tasks that use up to 25% of the little cluster into a single small core

sdm710/sdm712 (Schedutil)
- powersave:    min 1.3+0.6, idle 0.3+0.6
- balance:      min 0.5+1.1, idle 0.3+0.6
- performance:  min 0.5+1.1, idle 0.3+0.6
- fast:         min 0.5+1.5, idle 0.3+1.5
- Groups Tasks that use up to 25% of the little cluster into a single small core

sdm680/sdm685 (Schedutil)
- powersave:    min 1.3+0.8, idle 0.3+0.8
- balance:      min 0.9+1.0, idle 0.3+0.8 
- performance:  min 0.9+1.0, idle 0.3+0.8 
- fast:         min 0.9+1.7, idle 0.3+1.3 
- Groups Tasks that use up to 20% of the little cluster into a single small core

sdm675/sdm678 (Schedutil)
- powersave:    min 1.3+0.6, idle 0.3+0.6
- balance:      min 0.5+1.0, idle 0.3+0.6 
- performance:  min 0.5+1.2, idle 0.3+0.6  
- fast:         min 0.5+1.4, idle 0.3+1.2  
- Groups Tasks that use up to 20% of the little cluster into a single small core

sdm660/636 (Interactive + Project WIPE!)
- Still assembling the profiles
- Groups Tasks that use up to 15% of the little cluster into a single small core
```

- Battery Saver Mode: A mode that enables additional power-saving optimizations. It can be used with any profile; after all, it will only perform additional optimizations.

### Does your SOC not have support? And want to help with development? Read below
Perfd opt itself will optimize not only Snapdragon devices; other SoCs will also be able to integrate with the module, having their own specific profiles. If you didn't find your SoC in the compatibility list but want to contribute to the project and help me further cover it, please submit an issue answering the questions below:

Q. Is your SoC a MediaTek, Snapdragon, or?  
A. Please provide the name of your processor/CPU. Also, its codename. To obtain this, you can use this command: 
```
getprop ro.board.platform
```

Q. What is your SoC's frequency table?  
A. Enter the frequency table/available frequencies for your SoC here. It is only recommended to submit this if your kernel is not an ultra-optimized one that forced changes to the frequency table.

Q. What GPU do you have? And could you show me all the sysfs parameters for your GPU? If possible,   
A. Name your GPU, and if possible, show me the parameters and their values.

Q. If possible, show me any parameters you think are special about your SOC.   
A. The more information you give me about your SOC, the better the optimizations will be. And you don't need to try too hard; I can get some others from GitHub.

Q. If possible, please tell me the path to your DDR.
A. Just send me the path to your DDR. This will help me lock it to a more efficient governor and prevent unnecessary userspace changes. And also if you can, show me the available frequencies of your DDR.

## Requirements

1. Android 8-15
2. Rooted with Magisk or KSU

## Installation

1. Download zip in [Release Page](https://github.com/yc9559/perfd-opt/releases)
2. Flash in Magisk or KSU manager
3. Reboot and Check whether `/sdcard/Android/panel_powercfg.txt` exists. Remember Each SOC will have a different profile, so don't expect the same optimizations as an old or new SOC.   
4. Schedhorizon compatible, which will be preferred if the user has it in their kernel. It will also be optimized for "efficient" scaling.  
5. Schedutil is preferred for SOCs that were released without Interactive and without Project WIPE compatibility!
6. It is recommended that the user kernel not be heavily modified, such as kernels that change the frequency table.

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

### How to activate battery saving mode
Still in creation, we only have the base for now

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

@Matt Yang
the creator of perfd ​​opt, original credits to him for being the king of modules in 2019-2020

@AxionOS Devs
imported some optimizations to integrate the user experience improvement that perfd ​​offers

Credits to the artist of the image I used as the cover
```
