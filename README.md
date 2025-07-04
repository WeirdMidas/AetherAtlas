# Perfd opt

The previous [Project WIPE](https://github.com/yc9559/cpufreq-interactive-opt), automatically adjust the `interactive` parameters via simulation and heuristic optimization algorithms, and working on all mainstream devices which use `interactive` as default governor. The recent [WIPE v2](https://github.com/yc9559/wipe-v2), improved simulation supports more features of the kernel and focuses on rendering performance requirements, automatically adjusting the `interactive`+`HMP`+`input boost` parameters. However, after the EAS is merged into the mainline, the simulation difficulty of auto-tuning depends on raise. It is difficult to simulate the logic of the EAS scheduler. In addition, EAS is designed to avoid parameterization at the beginning of design, so for example, the adjustment of schedutil has no obvious effect.  

[WIPE v2](https://github.com/yc9559/wipe-v2) focuses on meeting performance requirements when interacting with APP, while reducing non-interactive lag weights, pushing the trade-off between fluency and power saving even further. `QTI Boost Framework`, which must be disabled before applying optimization, is able to dynamically override parameters based on perf hint. This project utilizes `QTI Boost Framework` and extends the ability of override custom parameters. When launching APPs or scrolling the screen, applying more aggressive parameters to improve response at an acceptable power penalty. When there is no interaction, use conservative parameters, use small core clusters as much as possible, and run at a higher energy efficiency OPP under heavy load.  

However... The Weird!Midas fork tends to go one step further than these simple optimizations, now knowing how the architectures work, at least the basics of them. perfd ​​opt can now use the QTI Boost Framework with better coherence with the scheduler and the device's TDP. This means that the way the module interacts with the device has been expanded to provide raw performance and energy efficiency above what perfd ​​can normally offer.

Details see [the lead project](https://github.com/yc9559/sdm855-tune/commits/master) & [perfd-opt commits](https://github.com/yc9559/perfd-opt/commits/master)    

## Profiles

- powersave: based on balance mode, but with lower max frequency
- balance: smoother than the stock config with lower power consumption
- performance: without frequency limitation with the addition of frequency stabilization
- fast: providing stable performance capacity considering the TDP limitation of device chassis

```plain
For the sake of work efficiency, the compatibility between the SOCs 
was reset, that is, I removed the SOCs that were compatible with
Matt Yang's profile. I hope you understand my decision, I had to 
align my work and make everything easier.

sdm680
- powersave:    0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
- balance:      0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
- performance:  0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
- fast:         0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0

sdm665
- powersave:    0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
- balance:      0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
- performance:  0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
- fast:         0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0

sdm660
- powersave:    0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
- balance:      0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
- performance:  0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
- fast:         0.0+0.0g, boost 0.0+0.0g, min 0.0+0.0
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
```
