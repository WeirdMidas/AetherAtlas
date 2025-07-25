# Perfd-opt

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, when the HMP scheduler was removed from mainline and in sequence the interactive was also removed. The new scheduler was integrated into recent Android devices (from 2020 onwards) and replaced the mainstream: EAS (Energy-Aware Scheduling), a scheduler that focuses on energy efficiency rather than raw performance, abandoning very old solutions like CFS (Completely Fair Scheduler) and even HMP (Heterogeneous Multi-Processing). But that's not all; trackers like WALT (Window-Assisted Load Tracking) and PELT (Per-Entity Load Tracking) were also integrated, allowing load tracking to be as stable or as fast as possible, depending on the device used.
The way to manage the task of each scheduler and tracker became complicated, causing many users to have to specialize in each scheduler to know when and where to optimize to reduce energy consumption, without penalty of performance losses or latency. Causing information to be lost and everything to be obfuscated, making it difficult for users who would like to do this to optimize. This caused even schedutil to be affected, reducing the accuracy of tests because schedutil often works differently in each scheduler.

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. And this is where the changes compared to the original perfd ​​opt come in. Using the device's boosting mechanism, it is disabled to allow the rewriting of values, and then enabled, allowing mechanisms such as powerhint, perfboostconfig and others to use the rewritten values, improving integration. Based on this, Perfd opt tries to find a way to, in turn: make the EAS and the device's Tracker more "united", this means that the WALT, or even the PELT, can synchronize better with the EAS, allowing them to track loads, help the EAS to perform better, both energetically and in high performance, allowing the Scheduler and the device's Tracker to work together, improving the consistency of the WALT and the response of the PELT. Where the Scheduler and Tracker will follow the "**Opportunistic** Rice-to-idle" strategy, this means that the way the Scheduler and Tracker work is completely opportunistic with the bias in saving energy by having an immediate "to-idle" entry if the demand ends, of course, based on the performance demand, a more aggressive ramping (or better, "rice") is preferred, always looking for efficient frequencies that can feed the need for immediate performance of the situation. However, as the power profile currently being used becomes closer to high performance, more preference is given to powerful cores, and then, upon completion of the interaction, the device runs directly to idle, using much lower frequencies compared to the standard ones. 

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features

- A Scheduler and CPU/GPU only optimization module, without placebo and with total focus on proposing an improved user experience in both efficiency and raw performance. With maximum priority in integrating the dynamic Android workload completely into each compatible SOC.
- Integrate a tracker optimization method called "Rice-to-idle." This type of tracker seeks rapid response, and not only that, but also responds quickly to demand and uses the device's IPC as a baseline. This method resolves as many tasks as possible in a short period of time before idling as quickly as possible, allowing for energy savings that border on the line between fluidity and energy savings.
- Integrate the "Scheduler of Opportunism" (SOP) behavior. This type of EAS scheduler optimization optimizes parameters such as the SOC boost framework and other subsystems that integrate deeply with the scheduler. With this EAS optimization method, the "rice-to-idle" tracker ultimately benefits. Because the scheduler makes much more efficient resource allocation decisions, it always prioritizes energy savings even under the most demanding performance profiles.
- Optimize Boost Framework. Eliminate unnecessary boosting and focus only on those that can improve the device's fluidity while minimizing the impact on power savings. The SOC's Boost Framework will be used to provide immediate performance in these situations, focusing on efficiency (quickly completing the task and resting immediately).
- Respect the way each SOC architecture works. dynamlQ and big.LITTLE architectures differ in their task handling, which in turn: different optimizations are applied to each, with the two seeking different ways of handling tasks.
  - big.LITTLE will pursue simple but classic efficiency, prioritizing the use of small cores for light and moderate tasks. Whereas big cores will be reserved exclusively for heavy or critical display tasks, prioritizing fluidity in this regard.
  - DynamLQ will pursue intelligent and advanced efficiency, prioritizing a scheduling method that, unlike big.LITTLE, seeks to push the limits of efficiency and performance simultaneously.

## Profiles

- powersave: based on balance mode, but with a lower idle frequency and a faster to-idle entry.
  - Quickly goes to idle after interaction;
  - Top-app does not receive aggressive boosting, preferring justice over processes;
- balance: smooth and balanced, better and more economical than the stock configuration. it's a balance between rice and to-idle.
  - Quickly goes to idle after interaction;
  - Top-app does not receive aggressive boosting, preferring justice over processes;
- performance: without frequency limitation, prefer a more aggressive rice over an efficient to-idle.
  - Does not quickly go into idle after interaction;
  - Top-app receives more aggressive boosting. With the boost being 10 min uclamp and schedtune.boost respectively;
- fast: has both rice and aggressive to-idle, always seeking maximum performance and energy savings simultaneously, always respecting the device chassis TDP limit.
  - Quickly goes to idle after interaction;
  - Top-app receives more aggressive boosting. With the boost being 30 min uclamp and schedtune.boost respectively;

```plain
For the sake of work efficiency, the compatibility between the SOCs 
was reset, that is, I removed the SOCs that were compatible with
Matt Yang's profile. I hope you understand my decision, I had to 
align my work and make everything easier.

SOC compatibility and technical specifications:
sdm730/sdm730g (Schedutil)
- powersave:    min 0.5+0.6, idle 0.3+0.3 
- balance:      min 0.5+1.0, idle 0.3+0.6 
- performance:  min 0.5+1.2, idle 0.3+0.6 
- fast:         min 0.5+1.4, idle 0.3+1.2 
- Contains Boost Framework Optimizations: No

sdm710/sdm712 (Schedutil)
- powersave:    min 0.9+1.1, idle 0.3+0.3 
- balance:      min 0.9+1.1, idle 0.3+0.6 
- performance:  min 0.9+1.5, idle 0.3+0.6 
- fast:         min 0.9+1.5, idle 0.3+1.5
- Contains Boost Framework Optimizations: No

sdm680/sdm685 (Schedutil)
- powersave:    min 0.6+0.8, idle 0.3+0.3 
- balance:      min 0.6+1.0, idle 0.3+0.8 
- performance:  min 0.6+1.0, idle 0.3+0.8 
- fast:         min 0.6+1.7, idle 0.3+1.3 
- Contains Boost Framework Optimizations: Yes

sdm675 (Schedutil)
- powersave:    min 0.5+0.6, idle 0.3+0.3 
- balance:      min 0.5+1.0, idle 0.3+0.6 
- performance:  min 0.5+1.2, idle 0.3+0.6  
- fast:         min 0.5+1.4, idle 0.3+1.2 
- Contains Boost Framework Optimizations: No

sdm660 (Interactive + Project WIPE!)
- powersave:    min 0.8+1.3, idle 0.3+0.3 
- balance:      min 0.8+1.3, idle 0.3+0.3 
- performance:  min 0.8+1.3, idle 0.3+0.3 
- fast:         min 0.8+1.3, idle 0.3+0.3 
- Contains Boost Framework Optimizations: No

sdm636 (Interactive + Project WIPE!)
- powersave:    min 0.8+1.3, idle 0.3+0.3 
- balance:      min 0.8+1.3, idle 0.3+0.3
- performance:  min 0.8+1.3, idle 0.3+0.3
- fast:         min 0.8+1.3, idle 0.3+0.3 
- Contains Boost Framework Optimizations: No
```

## Requirements

1. Android 8-15
2. Rooted with Magisk or KSU

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
