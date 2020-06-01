---
title: ADB常用命令
date: 2020-06-01 10:43:08
tags: [Android, ADB]
summary: ADB常用命令
---

<!-- toc --> 

## 一、查看当前连接设备
**命令格式：**   
`adb devices`

## 二、为命令指定目标设备
如果有多个设备/模拟器连接，则需要为命令指定目标设备。

参数 | 含义
:-:|---
**-d** | **指定当前唯一通过 USB 连接的 Android 设备为命令目标**
-e	 | 指定当前唯一运行的模拟器为命令目标
-s <serialNumber>	 | 指定相应 serialNumber 号的设备/模拟器为命令目标
其中`serialNumber`使用`adb devices`命令获取，如下
```
localhost:zs_student_android ltt$ adb devices
List of devices attached
e68e6c20        device
10.201.62.78:5555       device
```
输出里的 e68e6c20、10.201.62.78:5555 即为 serialNumber。示例如下:   
`adb -s 10.201.62.78:5555 install test.apk`

## 三、断开无线连接的设备
**命令格式：**  
`adb disconnect <device-ip-address>`

## 四、应用管理
### 1. 查看应用列表  
**命令格式：**  
`adb shell pm list packages [-f] [-d] [-e] [-s] [-3] [-i] [-u] [--user USER_ID] [FILTER]`  
参数 |显示列表  
:-:|---
无|所有应用  
-f|显示应用关联的 apk 文件
-d|只显示 disabled 的应用
-e|只显示 enabled 的应用
-s|只显示系统应用
-3|只显示第三方应用
-i|显示应用的 installer
-u|包含已卸载应用
<FILTER>|包名包含 <FILTER> 字符串
### 2. 安装APK
**命令格式：**  
`adb install [-lrtsdg] <path_to_apk>`  
参数|含义  
:-:|---
-l|将应用安装到保护目录 /mnt/asec
**-r**|**允许覆盖安装**
-t|允许安装 AndroidManifest.xml 里 application 指定 android:testOnly="true" 的应用
-s|将应用安装到 sdcard
-d|允许降级覆盖安装
-g|授予所有运行时权限
运行命令后成功输出如下：  
```
localhost:zs_student_android ltt$ adb -d install -r debug.apk 
Performing Streamed Install
Success
```
### 3. 卸载应用   
**命令格式：**  
`adb uninstall [-k] <packagename>`  
说明：<packagename> 表示应用的包名，-k 参数可选，表示卸载应用但保留数据和缓存目录  

### 4. 清除应用数据与缓存   
**命令格式：**  
`adb shell pm clear <packagename>`  
说明：<packagename> 表示应用名包，这条命令的效果相当于在设置里的应用信息界面点击了「清除缓存」和「清除数据」。

### 5. 查看顶部Activity
- windows环境下:  
`adb shell dumpsys activity | findstr "mFocusedActivity"`

-  Linux、Mac环境下：  
 `adb shell dumpsys activity | grep "mFocusedActivity"`

