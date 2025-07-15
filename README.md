# Perfd-opt

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, when the HMP scheduler was removed from mainline and in sequence the interactive was also removed. New schedulers made by Google and central Linux such as WALT, EAS, generic CFS and PELT came into play. The way to manage the task of each scheduler became complicated, causing many users to have to specialize in each scheduler to know when and where to optimize to reduce energy consumption, without penalty of performance losses or latency. Causing information to be lost and everything to be obfuscated, making it difficult for users who would like to do this to optimize. This caused even schedutil to be affected, reducing the accuracy of tests because schedutil often works differently in each scheduler.

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. And this is where the changes compared to the original perfd ​​opt come in. Using the device's boosting mechanism, it is disabled to allow the rewriting of values, and then enabled, allowing mechanisms such as powerhint, perfboostconfig and others to use the rewritten values, improving integration. Based on this, the module will follow the strategy: "**opportunistic** rice-to-idle". A scheduling strategy designed to improve user experience on Android devices while simultaneously saving power by "running to idle." This means that during interactions, an aggressive response is made, but not without passing through the small cluster, biased toward it for certain demands. If unmet, it jumps directly to more powerful clusters. However, as the power profile currently being used becomes closer to high performance, more preference is given to powerful cores, and then, upon completion of the interaction, the device runs directly to idle, using much lower frequencies compared to the standard ones. Furthermore, the module comes with a classic "GameService," which allows for improved gaming experience for users, if desired.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Profiles

- powersave: based on "balance" mode, which, unlike "balance," has lower idle frequencies and a higher bias toward small cores.
- balance: smoother and more efficient than the stock configuration. it's biased toward the small cluster.
- performance: more aggressive during ramping, it's biased toward big cores, favoring maximum performance demand.
- fast: aggressive during "Rice" and "To-Idle," it always tries to save energy while maintaining performance. even without bias toward small cores, it always prioritizes respecting the device's chassis TDP.

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
- fast:         min 0.6+1.7  idle 0.3+1.3
```

- GameService: A daemon that serves to improve performance in games dynamically, through: pinning on the most powerful cores, reducing the migration cost for maximum multithreading performance and other useful boosting for the user.

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

credits to the artist of the image I used for the cover, whose name I can't say because I couldn't find it, sorry.
```
