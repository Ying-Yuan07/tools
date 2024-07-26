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
        self.adb.run('shell "echo 1 > /sys/kernel/debug/tracing/events/vmscan/balance_pgdat_urgc/enable"') 
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



## **4. 函数调用链及其执行时间function&function_graph**[1]

**（1）function 主要用于跟踪内核函数的调用栈（其被调用过程）**

**（2）function_graph 主要用于跟踪内核函数内部调用流程及耗时**

切换到root，查看可用的tracer list，在服务器端支持function_graph，在pixel3/6上只支持nop

```shell
/sys/kernel/debug/tracing # cat available_tracers
timerlat osnoise hwlat blk mmiotrace function_graph wakeup_dl wakeup_rt wakeup function nop
```

下面以服务器5.15.0-113-generic，Ubuntu22.04为例

### function

```shell
/sys/kernel/debug/tracing # echo nop > current_tracer           ----清空跟踪器，与跟踪结果
/sys/kernel/debug/tracing # echo function > current_tracer      ----设置当前跟踪器为function
/sys/kernel/debug/tracing # echo do_fault > set_ftrace_filter   ----设置跟踪函数为do_fault
/sys/kernel/debug/tracing # echo 1 > options/func_stack_trace   ----跟踪函数调用栈
/sys/kernel/debug/tracing # echo 1 > tracing_on                 ----开始跟踪
/sys/kernel/debug/tracing # echo 0 > tracing_on                 ----关闭跟踪
/sys/kernel/debug/tracing # cat trace                           ----查看跟踪结果
```



### function_graph

```shell
#切换到root
/sys/kernel/debug/tracing # echo nop > current_tracer            ----清空跟踪器，与跟踪结果
/sys/kernel/debug/tracing # echo function_graph > current_tracer ----设置当前跟踪器为function_graph
/sys/kernel/debug/tracing # echo do_fault > set_graph_function   ----设置跟踪函数为do_fault，多个函数用空格间隔
/sys/kernel/debug/tracing # echo 1 > options/funcgraph-tail 	 ----增加函数尾部注释
/sys/kernel/debug/tracing # echo 1 > tracing_on 				 ----开始跟踪
/sys/kernel/debug/tracing # echo 0 > tracing_on 				 ----关闭跟踪
/sys/kernel/debug/tracing # cat trace 							 ----查看跟踪结果
```

结果

