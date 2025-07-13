# Perfd opt

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, when the HMP scheduler was removed from mainline and in sequence the interactive was also removed. New schedulers made by Google and central Linux such as WALT, EAS, generic CFS and PELT came into play. The way to manage the task of each scheduler became complicated, causing many users to have to specialize in each scheduler to know when and where to optimize to reduce energy consumption, without penalty of performance losses or latency. Causing information to be lost and everything to be obfuscated, making it difficult for users who would like to do this to optimize. This caused even schedutil to be affected, reducing the accuracy of tests because schedutil often works differently in each scheduler.

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. And this is where the changes compared to the original perfd ​​opt come in. Using the device's boosting mechanism, it is disabled to allow the rewriting of values, and then enabled, allowing mechanisms such as powerhint, perfboostconfig and others to use the rewritten values, improving integration. Based on this, the module's strategy begins: each compatible SOC has its own profile containing optimizations specific to it, where these optimizations will serve one thing: They will follow the "**opportunistic** rice-to-idle" strategy. This means that the device has a more "power-aware" task placement, where, if its demand is satisfied with the small cores, they are used; if necessary, the large cores are ramped directly to them without waiting. This allows the rice-to-idle strategy to be more "power-aware," reducing its penalties and allowing the strategy to become even more dynamic, respecting the Android workload at higher levels. And then, when resting, use lower frequencies, allowing small cores to be more preferred for low-consumption tasks, but not trivialized! Favoring a much more balanced scaling and close to what EAS once was.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Features

- Pure CPU and Scheduler optimization. With a full focus on integrating the "Rice-To-Idle" strategy that Snapdragon devices typically prefer, due to their sensitivity to latency. Initially, only Snapdragon devices are compatible. If the module is successful, we will integrate other SOCs into the set, such as Mediatek, Exynos, etc.
- Follow "**opportunistic** rice-to-idle," a scheduling strategy I created that allows the CPU to respond immediately to demand. However, instead of aggressively jumping between frequencies or clusters, opportunistic rice-to-idle seeks to satisfy demand with the small cluster, and if that's not enough, jumps directly to the big or prime cluster, allowing the CPU to meet multiple performance demands that have different performance and efficiency needs. Then, upon completing the task, the device quickly enters idle, with lower minimum frequencies than normal, allowing the device to rest and save power with maximum idle efficiency.
- The "immediate" response of our opportunistic Rice-to-idle strategy is more focused on ramping the CPU based on human perception. This means that the display, input boost, and even low-power hysteria are aligned to generate a 1-second window of "immediate response" time. Considering that human perception is 100ms, where above that point, noticeable lag is generated, this immediate performance window is enough to mitigate thrashing and improve scrolling performance and fluidity. If the user is using more aggressive power profiles, a greater preference is given to higher frequencies, and even more preference to more powerful cores. Depending on the most efficient or high-performance power profile used by the user at the time, the preference for cores beyond the little cores becomes more or less noticeable.
- Secondary improvements that favor our scheduler optimization, such as improvements to the CPUset, which isolates tasks that can be executed on small cores, and allows the CPU to correctly allocate tasks between their respective cgroups. We will not touch cgroups that require high performance, such as foreground and others, only those that can have their demand satisfied with the small ones.
- Follow a list of "inclusions", which are for "general but specific" optimizations for each SOC, such as the opportunistic refresh rate, which allows the device to use higher or lower rates depending on the interaction, favoring the "opportunistic rice-to-idle", inclusion of the "devfreq boost" which will be explained in some features later and also the inclusion of "task_profiles moderm", which means that the task_profiles of that specific SOC has been updated to the latest one (usually given to devices with cgroupv2).
- Include the dynamics of "Screen off", "preferred cluster" and "assistance cluster". These favor the scheduling strategy that perfd ​​opt will focus on for tasks in SOCs with variable cores. Screen off is when the screen turns off, which cores will handle tasks during the screen off duration. Preferred clusters are for SOCs that, for example, have six small cores, where these cores will be the favorites for user tasks, leaving the big/prime cores exclusively for heavy tasks. Assistance cluster means whether the device will have its prime cores assisting the big cores, this is to provide small improvements in load balancing in high-load situations where the delay in sending to the prime core cannot occur.
- Complete addition of a set of props that modernize certain subsystems and other areas. Allowing Surfaceflinger, Audioserver, Wi-Fi, and even Modem to be more efficient and have better quality/efficiency than standard, allowing SOCs to extract the most from their subsystems in terms of both power savings and quality.
- Included the "DevFreq boost" dynamic in the specific SOC boost framework. This means that the form of devfreq boost created by suitan will be able to be "imitated" by the boost framework that the SOC has. However, it is limited, containing only this dynamic in SOCs compatible with DDR BW v2 and with boost framework that work with paths. Which means that older SOCs will not be able to benefit from this optimization. Currently this dynamic covers the suitan commits mentioned above: devfreq boost for input, devfreq boost when a frame is committed, devfreq boost when a zygote-forked process becomes top-app and run devfreq boost on big cores.
- Have different and specific optimizations for devices that are not big.LITTLE, such as dynamlQ, allowing Perfd opt to fully adapt to the SOC and get the most out of it.
- If the processor has the same architecture (example: sdm685 and 680 that are from the same ID set), use hints like powerhint, perfboostconfig, msm_irqbalance.conf and even task_profiles for both, allowing the older one to take advantage of the newer one's optimizations and vice versa, facilitating the maintenance of both. BUT, only if both demonstrate to have the same architecture and are versions with slight optimizations between them and are in the same ID line, we will not put this on SOCs that have had many changes or have very different IDs.

## Profiles

- powersave: based on "balance" mode, but with a more aggressive "to-idle" mode, which may impact the UX for some users.
- balance: smoother and consumes less power than the stock setting. It tends to be more prone to idle.
- performance: more aggressive and consumes the most power, it tends to have the most aggressive "rice" compared to the others.
- fast: more stable as it meets the power needs of the device's chassis, allowing for fewer frequency variations and a more aggressive "rice-to-idle" on both sides.

```plain
For the sake of work efficiency, the compatibility between the SOCs 
was reset, that is, I removed the SOCs that were compatible with
Matt Yang's profile. I hope you understand my decision, I had to 
align my work and make everything easier.

SOC compatibility and technical specifications:
sdm680/sdm685 (Schedutil)
- powersave:    min 0.6+0.8, idle 0.3+0.3
- balance:      min 0.6+1.0, idle 0.3+0.8
- performance:  min 0.6+1.0, idle 0.3+0.8
- fast:         min 0.6+1.7, idle 0.3+1.3
- Inclusion of devfreq boost: yes
- Inclusion of opportunistic screen refresh rate: yes
- inclusion of modern task_profiles.json: yes
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
3. If your SOC comes with the ability to choose the form of Devfreq boost, only choose via Boost Framework if you don't have suitan's devfreq boost or other in your kernel, it is to avoid overboosting purposes
4. Reboot
5. Check whether `/sdcard/Android/panel_powercfg.txt` exists

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