### 6. 查看应用详细信息  
**命令格式：**  
`adb shell dumpsys package <packagename>`  
说明：输出中包含很多信息，包括 Activity Resolver Table、Registered ContentProviders、包名、userId、安装后的文件资源代码等路径、版本信息、权限信息和授予状态、签名版本信息等。   
### 7. 强制停止应用
命令格式：  
`adb shell am force-stop <packagename>`  
## 五、文件管理  
### 1. 复制设备里的文件到电脑
**命令格式：**
`adb pull <设备里的文件路径> [电脑上的目录]`  
说明：其中 电脑上的目录 参数可以省略，默认复制到当前目录。  
*小技巧：设备上的文件路径可能需要 root 权限才能访问，如果你的设备已经 root 过，可以先使用 adb shell 和 su 命令在 adb shell 里获取 root 权限后，先 cp /path/on/device /sdcard/filename 将文件复制到 sdcard，然后 adb pull /sdcard/filename /path/on/pc。*  
### 2. 复制电脑里的文件到设备
**命令格式：**  
`adb push <电脑上的文件路径> <设备里的目录>`  
*小技巧：设备上的文件路径普通权限可能无法直接写入，如果你的设备已经 root 过，可以先 adb push /path/on/pc /sdcard/filename，然后 adb shell 和 su 在 adb shell 里获取 root 权限后，cp /sdcard/filename /path/on/device。*  
## 六、查看设备信息  
### 1. 查看型号  
**命令格式：**  
`adb shell getprop ro.product.model`  
输出示例：  
```
localhost:zs_student_android ltt$ adb -d shell
natrium:/ $ getprop ro.product.model
MI 5s Plus
```
### 2. 电池状况  
**命令格式：**  
`adb shell dumpsys battery`  
输出示例：  
```
localhost:zs_student_android ltt$ adb -d shell
natrium:/ $ dumpsys battery
Current Battery Service state:
  AC powered: false
  USB powered: true
  Wireless powered: false
  Max charging current: 500000
  Max charging voltage: 5000000
  Charge counter: 3057193
  status: 2
  health: 2
  present: true
  level: 100
  scale: 100
  voltage: 4322
  temperature: 355
  technology: Li-poly
natrium:/ $ 
```
### 3. 屏幕分辨率  
**命令格式：**
`adb shell wm size`  
输出示例:  
```
natrium:/ $ wm size          
Physical size: 1080x1920
natrium:/ $ 
```
### 4. 屏幕密度  
**命令格式：**  
`adb shell wm density`  
输出示例:  
```
natrium:/ $  wm density
Physical density: 480
natrium:/ $ 
```
### 5. 显示屏参数  
**命令格式：**  
`adb shell dumpsys window displays`  
输出示例:  
```
natrium:/ $ dumpsys window displays
WINDOW MANAGER DISPLAY CONTENTS (dumpsys window displays)
  Display: mDisplayId=0
    init=1080x1920 480dpi cur=1080x1920 app=1080x1920 rng=1080x1008-1920x1848
    deferred=false mLayoutNeeded=false mTouchExcludeRegion=SkRegion((0,0,1080,1920))
```
### 6. IP 地址  

