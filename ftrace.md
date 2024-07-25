# ftrace

## 1.定义trace event

trace event 的定义在`KERNEL_PATH/include/trace/events/*.h`中， 比如与内存管理的相关的在`vmscan.h`中，定义trace event 有两种方式，

- 一次定义一个函数`DEFINE_EVENT`

- 定义一个函数类`DECLARE_EVENT_CLASS`,通过函数类批量定义相同参数与相同打印格式的函数

  

### DECLARE_EVENT_CLASS

定义一个`kswapd_urgc`类，并通过`kswapd_urgc`类定义名称为`balance_pgdat_urgc`, `reclaim_clean_pages_from_list_urgc`的trace event; 通过在trace event的名称前加上`trace_`前缀在指定的函数中插入trace event

```c
 DECLARE_EVENT_CLASS(kswapd_urgc,
    TP_PROTO(int priority, unsigned long nr_scanned, unsigned long nr_reclaimed, unsigned long nr_reclaimed_anon, unsigned long nr_reclaimed_file, unsigned long nr_reclaimed_incative_anon_urgc, unsigned long nr_rotated_active_anon_urgc, 
	int order),

    TP_ARGS(priority, nr_scanned, nr_reclaimed, nr_reclaimed_anon, nr_reclaimed_file, nr_reclaimed_incative_anon_urgc, nr_rotated_active_anon_urgc, order),

    TP_STRUCT__entry(
        __field(int, priority)
        __field(unsigned long, nr_scanned)
		__field(unsigned long, nr_reclaimed)
		__field(unsigned long, nr_reclaimed_anon)
		__field(unsigned long, nr_reclaimed_file)
		__field(unsigned long, nr_reclaimed_incative_anon_urgc)
		__field(unsigned long, nr_rotated_active_anon_urgc)
        __field(int, order)
        ),

    TP_fast_assign(
        __entry->priority	= priority;
        __entry->nr_scanned	= nr_scanned;
		__entry->nr_reclaimed	= nr_reclaimed;
		__entry->nr_reclaimed_anon	= nr_reclaimed_anon;
		__entry->nr_reclaimed_file	= nr_reclaimed_file;
		__entry->nr_reclaimed_incative_anon_urgc	= nr_reclaimed_incative_anon_urgc;
		__entry->nr_rotated_active_anon_urgc	= nr_rotated_active_anon_urgc;
        __entry->order	= order;
        ),

    TP_printk("%d, %lu, %lu, %lu, %lu, %lu, %lu, %d",
        __entry->priority,
        __entry->nr_scanned,
		__entry->nr_reclaimed,
		__entry->nr_reclaimed_anon,
		__entry->nr_reclaimed_file,
		__entry->nr_reclaimed_incative_anon_urgc,
		__entry->nr_rotated_active_anon_urgc,
		__entry->order)
);
 DEFINE_EVENT(kswapd_urgc, balance_pgdat_urgc,
     TP_PROTO(int priority, unsigned long nr_scanned, unsigned long nr_reclaimed, unsigned long nr_reclaimed_anon, unsigned long nr_reclaimed_file, unsigned long nr_reclaimed_incative_anon_urgc, unsigned long nr_rotated_active_anon_urgc, int order),
     TP_ARGS(priority, nr_scanned, nr_reclaimed, nr_reclaimed_anon, nr_reclaimed_file, nr_reclaimed_incative_anon_urgc, nr_rotated_active_anon_urgc, order)
 );

 DEFINE_EVENT(kswapd_urgc, reclaim_clean_pages_from_list_urgc, // call in __alloc_contig_migrate_range
     TP_PROTO(int priority, unsigned long nr_scanned, unsigned long nr_reclaimed, unsigned long nr_reclaimed_anon, unsigned long nr_reclaimed_file, unsigned long nr_reclaimed_incative_anon_urgc, unsigned long nr_rotated_active_anon_urgc, int order),
     TP_ARGS(priority, nr_scanned, nr_reclaimed, nr_reclaimed_anon, nr_reclaimed_file, nr_reclaimed_incative_anon_urgc, nr_rotated_active_anon_urgc, order)
 );
```







## 2.插入trace event

在`KERNEL_PATH/mm/vmscan.c`中插入名字为`balance_pgdat_urgc`的trace event

```c
#include <trace/events/vmscan.h> 

static int balance_pgdat(pg_data_t *pgdat, int order, int highest_zoneidx)
{
...
	trace_balance_pgdat_urgc(sc.priority, sc.nr_scanned, sc.nr_reclaimed, sc.nr_reclaimed_anon, sc.nr_reclaimed_file, sc.nr_reclaimed_incative_anon_urgc, sc.nr_rotated_active_anon_urgc, sc.order);
...
	return sc.order;
}
```





## 3.打印trace event

重新编译内核，刷机之后查看trace event 是否定义成功，如果定义成功，通过adb shell 进入手机，`/sys/kernel/debug/tracing/events/`或者`/sys/kernel/tracing/events/`路径下找到定义的trace event。以pixel 3 android12 为例，我们在`KERNEL_PATH/include/trace/events/vmscan.h`文件中定义了balance_pgdat_urgc，则在`/sys/kernel/debug/tracing/events/vmscan`路径下可以找到该trace event

```shell
blueline:/sys/kernel/debug/tracing/events/vmscan # ls -lat | grep balance_pgdat_urgc
drwxr-xr-x  2 root root 0 1970-01-01 08:00 balance_pgdat_urgc
```



### 开启trace event

```shell
    def startRWFtrace(self):
        self.adb.run('shell "echo > /sys/kernel/debug/tracing/trace"')  # clear trace
        self.adb.run('shell "echo 96000 > /sys/kernel/debug/tracing/buffer_size_kb"')# set ftrace buffer
        # set r/w events
        self.adb.run('shell "echo 1 > /sys/kernel/debug/tracing/tracing_on"')
        self.adb.run('shell "echo 1 > /sys/kernel/debug/tracing/events/vmscan/reclaim_pages_urgc/enable"') 
        # start ftrace
        self.adb.run('shell "echo 1 > /sys/kernel/debug/tracing/tracing_on"')
        return
```

#### 关闭 trace event，并打印ftrace log

```shell
    # stop ftrace, clear events, save read/write ops, the latency is about 1s
    def endRWFtrace(self,ftrace_path, fg_app_name, ops, bg_app_num, turns):
        # stop ftrace, clear events,
        starttime = time.time()
        self.adb.run('shell "echo 0 > /sys/kernel/debug/tracing/tracing_on"')
        self.adb.run('shell "echo 0 > /sys/kernel/debug/tracing/events/vmscan/balance_pgdat_urgc/enable"')#todo for debug
        self.adb.run('shell "cat /sys/kernel/debug/tracing/trace > /data/local/tmp/%s_%sRWFtrace.txt"' % (fg_app_name, ops))  # save read/write ops
        # pull trace from mobile to local
        dest = time.strftime("%Y%m%d%H%M%S", time.localtime())
        self.adb.run('pull /data/local/tmp/%s_%sRWFtrace.txt  %s/%s/%s_RWFtrace_bg_app_num%d_%d_%s.txt' % (fg_app_name, ops, ftrace_path, fg_app_name, ops, bg_app_num, turns, dest))
        self.adb.run('shell "echo > /sys/kernel/debug/tracing/trace"')  # clear trace
        return
```

