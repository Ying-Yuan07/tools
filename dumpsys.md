## 简介



## gfxinfo

【1】由于 Choreographer 的位置，许多性能监控的手段都是利用 Choreographer 来做的，除了自带的掉帧计算，Choreographer 提供的 FrameCallback 和 FrameInfo 都给 App 暴露了接口，让 App 开发者可以通过这些方法监控自身 App 的性能，其中常用的方法如下

```shell
adb shell dumpsys gfxinfo <应用包名> framestats reset #执行 2-3 次，清除历史数据
adb shell dumpsys gfxinfo <应用包名> framestats 
```

提供了关于指定Android应用程序（例如`com.ss.android.ugc.aweme`）图形渲染的详细信息

以下是这个命令通常输出的内容及其含义：

1. 重点关注

2. 1. **Janky frames** ：超过 Vsync 周期的 Frame，不一定出现卡顿
   2. **95th percentile** ：95% 的值
   3. **HISTOGRAM** ：原始数值
   4. **PROFILEDATA** ：每一帧的详细原始数据



1. **基本信息**:
   - **运行时间（Uptime）**: 应用程序自启动以来的运行时间。
   - **实际时间（Realtime）**: 自设备启动以来的实际时间。
2. **帧统计**:
   - **总渲染帧数**: 应用程序启动以来渲染的总帧数。
   - **卡顿帧数（Janky frames）**: 未能达到期望的渲染帧率（通常为60帧每秒）的帧数。
   - **百分位数（Percentiles）**: 帧渲染时间的延迟百分位数（50th, 90th, 95th, 99th）。
   - **丢失Vsync次数（Number of Missed Vsync）**: 未能及时匹配垂直同步的帧数。
   - **高输入延迟次数（Number High input latency）**: 高输入响应延迟的次数。
   - **UI线程慢次数（Number Slow UI thread）**: UI线程响应缓慢的次数。
   - **位图上传慢次数（Number Slow bitmap uploads）**: 位图上传操作响应缓慢的次数。
   - **绘制命令慢次数（Number Slow issue draw commands）**: 绘制命令执行缓慢的次数。
   - **帧期限未达次数（Number Frame deadline missed）**: 未能按时完成帧渲染的次数。
3. **帧时间直方图（HISTOGRAM）**:
   - 显示了各个时间区间内帧渲染所占的比例，单位为毫秒。
4. **GPU百分位数**:
   - 显示了GPU渲染时间的延迟百分位数（50th, 90th, 95th, 99th）。
5. **CPU和GPU内存使用**:
   - 包括Java堆、Native堆、图形缓存等内存区域的详细使用情况。



adb shell dumpsys gfxinfo com.meizu.flyme.launcher framestats



## SurfaceFlinger 

于获取设备上 SurfaceFlinger 的帧统计信息的命令。这个命令可以帮助你了解应用程序的帧率以及帧时间等信息，从而用于性能分析和优化

```shell
adb shell dumpsys SurfaceFlinger --latency <SurfaceView_name>
```

`<SurfaceView_name>` 应替换为你需要分析的应用或 SurfaceView 的名称。

要查看应用中的 `SurfaceView` 名称，可以使用以下方法：

### 使用 `adb shell dumpsys SurfaceFlinger --list`

这个命令列出所有 `SurfaceFlinger` 管理的 surface：

```shell
adb shell dumpsys SurfaceFlinger --list 
adb shell dumpsys SurfaceFlinger --list | grep bili #查看关于bilibili的surface
```



```shell
blueline:/ # dumpsys SurfaceFlinger --latency tv.danmaku.bili/tv.danmaku.bili.MainActivityV2#0
16666666
156713354463747 156713380684426 156713356378538
156713370290207 156713397089584 156713372318800
```



### 输出解释

1. 第一行的单个数值 `16666666` 表示的是理想帧间隔时间，单位为纳秒。这个数值是 60Hz 屏幕的帧间隔时间（1 秒除以 60 得到的间隔时间为 16.666666 毫秒）。

   从第二行开始，每行包含三个时间戳：

   - 第一个时间戳：表示帧的显示时间。
   - 第二个时间戳：表示 vsync 信号的时间。
   - 第三个时间戳：表示渲染完成的时间。

### 分析方法

为了分析帧性能，可以关注有效的第二个和第三个时间戳之间的差异，这代表渲染所花费的时间。有效的第二个时间戳是 vsync 信号时间，有效的第三个时间戳是渲染完成的时间。

1. 利用 SurfaceFlinger PageFlip 机制进行监控
2. 使用 ：adb service call SurfaceFlinger 1013

```shell
blueline:/ # service call SurfaceFlinger 1013
Result: Parcel(000baafb    '....')
```

`service call SurfaceFlinger 1013` 命令返回的结果是一个 Parcel 格式的数据，它的解析需要进一步的处理。让我们更详细地了解如何处理这个 Parcel 数据，以便提取出有用的信息。

### SurfaceFlinger 1013 的作用