**命令格式：**  
`adb shell ifconfig | grep Mask`  
输出示例:  
```
|natrium:/ $ ifconfig
lo        Link encap:UNSPEC  
          inet addr:127.0.0.1  Mask:255.0.0.0 
          inet6 addr: ::1/128 Scope: Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:36944 errors:0 dropped:0 overruns:0 frame:0 
          TX packets:36944 errors:0 dropped:0 overruns:0 carrier:0 
          collisions:0 txqueuelen:0 
          RX bytes:3245483 TX bytes:3245483 

dummy0    Link encap:UNSPEC  
          inet6 addr: fe80::b4e7:cfff:fef9:4fae/64 Scope: Link
          UP BROADCAST RUNNING NOARP  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0 
          TX packets:166 errors:0 dropped:0 overruns:0 carrier:0 
          collisions:0 txqueuelen:0 
          RX bytes:0 TX bytes:11620 

wlan0     Link encap:UNSPEC    Driver cnss_wlan_pci
          inet addr:10.211.56.115  Bcast:10.211.63.255  Mask:255.255.248.0 
          inet6 addr: fe80::c60b:cbff:fe82:a158/64 Scope: Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7718005 errors:0 dropped:0 overruns:0 frame:0 
          TX packets:867328 errors:0 dropped:3 overruns:0 carrier:0 
          collisions:0 txqueuelen:3000 
          RX bytes:2542947775 TX bytes:120155558 

rmnet_ipa0 Link encap:UNSPEC  
          UP RUNNING  MTU:2000  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0 
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0 
          collisions:0 txqueuelen:1000 
          RX bytes:0 TX bytes:0 
```
### 7. 更多硬件与系统属性
**命令格式：**  
`adb shell cat /system/build.prop`  
**说明：**  
这会输出很多信息，包括前面几个小节提到的「型号」等。  
输出里还包括一些其它有用的信息，它们也可通过 adb shell getprop <属性名> 命令单独查看，列举一部分属性如下：  
属性名|含义
:-:|---
ro.build.version.sdk|SDK 版本
ro.build.version.release|Android 系统版本
ro.build.version.security_patch|Android 安全补丁程序级别
ro.product.model|型号
ro.product.brand|品牌
ro.product.name|设备名
ro.product.board|处理器型号
ro.product.cpu.abilist|CPU 支持的 abi 列表[节注一]
persist.sys.isUsbOtgEnabled|是否支持 OTG
dalvik.vm.heapsize|每个应用程序的内存上限
ro.sf.lcd_density|屏幕密度  
输出示例：  
```

:/ # cat /system/build.prop

# begin build properties
# autogenerated by buildinfo.sh
ro.build.id=PQ3A.190801.002
ro.build.display.id=lineage_natrium-userdebug 9 PQ3A.190801.002 cd6bf65336
ro.build.version.incremental=cd6bf65336
ro.build.version.sdk=28
ro.build.version.preview_sdk=0
ro.build.version.codename=REL
ro.build.version.all_codenames=REL
ro.build.version.release=9
ro.build.version.security_patch=2019-11-05
ro.build.version.base_os=
ro.build.version.min_supported_target_sdk=17
ro.build.date=Tue Dec 10 11:41:15 UTC 2019
ro.build.date.utc=1575978075
ro.build.type=userdebug
ro.build.user=buildkite-agent
ro.build.host=lineageos-buildkite
ro.build.tags=release-keys
ro.build.flavor=lineage_natrium-userdebug
ro.product.model=MI 5s Plus
ro.product.brand=Xiaomi
ro.product.name=natrium
ro.product.device=natrium
# ro.product.cpu.abi and ro.product.cpu.abi2 are obsolete,
# use ro.product.cpu.abilist instead.
ro.product.cpu.abi=arm64-v8a
ro.product.cpu.abilist=arm64-v8a,armeabi-v7a,armeabi
ro.product.cpu.abilist32=armeabi-v7a,armeabi
ro.product.cpu.abilist64=arm64-v8a
ro.product.manufacturer=Xiaomi
ro.product.locale=en-US
ro.wifi.channels=
# ro.build.product is obsolete; use ro.product.device
ro.build.product=natrium
# Do not try to parse description, fingerprint, or thumbprint
ro.build.description=natrium-user 7.0 NRD90M V9.6.2.0.NBGMIFD release-keys
ro.build.fingerprint=Xiaomi/natrium/natrium:7.0/NRD90M/V9.6.2.0.NBGMIFD:user/release-keys
ro.build.characteristics=default
ro.lineage.device=natrium
# end build properties

#
# ADDITIONAL_BUILD_PROPERTIES
#
ro.treble.enabled=false
persist.sys.dalvik.vm.lib.2=libart.so
dalvik.vm.isa.arm64.variant=kryo
dalvik.vm.isa.arm64.features=default
dalvik.vm.isa.arm.variant=kryo
dalvik.vm.isa.arm.features=default
dalvik.vm.lockprof.threshold=500
net.bt.name=Android
dalvik.vm.stack-trace-dir=/data/anr
ro.lineage.version=16.0-20191210-NIGHTLY-natrium
ro.lineage.releasetype=NIGHTLY
ro.lineage.build.version=16.0
ro.modversion=16.0-20191210-NIGHTLY-natrium
ro.lineagelegal.url=https://lineageos.org/legal
ro.lineage.display.version=16.0-20191210-NIGHTLY-natrium
ro.lineage.build.version.plat.sdk=9
ro.lineage.build.version.plat.rev=0
ro.build.expect.modem=2018-11-21 10:46:10,8.11.23
ro.expect.recovery_id=0xdba5ab1265429e5480be01f327fb9860e39a1735000000000000000000000000
```  
*小提示：没有root可能会提示`cat: /system/build.prop: Permission denied`，这时在adb shell 里直接执行`su`就会看到用户命令提示符由”$”变成了”#”表示adb shell获取了root权限(仅限手机已root，且有su包)*  

***参考：[玩转ADB命令（ADB命令使用大全）https://blog.csdn.net/zhonglunshun/article/details/78362439](https://blog.csdn.net/zhonglunshun/article/details/78362439)***
