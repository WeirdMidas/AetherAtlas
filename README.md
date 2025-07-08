# Perfd opt

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, when the HMP scheduler was removed from mainline and in sequence the interactive was also removed. New schedulers made by Google and central Linux such as WALT, EAS, generic CFS and PELT came into play. The way to manage the task of each scheduler became complicated, causing many users to have to specialize in each scheduler to know when and where to optimize to reduce energy consumption, without penalty of performance losses or latency. Causing information to be lost and everything to be obfuscated, making it difficult for users who would like to do this to optimize. This caused even schedutil to be affected, reducing the accuracy of tests because schedutil often works differently in each scheduler.

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. And this is where the changes compared to the original perfd ​​opt come in. Using the device's boosting mechanism, it is disabled to allow the rewriting of values, and then enabled, allowing mechanisms such as powerhint, perfboostconfig and others to use the rewritten values, improving integration. Based on this, the module's strategy begins: each compatible SOC has its own profile containing optimizations specific to it, where these optimizations will serve one thing: to follow the rice-to-idle strategy, allowing the device to respond quickly to interaction, satisfying demand without restrictions. And then, when resting, use lower frequencies, allowing small cores to be more preferred for low-consumption tasks, but not trivialized! Favoring a much more balanced scaling and close to what EAS once was.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features

- Pure CPU and Scheduler optimization. With a full focus on integrating the "Rice-To-Idle" strategy that Snapdragon devices typically prefer, due to their sensitivity to latency. Initially, only Snapdragon devices are compatible. If the module is successful, we will integrate other SOCs into the set, such as Mediatek, Exynos, etc.
- Follow the "rice-to-idle" strategy, which means that the device in idle has frequencies much lower than the standard. However, when interaction occurs, the device returns to standard performance, allowing it to scale and satisfy the task so it can rest quickly.
- Introduce "roll-to-idle", a scheduling strategy similar to rice-to-idle but more focused on using "efficient frequencies" rather than responding to demand quickly. It will be used in certain SOCs that perform better with efficient frequencies.
- Secondary improvements that favor our scheduler optimization, such as improvements to the CPUset, which isolates tasks that can be executed on small cores, and allows the CPU to correctly allocate tasks between their respective cgroups. We will not touch cgroups that require high performance, such as foreground and others, only those that can have their demand satisfied with the small ones.
- Include the dynamics of "Screen off", "preferred cluster" and "assistance cluster". These favor the scheduling strategy that perfd ​​opt will focus on for tasks in SOCs with variable cores. Screen off is when the screen turns off, which cores will handle tasks during the screen off duration. Preferred clusters are for SOCs that, for example, have six small cores, where these cores will be the favorites for user tasks, leaving the big/prime cores exclusively for heavy tasks. Assistance cluster means whether the device will have its prime cores assisting the big cores, this is to provide small improvements in load balancing in high-load situations where the delay in sending to the prime core cannot occur.
- Improvements and optimizations for the overall user experience. Such as reducing energy consumption in media (video, audio and photo) without negative impact, in fact even improving their quality due to improved efficiency. In addition, optimizations in scrolling, reducing the latency of short and long scrolling for better viewing. In addition to other optimizations that improve the user experience.
- Included the "devfreq boost" dynamic via boost framework. An import of devfreq boost from suitan but adapted for perfhint mechanisms like QTI Boost Framework and etc. The user can choose whether to use the devfreq boost from perfd ​​opt, or from his kernel if he has it. It is recommended to use the one from the kernel as well as the one from suitan if you has it, because the module is unable to disable it.

## Profiles

- powersave: based on balance mode, but with more restrictions on "rice-to-idle". Which makes it the profile that saves the most energy and has UX performance equal to stock
- balance: smoother than stock configuration and with reduced power consumption. The selected default is also the ideal one for most tasks, with the most efficient and non-aggressive "rice-to-idle" strategy
- performance: fully integrated with the "rice-to-idle" strategy, it favors more aggressive frequency ramping to be able to rest more quickly. It is the profile that consumes the most energy of all
- fast: provide stable performance while respecting the TDP of the device chassis. It is a lower profile than performance, but with better FPS variation for example

```plain
For the sake of work efficiency, the compatibility between the SOCs 
was reset, that is, I removed the SOCs that were compatible with
Matt Yang's profile. I hope you understand my decision, I had to 
align my work and make everything easier.

SOC compatibility and technical specifications:
sdm680 (Schedutil)
- powersave:    min 0.6+0.8, idle 0.3+0.3
- balance:      min 0.6+1.0, idle 0.3+0.8
- performance:  min 0.6+1.0, idle 0.3+0.8
- fast:         min 0.6+1.7, idle 0.3+1.3
- Scheduling Used: Rice-to-idle, favors fluidity over battery
- Screen off: Uses cores 0-3 for tasks running during this period
- Preferred Cluster: None, load balancing is balanced
- Cluster Assistance: None, lack of prime cores to support
```

## Requirements

1. Android 8-15
2. Rooted with Magisk or KSU

## Installation

1. Download zip in [Release Page](https://github.com/yc9559/perfd-opt/releases)
2. Flash in Magisk or KSU manager
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

@Matt Yang
the creator of perfd ​​opt, original credits to him for being the king of modules in 2019-2020

@AxionOS Devs
imported some optimizations to integrate the user experience improvement that perfd ​​offers
```