在 SurfaceFlinger 中，`1013` 是用来获取与显示刷新和帧率相关的统计信息的函数代码。调用这个接口可以获取 SurfaceFlinger 的当前状态，包括帧率、刷新率等。

### Parcel 数据格式

Parcel 是 Android 内部使用的一种二进制序列化格式，通常用来在进程之间传递复杂的数据结构。我们需要解析这个 Parcel 数据来获得具体的帧率和刷新率信息。

### 使用 Python 解析 Parcel 数据





## meminfo

### total system

```shell
adb shell dumpsys meminfo
```

可以获取设备上进程的内存使用情况。这条命令会输出当前系统上所有进程的内存信息，包括总的内存使用情况和按进程的详细内存使用情况。以及在内存中的数据类型

1. **VSS**：
Virtual Set Size 虚拟耗用内存（包含共享库占用的内存）
VSS表示一个进程可访问的全部内存地址空间的大小。这个大小包括了进程已经申请但尚未使用的内存空间。在实际中很少用这种方式来表示进程占用内存的情况，用它来表示单个进程的内存使用情况是不准确的。

此大小还包括可能不驻留在RAM中的内存，如已分配但未写入的malloc。 VSS对于确定进程的实际内存使用非常少用。

2. **RSS**： Resident Set Size 实际使用物理内存（包含共享库占用的内存）
表示一个进程在RAM中实际使用的空间地址大小，包括了全部共享库占用的内存，这种表示进程占用内存的情况也是不准确的。

RSS可能会产生误导，因为它报告进程使用的所有共享库的总数，即使共享库只加载到内存中一次，无论有多少进程使用它。 RSS不是单个进程的内存使用的准确表示。

3. **PSS**： Proportional Set Size 实际使用的物理内存（比例分配共享库占用的内存）
表示一个进程在RAM中实际使用的空间地址大小，它按比例包含了共享库占用的内存。假如有3个进程使用同一个共享库，那么每个进程的PSS就包括了1/3大小的共享库内存。这种方式表示进程的内存使用情况较准确，但当只有一个进程使用共享库时，其情况和RSS一模一样。

PSS可能有点误导，因为当进程被杀死时，PSS不能准确地表示返回到整个系统的内存。

4. **USS**： Unique Set Size 进程独自占用的物理内存（不包含共享库占用的内存）
表示一个进程本身占用的内存空间大小，不包含其它任何成分，这是表示进程内存大小的最好方式！

USS是一个非常有用的数字，因为它表示运行特定进程的真正增量成本。当进程被终止时，USS是实际返回到系统的总内存。 USS是判断进程中的内存泄漏时最值得注意的数字。

总结
一般来说内存占用大小有如下规律：VSS >= RSS >= PSS >= USS

```shell
blueline:/ # dumpsys meminfo
Applications Memory Usage (in Kilobytes):
Uptime: 180009168 Realtime: 1100407792


Total RSS by process:
    524,891K: tv.danmaku.bili (pid 18440 / activities)
    372,567K: system (pid 1814)
    280,152K: com.xingin.xhs (pid 23101)
	...
Total RSS by OOM adjustment:
    541,227K: Native
         87,500K: app_process (pid 23488)
         58,131K: surfaceflinger (pid 723)
         52,380K: zygote64 (pid 1024)
         28,172K: zygote (pid 23509)
         21,152K: zygote (pid 1026)
         10,820K: vendor.google.wifi_ext@1.0-service-vendor (pid 1081)
         ...
Total RSS by category:
    593,556K: .so mmap
    511,548K: .art mmap
    420,988K: Native
    376,644K: .jar mmap
    277,412K: Dalvik
    259,852K: .dex mmap
    171,312K: .oat mmap
    138,488K: EGL mtrack
    110,236K: GL mtrack
     76,592K: Other mmap
     70,608K: Dalvik Other
     61,996K: .apk mmap
     51,020K: Unknown
     42,420K: Gfx dev
     28,632K: Other dev
     28,200K: Stack
     22,332K: Ashmem
        860K: .ttf mmap
          0K: Cursor
          0K: Other mtrack
          ...
 Total RAM: 3,665,320K (status normal)
 Free RAM: 1,757,975K (  549,199K cached pss + 1,039,480K cached kernel +   169,296K free)
 Used RAM: 2,193,200K (1,842,780K used pss +   350,420K kernel)
 Lost RAM:   242,720K
     ZRAM:   151,440K physical used for   686,968K in swap (2,097,148K total swap)
   Tuning: 256 (large 512), oom   322,560K, restore limit   107,520K (high-end-gfx)         
```











### target app

```shell
adb shell dumpsys meminfo com.ss.android.ugc.aweme
```

用于获取指定应用程序（例如 `com.ss.android.ugc.aweme`）的内存使用情况信息。以下是该命令的输出内容及其含义解释：

#### 基本信息

- **Uptime**: 应用程序自启动以来的运行时间。
- **Realtime**: 设备自启动以来的实际时间。

#### 内存使用情况

