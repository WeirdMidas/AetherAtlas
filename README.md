# Perfd-opt

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, when the HMP scheduler was removed from mainline and in sequence the interactive was also removed. The new scheduler was integrated into recent Android devices (from 2020 onwards) and replaced the mainstream: EAS (Energy-Aware Scheduling), a scheduler that focuses on energy efficiency rather than raw performance, abandoning very old solutions like CFS (Completely Fair Scheduler) and even HMP (Heterogeneous Multi-Processing). But that's not all; trackers like WALT (Window-Assisted Load Tracking) and PELT (Per-Entity Load Tracking) were also integrated, allowing load tracking to be as stable or as fast as possible, depending on the device used. The way to manage the task of each scheduler and tracker became complicated, causing many users to have to specialize in each scheduler to know when and where to optimize to reduce energy consumption, without penalty of performance losses or latency. Causing information to be lost and everything to be obfuscated, making it difficult for users who would like to do this to optimize. This caused even schedutil to be affected, reducing the accuracy of tests because schedutil often works differently in each scheduler.

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. Based on this old project, Perfd opt tries to find a way to, in turn: make the EAS and the device's Tracker more "united", this means that the WALT, or even the PELT, can synchronize better with the EAS, allowing them to track loads, help the EAS to perform better, both energetically and in high performance, allowing the Scheduler and the device's Tracker to work together, improving the consistency of the WALT and the response of the PELT. Where the Scheduler and Tracker will follow the "**Opportunistic** Rice-to-idle" strategy, this means that the way the Scheduler and Tracker work is completely opportunistic with the bias in saving energy by having an immediate "to-idle" entry if the demand ends, of course, based on the performance demand, a more aggressive ramping (or better, "rice") is preferred, always looking for efficient frequencies that can feed the need for immediate performance of the situation. However, as the power profile currently being used becomes closer to high performance, more preference is given to powerful cores, and then, upon completion of the interaction, the device runs directly to idle, using much lower frequencies compared to the standard ones. 

