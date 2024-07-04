## 简介





## gfxinfo

```shell
adb shell dumpsys gfxinfo com.ss.android.ugc.aweme
```

提供了关于指定Android应用程序（例如`com.ss.android.ugc.aweme`）图形渲染的详细信息

以下是这个命令通常输出的内容及其含义：

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



## meminfo

```shell
adb shell dumpsys meminfo com.ss.android.ugc.aweme
```

用于获取指定应用程序（例如 `com.ss.android.ugc.aweme`）的内存使用情况信息。以下是该命令的输出内容及其含义解释：

### 基本信息

- **Uptime**: 应用程序自启动以来的运行时间。
- **Realtime**: 设备自启动以来的实际时间。

### 内存使用情况

- 总内存使用情况

  （单位：KB）:

  - **Pss**: 进程总体的Pss（Proportional Set Size），包括共享库等共享内存区域。
  - **Private Dirty**: 进程私有的Dirty内存。
  - **Private Clean**: 进程私有的Clean内存。
  - **SwapPss**: 已被交换到交换空间的Pss。
  - **Rss**: 进程的Resident Set Size，包括共享库在内的所有物理内存。

### 内存区域详细信息

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

### 应用程序摘要

- **Java Heap**: Java堆内存使用情况。
- **Native Heap**: 本地堆内存使用情况。
- **Code**: 应用程序代码占用的内存。
- **Stack**: 栈内存使用情况。
- **Graphics**: 图形相关内存使用情况。
- **Private Other**: 其他私有内存使用情况。
- **System**: 系统占用的内存。
- **Unknown**: 未知类型的内存使用情况。

### 对象统计

- **Views, ViewRootImpl, AppContexts, Activities, Assets, AssetManagers**: 应用程序中的各种对象统计信息。

### SQL和数据库

- **MEMORY_USED**: SQL数据库使用的内存量。
- **PAGECACHE_OVERFLOW**: 页面缓存溢出量。
- **MALLOC_SIZE**: 分配的内存大小。

### 数据库信息

- 列出应用程序中打开的数据库的详细信息，包括数据库路径、页大小、数据库大小等。

通过分析这些信息，开发者可以了解应用程序在设备上的内存使用情况，帮助定位内存泄漏、优化内存使用和提升应用程序的整体性能