- 总内存使用情况

  （单位：KB）:

  - **Pss**: 进程总体的Pss（Proportional Set Size），包括共享库等共享内存区域。
  - **Private Dirty**: 进程私有的Dirty内存。
  - **Private Clean**: 进程私有的Clean内存。
  - **SwapPss**: 已被交换到交换空间的Pss。
  - **Rss**: 进程的Resident Set Size，包括共享库在内的所有物理内存。

#### 内存区域详细信息

- **Native Heap**: 应用程序使用的本地堆内存。
- **Dalvik Heap**: 应用程序使用的Dalvik堆内存。
- **Dalvik Other**: Dalvik堆之外的其他内存。
- **Stack**: 栈内存。
- **Ashmem**: 匿名共享内存。
- **Gfx dev**: 图形设备相关内存。
- **Other dev**: 其他设备相关内存。
- **.so mmap**: 应用程序加载的共享库内存映射。
- **.jar mmap, .apk mmap, .ttf mmap, .dex mmap, .oat mmap, .art mmap**: 各种文件类型的内存映射。
- **Other mmap**: 其他类型的内存映射。
- **EGL mtrack, GL mtrack**: EGL和OpenGL相关内存。
- **Unknown**: 未知类型内存。

#### 应用程序摘要

- **Java Heap**: Java堆内存使用情况。
- **Native Heap**: 本地堆内存使用情况。
- **Code**: 应用程序代码占用的内存。
- **Stack**: 栈内存使用情况。
- **Graphics**: 图形相关内存使用情况。
- **Private Other**: 其他私有内存使用情况。
- **System**: 系统占用的内存。
- **Unknown**: 未知类型的内存使用情况。

#### 对象统计

- **Views, ViewRootImpl, AppContexts, Activities, Assets, AssetManagers**: 应用程序中的各种对象统计信息。

#### SQL和数据库

- **MEMORY_USED**: SQL数据库使用的内存量。
- **PAGECACHE_OVERFLOW**: 页面缓存溢出量。
- **MALLOC_SIZE**: 分配的内存大小。

#### 数据库信息

- 列出应用程序中打开的数据库的详细信息，包括数据库路径、页大小、数据库大小等。

通过分析这些信息，开发者可以了解应用程序在设备上的内存使用情况，帮助定位内存泄漏、优化内存使用和提升应用程序的整体性能



#### target process

```shell
dumpsys meminfo [pid]
```







## 查看应用启动activity

```shell
#手动启动目标app，并通过指令查看刚刚启动的应用启动activity
dumpsys activity recents
#通过指令启动目标app：am start -W {应用启动activity}
am start -W tv.danmaku.bili/.MainActivityV2
```

看`Recent #0`中的`intent`,在下面的输出中，可以看出bilibili的启动activity是`tv.danmaku.bili/.MainActivityV2`

```shell
blueline:/ # dumpsys activity recents
ACTIVITY MANAGER RECENT TASKS (dumpsys activity recents)
mRecentsUid=10078
mRecentsComponent=ComponentInfo{com.android.launcher3/com.android.quickstep.RecentsActivity}
mFreezeTaskListReordering=false
mFreezeTaskListReorderingPendingTimeout=false
  Recent tasks:
  * Recent #0: Task{f8619f5 #14001 type=standard A=10097:tv.danmaku.bili U=0 visible=true mode=fullscreen translucent=false sz=1}
    userId=0 effectiveUid=u0a97 mCallingUid=u0a78 mUserSetupComplete=true mCallingPackage=com.android.launcher3 mCallingFeatureId=null
    affinity=10097:tv.danmaku.bili
    intent={act=android.intent.action.MAIN cat=[android.intent.category.LAUNCHER] flg=0x10200000 cmp=tv.danmaku.bili/.MainActivityV2} # 启动activity
    mActivityComponent=tv.danmaku.bili/.MainActivityV2
    autoRemoveRecents=false isPersistable=true activityType=1
    rootWasReset=true mNeverRelinquishIdentity=true mReuseTask=false mLockTaskAuth=LOCK_TASK_AUTH_PINNABLE
    Activities=[ActivityRecord{bef5d94 u0 tv.danmaku.bili/.MainActivityV2 t14001}]
    askedCompatMode=false inRecents=true isAvailable=true
    mRootProcess=ProcessRecord{6fc8032 18440:tv.danmaku.bili/u0a97}
    taskId=14001 rootTaskId=14001
    hasChildPipActivity=false
    mHasBeenVisible=true
    mResizeMode=RESIZE_MODE_RESIZEABLE_VIA_SDK_VERSION mSupportsPictureInPicture=false isResizeable=true
    lastActiveTime=1031538663 (inactive for 893s)
```







## refs

[1] Gracker, Android Systrace 基础知识(8) - Vsync-App ：基于 Choreographer 的渲染机制详解.  https://mp.weixin.qq.com/s?__biz=MzIwNTQxMjM5MA%3D%3D&chksm=9730020fa0478b19cdeb8e3a0274111ed63f777309ae202ffb3ac15e50e7eb41f7a3737cce6d&idx=1&mid=2247484230&scene=21&sn=04767b1209bc0696595d908676423cb0#wechat_redirect  2020.06.03