See details of the original project created by Matt Yang [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features

- A Scheduler and CPU/GPU only optimization module, without placebo and with total focus on proposing an improved user experience in both efficiency and raw performance. With maximum priority in integrating the dynamic Android workload completely into each compatible SOC.
- Integrate a tracker optimization method called "Rice-to-idle." This type of tracker seeks rapid response, and not only that, but also responds quickly to demand and uses the device's IPC as a baseline. This method resolves as many tasks as possible in a short period of time before idling as quickly as possible, allowing for energy savings that border on the line between fluidity and energy savings.
- Integrate the "Scheduler of Opportunism" (SOP) behavior. This type of EAS scheduler optimization optimizes parameters such as subsystems that integrate deeply with the scheduler. With this EAS optimization method, the "rice-to-idle" tracker ultimately benefits. Because the scheduler makes much more efficient resource allocation decisions, it always prioritizes energy savings even under the most demanding performance profiles.
- Optimize the GPU and display to reduce power consumption of these two components, which are the biggest power consumers. The focus isn't on reducing GPU performance or refresh rate, but rather on making them more efficient and more opportunistic, allowing them to enter deeper idle states with zero impact on user-perceived fluidity.
- Optimize the way CPU sets operate as a whole. Improve the way tasks are allocated between cores and enable more efficiency and decision-making across the entire EAS scheduler.
- Respect the way each SOC architecture works. dynamlQ and big.LITTLE architectures differ in their task handling, which in turn: different optimizations are applied to each, with the two seeking different ways of handling tasks.
  - big.LITTLE has a separate cache between cores, which makes it quite limited in context switching situations. Therefore, its optimizations are more focused on improving task dispatch across cores and keeping tasks on optimal cores.
  - DynamLQ has a shared L3 cache, so it doesn't require additional optimizations that favor cache locality or task dispatch, allowing the DynamLQ scheduler to deliver on demand more efficiently.
- Additional optimizations and improvements to the rest of the Android subsystems that also interact with the scheduler, such as improvements to the encoder to reduce energy consumption in videos, audio and music to improve user UX.

## Profiles

- powersave: based on balance mode, but with a lower idle frequency and a faster to-idle entry.
  - Quickly goes to idle after interaction.
  - Top-app does not receive aggressive boosting, preferring justice over processes.
  - Adrenoboost in 1, preferring to improve GPU performance for a smooth UI and slightly better gaming.
- balance: smooth and balanced, better and more economical than the stock configuration. it's a balance between rice and to-idle.
  - Quickly goes to idle after interaction.
  - Top-app does not receive aggressive boosting, preferring justice over processes.
  - Adrenoboost in 1, preferring to improve GPU performance for a smooth UI and slightly better gaming.
- performance: without frequency limitation, prefer a more aggressive rice over an efficient to-idle.
  - Does not quickly go into idle after interaction.
  - Top-app receives more aggressive boosting. With the boost being 10 min uclamp or schedtune.boost respectively.
  - Adrenoboost in 2, preferring performance over battery life, prioritizing immediate GPU response but not at its maximum to avoid overload.
- fast: has both rice and aggressive to-idle, always seeking maximum performance and energy savings simultaneously, always respecting the device chassis TDP limit.
  - Quickly goes to idle after interaction.
  - Top-app receives more aggressive boosting. With the boost being 30 min uclamp or schedtune.boost respectively.
  - Adrenoboost in 1, preferring to improve GPU performance for a smooth UI and slightly better gaming.

```plain
For the sake of work efficiency, the compatibility between the SOCs 
was reset, that is, I removed the SOCs that were compatible with
Matt Yang's profile. I hope you understand my decision, I had to 
align my work and make everything easier.

idle = frequency that the SOC clusters will be at after the interaction ends
min = floor frequency that occurs during the interaction, which prevents the CPU from dropping to idle frequencies during the interaction
high speed + load = floor frequency and load for moments of high performance. If these demands occur, the CPU will be prohibited from dropping below these frequencies, avoiding frequency fluctuations during moments of extreme load

SOC compatibility and technical specifications:
sdm765/sdm765g (Schedutil)
- powersave:    min 0.9+1.2+0.8, idle 0.3+0.6+0.8
- balance:      min 0.9+1.2+0.8, idle 0.3+0.6+0.6
- performance:  min 0.9+1.2+0.8, idle 0.3+0.6+0.8
- fast:         min 0.9+1.4+1.7, idle 0.3+1.1+1.4
- high speed: 1.0+1.4+1.7, 90+90+80 load

sdm730/sdm730g (Schedutil)
- powersave:    min 0.9+0.6, idle 0.3+0.6
- balance:      min 0.9+1.0, idle 0.3+0.6 
- performance:  min 0.9+1.2, idle 0.3+0.6  
- fast:         min 0.9+1.4, idle 0.3+1.2 
- high speed: 1.0+1.4, 90+90 load

sdm710/sdm712 (Schedutil)
- powersave:    min 0.9+1.1, idle 0.3+0.6
- balance:      min 0.9+1.1, idle 0.3+0.6 
- performance:  min 0.9+1.1, idle 0.3+0.6 
- fast:         min 0.9+1.5, idle 0.3+1.1
- high speed: 0.9+1.5, 90+90 load

sdm680/sdm685 (Schedutil)
- powersave:    min 0.9+1.3, idle 0.3+0.8
- balance:      min 0.9+1.3, idle 0.3+0.8 
- performance:  min 0.9+1.3, idle 0.3+0.8 
- fast:         min 0.9+1.7, idle 0.3+1.3 
- high speed: 1.1+1.7, 90+90 load

sdm675 (Schedutil)
- powersave:    min 0.9+0.6, idle 0.3+0.6
- balance:      min 0.9+1.0, idle 0.3+0.6 
- performance:  min 0.9+1.2, idle 0.3+0.6  
- fast:         min 0.9+1.4, idle 0.3+1.2 
- high speed: 1.0+1.4, 90+90 load

sdm660/sdm636 (Interactive + Project WIPE!)
- powersave:    high speed 1.0+1.3, 98+98 load, min 0.9+1.3, idle 0.3+0.8
- balance:      high speed 1.0+1.3, 98+83 load, min 0.9+1.3, idle 0.3+0.8
- performance:  high speed 1.0+1.3, 89+98 load, min 0.9+1.3, idle 0.3+0.8
- fast:         high speed 1.0+1.5, 98+81 load, min 0.9+1.5, idle 0.3+1.3 
```

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
A. Just send me the path to your DDR. This will help me lock it to a more efficient governor and prevent unnecessary userspace changes.

## Requirements

1. Android 8-15
2. Rooted with Magisk or KSU

## Warning

1. It is recommended to use the module only on kernels without extreme optimization, such as changes to the frequency table for better efficiency, etc. Generic, optimized but not ultra-optimized kernels can benefit from the module with pleasure.

## Installation

1. Download zip in [Release Page](https://github.com/yc9559/perfd-opt/releases)
2. Flash in Magisk or KSU manager
3. Reboot and Check whether `/sdcard/Android/panel_powercfg.txt` exists

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

@Matt Yang
the creator of perfd ​​opt, original credits to him for being the king of modules in 2019-2020

@AxionOS Devs
imported some optimizations to integrate the user experience improvement that perfd ​​offers
```
