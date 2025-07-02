# Perfd opt
![1000006461](https://github.com/user-attachments/assets/bd9a1384-dde5-4654-a80d-6687fc9e714a)

# Modern Scheduler Optimization with a Focus on Energy Efficiency

Modern Scheduler and CPU Optimization. Focused on delivering better CPU ramping, Touch response, app opening, energy savings and reducing moments that occur performance loss, of course, always respecting the TDP and architecture of the device for maximum respect for the user and use of its SOC.

The old WIPE project created by Matt Yang was initially known for its ability to adapt the Interactive + HMP + Input Boost parameters dynamically based on system load and device specifications. Later, he also created the perfd ​​opt project, known for pushing the boundaries between power saving and fluidity. However, the project has proven to be somewhat flawed. With its abrupt focus on aggressive responsiveness and power saving, perfd ​​opt had a lot of potential if optimized correctly.

The perfd ​​opt fork created by Weird Midas revives this old module created by Matt Yang, and adapts it to work in a way that follows the flow of the Android scheduler: Dynamic. The Android device works dynamically, with fast inputs and outputs between high, light and moderate loads, the Android device always adapts accordingly because it is a mobile device, so an adaptive approach is necessary. Based on this, the current perfd ​​opt uses the parameters and optimizations made by Matt Yang, but adapts, rebuilds and adjusts them to adapt to the Android platform with more coherence. Still focused on saving energy as much as possible, Perfd opt currently focuses on delivering a vastly improved dynamic behavior of Android with reduced energy costs.

The module integrates optimizations for most schedulers in its base, optimizing the behavior of generic CFS, WALT, EAS and even the old HMP. All focused on improving the dynamic behavior and energy efficiency of each one, favoring the best use of each feature of the device.

## Features

- Pure CPU and Scheduler optimization, without placebo when the goal is to improve the user experience in several aspects that impact the UX and system efficiency in terms of CPU & GPU usage. With the aim of reducing energy consumption.
- Choosing the best CPU governor for the specific processor, depending on its generation/era, allowing the adjustments to fit better with the architecture in question.
- Using a CPUset allocation that is more beneficial to the device architecture, favoring a more specialized multithreading in energy savings and increasing the speed of opening apps by up to 4-10% even in profiles with lower frequencies.
- Override sysfs performance adjustments and use QTI Boost Framework perfhints instead. Allowing you to have the same (or better) performance that you would have with these adjustments with reduced energy savings in idle.
- Optimize gaming performance using the perfhal, allowing the QTI Boost Framework to adapt system resources and improve game stability. This depends on the fps set in the game, allowing boosts to 30, 45, 60, 90, 120 and 144fps, which scales the aggressiveness/power requirement of the game according to the energy profiles used at the time. Of course, such "boosts" depend specifically on whether the device supports these fps, limiting the processor to use boosts that go up to its maximum screen refresh rate that it can handle.
- Additional and miscellaneous improvements to things that impact the user experience in the background. Such as reducing the energy consumption of media such as video and audio, in addition to allowing the camera to be better used, improving the experience of users who do not want to lose performance in media.

## Compatible SOCs and profiles

- powersave: based on balance mode, but with lower max frequency
- balance: smoother than the stock config with lower power consumption
- performance: without frequency limitation and with frequency sustainability optimizations
- fast: providing stable performance capacity considering the TDP limitation of device chassis

```plain
How it works:
Compatible SOC (Governor that it will use + if it has the boost mechanics available)
Profiles = Profiles such as powersave, balance, performance and fast will have their respective minimum and maximum frequencies, in addition to, of course, the frequency of the "boost" value if supported
- If the SOC is unable to have its frequency reduced due to limitations that could have a major impact on responsiveness, it will be marked as "No Jump". However, it will still have profile optimizations. Typically, SOCs with this marking are very old or have very low clocks
Run Freq = Frequency at which the CPU will immediately jump to the input, being a quick run to allow the processor to follow the flow of input > scheduler > governor
Intel Freq = Intermediate frequency below the two maximum frequency steps, favors energy consumption by allowing the system to satisfy the performance needs in high load situations with a slightly lower frequency
Equipped = It means what "additional" optimizations it comes equipped with besides the traditional scheduler and other ones:
- DDR: Comes equipped with bandwidth frequency control for the purpose of reducing power consumption

List of compatible SOCs:

sdm865 (schedutil + boost available)
- powersave:    1.8+1.6+2.4g, boost 1.8+2.0+2.6g, min 0.3+0.7+1.1
- balance:      1.8+2.0+2.6g, boost 1.8+2.4+2.7g, min 0.7+0.7+1.1
- performance:  1.8+2.4+2.8g, boost 1.8+2.4+2.8g, min 0.7+0.7+1.1
- fast:         1.8+2.0+2.7g, boost 1.8+2.4+2.8g, min 0.7+1.2+1.2
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with DDR

sdm855/sdm855+ (schedutil + boost available)
- powersave:    1.7+1.6+2.4g, boost 1.7+2.0+2.6g, min 0.3+0.7+0.8
- balance:      1.7+2.0+2.6g, boost 1.7+2.4+2.7g, min 0.5+0.7+0.8
- performance:  1.7+2.4+2.8g, boost 1.7+2.4+2.8/2.9g, min 0.5+0.7+0.8
- fast:         1.7+2.0+2.7g, boost 1.7+2.4+2.8/2.9g, min 0.5+1.2+1.2
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with DDR

sdm845 (schedutil + boost available)
- powersave:    1.7+2.0g, boost 1.7+2.4g, min 0.3+0.3
- balance:      1.7+2.4g, boost 1.7+2.7g, min 0.5+0.8
- performance:  1.7+2.8g, boost 1.7+2.8g, min 0.5+0.8
- fast:         1.7+2.4g, boost 1.7+2.8g, min 0.5+1.6
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with DDR

sdm765/sdm765g (schedutil + boost available)
- powersave:    1.8+1.7+2.0g, boost 1.8+2.0+2.2g, min 0.3+0.6+0.8
- balance:      1.8+2.0+2.2g, boost 1.8+2.2+2.3/2.4g, min 0.5+0.6+0.6
- performance:  1.8+2.2+2.3g, boost 1.8+2.2+2.3/2.4g, min 0.5+0.6+0.8
- fast:         1.8+2.0+2.2g, boost 1.8+2.2+2.3/2.4g, min 0.5+1.1+1.4
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with DDR

sdm730/sdm730g (schedutil + boost available)
- powersave:    1.7+1.5g, boost 1.7+1.9g, min 0.3+0.3
- balance:      1.7+1.9g, boost 1.7+2.1g, min 0.5+0.6
- performance:  1.8+2.2g, boost 1.8+2.2g, min 0.5+0.6
- fast:         1.8+1.9g, boost 1.8+2.2g, min 0.5+1.2
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with DDR

sdm710/sdm712 (schedutil + boost available)
- powersave:    1.7+1.8g, boost 1.7+2.0g, min 0.3+0.3
- balance:      1.7+2.0g, boost 1.7+2.2/2.3g, min 0.5+0.6
- performance:  1.7+2.2g, boost 1.7+2.2/2.3g, min 0.5+0.6
- fast:         1.7+2.0g, boost 1.7+2.2/2.3g, min 0.5+1.5
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with DDR

sdm695 (schedutil)
- It is still being ported and compatibility is being planned.

sdm680 (schedutil + boost available)
- powersave:    2.2+1.8g, boost 2.4+1.9g, min 0.3+0.3
- balance:      2.2+1.8g, boost 2.4+1.9g, min 0.6+0.8
- performance:  2.4+1.9g, boost 2.4+1.9g, min 0.6+0.8
- fast:         2.2+1.8g, boost 2.4+1.9g, min 0.6+1.3
- run freq: 1.4+1.6g
- intel freq: 1.6g+2.0g
- Equipped with DDR

sdm675 (schedutil + boost available)
- powersave:    1.7+1.5g, boost 1.7+1.7g, min 0.3+0.3
- balance:      1.7+1.7g, boost 1.7+1.9g, min 0.5+0.6
- performance:  1.8+2.0g, boost 1.8+2.0g, min 0.5+0.6
- fast:         1.8+1.7g, boost 1.8+2.0g, min 0.5+1.2
- run freq: 0.0+0.0g
- intel freq: 0.0+0.0g
- Equipped with DDR

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
```
