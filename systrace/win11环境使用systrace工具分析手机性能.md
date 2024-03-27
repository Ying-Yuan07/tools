# win11环境使用systrace工具分析手机性能

环境要求

python 2.7

## 1. Systrace工具下载与配置

1)到android官网下载最新版本的[platform-tools](https://developer.android.com/studio/releases/platform-tools),本环境用的`platform-tools_r31.0.3-windows`

2)将platform-tools安装解压，并将其解压后的路径配置到path环境变量，如下图所示。systrace 工具为platform-tools\systrace\systrace.py

![image-20230305113302622](win11环境使用systrace工具分析手机性能.assets/image-20230305113302622.png)

测试环境变量是否配置成功

打开PowerShell,输入指令adb,有adb指令即为配置成功

![image-20230305113530842](win11环境使用systrace工具分析手机性能.assets/image-20230305113530842.png)



## 2. 切换python版本

systrace.py的python版本为2.7

1）到python[官网](https://www.python.org/downloads/)下载python2.7,本环境用的[2.7.10](https://www.python.org/downloads/release/python-2710/)

2）将python包解压，本环境解压路径为`D:\Program Files\Python`

3）修改python名称：windows环境上可能存在多个python版本，为了便于切换，将python安装包解压后的路径修改为`D:\Program Files\Python`，将`D:\Program Files\Python\python.exe`修改为`D:\Program Files\Python\python27.exe`

4）配置python环境变量，在path环境变量中添加`D:\Program Files\Python27`与`D:\Program Files\Python27\Scripts`

![image-20230305114611095](win11环境使用systrace工具分析手机性能.assets/image-20230305114611095.png)



## 3. 其它环境

systrace.py的执行依赖pywin32与six包

1）pywin32安装：

​		方式1：`pip install pywin32`

​		[方式2](https://blog.csdn.net/weixin_43937959/article/details/123095273)：官网下载对应[python2.7版本](https://github.com/mhammond/pywin32/releases/download/b228/pywin32-228.win32-py2.7.exe)，手动安装

2） six安装：`pip install six`



## 4. Systrace工具[使用](https://blog.csdn.net/u011578734/article/details/109497064)[1]



`systrace.py`命令的一般用法是[2]：

> systrace.py [options] [category1 [category2 ...]]

其中，`[options]` 是一些命令参数，`[category]` 等是你感兴趣的系统模块，比如view代表view系统（包含绘制流程），am代表ActivityManager（包含Activity创建过程等）；分析不同问题的时候，可以选择不同你感兴趣的模块。需要重复的是，尽可能缩小需要Trace的模块，其一是数据量小易与分析；其二，虽然systrace本身开销很小，但是缩小需要Trace的模块也能减少运行时开销。比如你分析卡顿的时候，`power`,  `webview` 就几乎是无用的。



`[option]` 中比较重要的几个参数如下：

- **-a <package_name>**：这个选项可以开启指定包名App中自定义Trace Label的Trace功能。也就是说，如果你在代码中使用了`Trace.beginSection("tag")`, `Trace.endSection`；默认情况下，你的这些代码是不会生效的，因此，这个选项一定要开启！
- **-t N**：用来指定Trace运行的时间，取决于你需要分析过程的时间；还是那句话，在需要的时候尽可能缩小时间；当然，绝对不要把时间设的太短导致你操作没完Trace就跑完了，这样会出现`Did not finish` 的标签，分析数据就基本无效了。
-  **-l**：这个用来列出你分析的那个手机系统支持的Trace模块；也就是上面命令中 `[category1]`能使用的部分；不同版本的系统能支持的模块是不同的，一般来说，高版本的支持的模块更多。
- **-o FILE**：指定trace数据文件的输出路径，如果不指定就是当前目录的`trace.html`。





`systrace.py -l` 可以输出手机能支持的Trace模块，而且输出还给出了此模块的用途；常用的模块如下

`sched`: CPU调度的信息，非常重要；你能看到CPU在每个时间段在运行什么线程；线程调度情况，比如锁信息。

`gfx`：Graphic系统的相关信息，包括SerfaceFlinger，VSYNC消息，Texture，RenderThread等；分析卡顿非常依赖这个。

`view`: View绘制系统的相关信息，比如onMeasure，onLayout等；对分析卡顿比较有帮助。

`am`：ActivityManager调用的相关信息；用来分析Activity的启动过程比较有效。

`dalvik`: 虚拟机相关信息，比如GC停顿等。

`binder_driver`: Binder驱动的相关信息，如果你怀疑是Binder IPC的问题，不妨打开这个。

```shell
PS G:\git\project\United-LRU-RGC\script\motivation> python27 F:\1_tools\platform-tools_r31.0.3-windows\systrace\systrace.py -l
         gfx - Graphics
       input - Input
        view - View System
     webview - WebView
          wm - Window Manager
          am - Activity Manager
          sm - Sync Manager
       audio - Audio
       video - Video
      camera - Camera
         hal - Hardware Modules
         res - Resource Loading
      dalvik - Dalvik VM
          rs - RenderScript
      bionic - Bionic C Library
       power - Power Management
          pm - Package Manager
          ss - System Server
    database - Database
     network - Network
         adb - ADB
    vibrator - Vibrator
        aidl - AIDL calls
       nnapi - NNAPI
         rro - Runtime Resource Overlay
         pdx - PDX services
       sched - CPU Scheduling
         irq - IRQ Events
         i2c - I2C Events
        freq - CPU Frequency
        idle - CPU Idle
        disk - Disk I/O
         mmc - eMMC commands
        sync - Synchronization
       workq - Kernel Workqueues
  memreclaim - Kernel Memory Reclaim
  regulators - Voltage and Current Regulators
  binder_driver - Binder Kernel driver
  binder_lock - Binder global lock trace
   pagecache - Page cache
      memory - Memory
     thermal - Thermal event
        freq - CPU Frequency and System Clock (HAL)
         gfx - Graphics (HAL)
         ion - ION Allocation (HAL)
      memory - Memory (HAL)
       sched - CPU Scheduling and Trustzone (HAL)
  thermal_tj - Tj power limits and frequency (HAL)
```



1）生成5s HTML报告：

```python
python27 F:\1_tools\platform-tools_r31.0.3-windows\systrace\systrace.py -t 5 -o F:/1_tools/systrace_data/youku_coldlaunch.html
```



![image-20230305165932205](win11环境使用systrace工具分析手机性能.assets/image-20230305165932205.png)

2）用chrome打开生成的html文件

![image-20230305170859932](win11环境使用systrace工具分析手机性能.assets/image-20230305170859932.png)



## 5 [只抓取指定用户进程的trace](https://zhuanlan.zhihu.com/p/27331842)



```shell
 python27 F:\1_tools\platform-tools_r31.0.3-windows\systrace\systrace.py -t 600 memory pagecache memreclaim workq sync mmc disk idle freq irq rro database ss dalvik sched gfx view wm am binder_driver -a com.taobao.taobao -o taobao-600s.html
```











## refs

[1] https://blog.csdn.net/u011578734/article/details/109497064

[2] weishu,手把手教你使用Systrace（一）,https://zhuanlan.zhihu.com/p/27331842 , 2017.06.10.