```
# CPU  DURATION                  FUNCTION CALLS
# |     |   |                     |   |   |   |
  7)               |  do_fault() {
  7)               |    do_read_fault() {
  7)               |      filemap_map_pages() {
  7)   0.564 us    |        next_uptodate_page();
  7)   0.181 us    |        filemap_map_pmd();
  7)   0.172 us    |        _raw_spin_lock();
  7)   0.166 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.168 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.166 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.167 us    |              rcu_read_unlock_strict();
  7)   0.494 us    |            } /* unlock_page_memcg */
  7)   1.151 us    |          } /* page_add_file_rmap */
  7)   1.838 us    |        } /* do_set_pte */
  7)   0.171 us    |        unlock_page();
  7)   0.414 us    |        next_uptodate_page();
  7)   0.170 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.171 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.197 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.166 us    |              rcu_read_unlock_strict();
  7)   0.488 us    |            } /* unlock_page_memcg */
  7)   1.175 us    |          } /* page_add_file_rmap */
  7)   1.840 us    |        } /* do_set_pte */
  7)   0.167 us    |        unlock_page();
  7)   0.599 us    |        next_uptodate_page();
  7)   0.170 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.170 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.306 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.170 us    |              rcu_read_unlock_strict();
  7)   0.497 us    |            } /* unlock_page_memcg */
  7)   1.294 us    |          } /* page_add_file_rmap */
  7)   1.978 us    |        } /* do_set_pte */
  7)   0.166 us    |        unlock_page();
  7)   0.407 us    |        next_uptodate_page();
  7)   0.169 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.172 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.168 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.171 us    |              rcu_read_unlock_strict();
  7)   0.494 us    |            } /* unlock_page_memcg */
  7)   1.152 us    |          } /* page_add_file_rmap */
  7)   1.817 us    |        } /* do_set_pte */
  7)   0.169 us    |        unlock_page();
  7)   0.399 us    |        next_uptodate_page();
  7)   0.169 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.171 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.168 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.193 us    |              rcu_read_unlock_strict();
  7)   0.515 us    |            } /* unlock_page_memcg */
  7)   1.166 us    |          } /* page_add_file_rmap */
  7)   1.830 us    |        } /* do_set_pte */
  7)   0.167 us    |        unlock_page();
  7)   0.402 us    |        next_uptodate_page();
  7)   0.165 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.166 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.164 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.168 us    |              rcu_read_unlock_strict();
  7)   0.488 us    |            } /* unlock_page_memcg */
  7)   1.140 us    |          } /* page_add_file_rmap */
  7)   1.796 us    |        } /* do_set_pte */
  7)   0.165 us    |        unlock_page();
  7)   0.191 us    |        next_uptodate_page();
  7)   0.164 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.167 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.163 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.167 us    |              rcu_read_unlock_strict();
  7)   0.490 us    |            } /* unlock_page_memcg */
  7)   1.132 us    |          } /* page_add_file_rmap */
  7)   1.813 us    |        } /* do_set_pte */
  7)   0.167 us    |        unlock_page();
  7)   0.397 us    |        next_uptodate_page();
  7)   0.164 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.168 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.166 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.166 us    |              rcu_read_unlock_strict();
  7)   0.487 us    |            } /* unlock_page_memcg */
  7)   1.133 us    |          } /* page_add_file_rmap */
  7)   1.788 us    |        } /* do_set_pte */
  7)   0.167 us    |        unlock_page();
  7)   0.409 us    |        next_uptodate_page();
  7)   0.166 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.165 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.166 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.165 us    |              rcu_read_unlock_strict();
  7)   0.484 us    |            } /* unlock_page_memcg */
  7)   1.136 us    |          } /* page_add_file_rmap */
  7)   1.795 us    |        } /* do_set_pte */
  7)   0.296 us    |        unlock_page();
  7)   0.520 us    |        next_uptodate_page();
  7)   0.165 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.166 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.167 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.163 us    |              rcu_read_unlock_strict();
  7)   0.485 us    |            } /* unlock_page_memcg */
  7)   1.130 us    |          } /* page_add_file_rmap */
  7)   1.871 us    |        } /* do_set_pte */
  7)   0.166 us    |        unlock_page();
  7)   0.575 us    |        next_uptodate_page();
  7)   0.164 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.165 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.163 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.163 us    |              rcu_read_unlock_strict();
  7)   0.504 us    |            } /* unlock_page_memcg */
  7)   1.143 us    |          } /* page_add_file_rmap */
  7)   1.799 us    |        } /* do_set_pte */
  7)   0.166 us    |        unlock_page();
  7)   0.409 us    |        next_uptodate_page();
  7)   0.163 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.167 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.165 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.168 us    |              rcu_read_unlock_strict();
  7)   0.490 us    |            } /* unlock_page_memcg */
  7)   1.138 us    |          } /* page_add_file_rmap */
  7)   1.790 us    |        } /* do_set_pte */
  7)   0.169 us    |        unlock_page();
  7)   0.399 us    |        next_uptodate_page();
  7)   0.166 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.168 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.167 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.191 us    |              rcu_read_unlock_strict();
  7)   0.509 us    |            } /* unlock_page_memcg */
  7)   1.162 us    |          } /* page_add_file_rmap */
  7)   1.826 us    |        } /* do_set_pte */
  7)   0.171 us    |        unlock_page();
  7)   0.397 us    |        next_uptodate_page();
  7)   0.168 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.169 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.166 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.164 us    |              rcu_read_unlock_strict();
  7)   0.479 us    |            } /* unlock_page_memcg */
  7)   1.125 us    |          } /* page_add_file_rmap */
  7)   1.786 us    |        } /* do_set_pte */
  7)   0.165 us    |        unlock_page();
  7)   0.388 us    |        next_uptodate_page();
  7)   0.161 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.166 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.168 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.165 us    |              rcu_read_unlock_strict();
  7)   0.514 us    |            } /* unlock_page_memcg */
  7)   1.166 us    |          } /* page_add_file_rmap */
  7)   1.820 us    |        } /* do_set_pte */
  7)   0.197 us    |        unlock_page();
  7)   0.396 us    |        next_uptodate_page();
  7)   0.169 us    |        PageHuge();
  7)               |        do_set_pte() {
  7)   0.167 us    |          add_mm_counter_fast();
  7)               |          page_add_file_rmap() {
  7)   0.171 us    |            lock_page_memcg();
  7)               |            unlock_page_memcg() {
  7)   0.169 us    |              rcu_read_unlock_strict();
  7)   0.703 us    |            } /* unlock_page_memcg */
  7)   1.620 us    |          } /* page_add_file_rmap */
  7)   2.278 us    |        } /* do_set_pte */
  7)   0.163 us    |        unlock_page();
  7)   0.170 us    |        next_uptodate_page();
  7)   0.169 us    |        rcu_read_unlock_strict();
  7) + 54.590 us   |      } /* filemap_map_pages */
  7) + 54.999 us   |    } /* do_read_fault */
  7) + 55.337 us   |  } /* do_fault */
```



### 过滤器

```shell
通用    buffer_total_size_kb    显示所有CPU ring buffer 大小之和
通用    trace_options    trace 过程的复杂控制选项 控制Trace打印内容或者操作跟踪器 也可通过 options/目录设置
通用    options/    显示 trace_option 的设置结果
也可以直接设置，作用同 trace_options
func    function_profile_enabled    打开此选项，trace_stat就会显示function的统计信息
echo 0/1 > function_profile_enabled
func    set_ftrace_pid    设置跟踪的pid
func    set_ftrace_filter    用于显示指定要跟踪的函数
func    set_ftrace_notrace    用于指定不跟踪的函数，缺省为空
graph    max_graph_depth    函数嵌套的最大深度
graph    set_graph_function    设置要清晰显示调用关系的函数
缺省对所有函数都生成调用关系
Stack    stack_max_size    当使用stack跟踪器时，记录产生过的最大stack size
Stack    stack_trace    显示stack的back trace
Stack    stack_trace_filter    设置stack tracer不检查的函数名称
```



## refs

[1] ftrace之function及function_graph使用 https://www.cnblogs.com/linhaostudy/p/17167739.html
