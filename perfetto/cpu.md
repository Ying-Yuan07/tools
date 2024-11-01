## 简介[1]





## 1.Config

`config_cpu.pbtx`

```.pbtx
buffers: {
    size_kb: 522240
    fill_policy: DISCARD
}

buffers: {
    size_kb: 2048
    fill_policy: DISCARD
}
data_sources: {
    config {
        name: "android.packages_list"
        target_buffer: 1
    }
}
data_sources: {
    config {
        name: "linux.system_info"
        target_buffer: 1
    }
}
data_sources: {
    config {
        name: "linux.process_stats"
        target_buffer: 1
        process_stats_config {
            scan_all_processes_on_start: true
        }
    }
}
data_sources: {
    config {
        name: "linux.sys_stats"
        sys_stats_config {
            stat_period_ms: 250
            stat_counters: STAT_CPU_TIMES
            stat_counters: STAT_FORK_COUNT
        }
    }
}
data_sources: {
    config {
        name: "linux.ftrace"
        ftrace_config {
            ftrace_events: "sched/sched_switch"
            ftrace_events: "power/suspend_resume"
            ftrace_events: "sched/sched_wakeup"
            ftrace_events: "sched/sched_wakeup_new"
            ftrace_events: "sched/sched_waking"
            ftrace_events: "power/cpu_frequency"
            ftrace_events: "power/cpu_idle"
            ftrace_events: "sched/sched_process_exit"
            ftrace_events: "sched/sched_process_free"
            ftrace_events: "task/task_newtask"
            ftrace_events: "task/task_rename"
        }
    }
}

duration_ms: 60000
```



## 2. SQL

### 所有信息

```sql
select ts, dur, cpu, end_state, priority, process.name, thread.name
from sched_slice left join thread using(utid) left join process using(upid)
```

| ts            | dur     | cpu  | end_state | priority | name                                          | name_1          |
| ------------- | ------- | ---- | --------- | -------- | --------------------------------------------- | --------------- |
| 3609159120213 | 124479  | 1    | S         | 97       | /system/bin/surfaceflinger                    | TimerDispatch   |
| 3609159216776 | 1142604 | 3    | S         | 97       | /system/bin/surfaceflinger                    | surfaceflinger  |
| 3609159241984 | 122813  | 7    | S         | 104      | com.ss.android.ugc.aweme                      | HwBinder:7063_1 |
| 3609159244692 | 182136  | 1    | S         | 130      | /system/bin/logd                              | logd.writer     |
| 3609159364797 | 392604  | 7    | R         | 120      | NULL                                          | swapper         |
| 3609159426828 | 167448  | 1    | R+        | 130      | /system/bin/logd                              | logd.reader.per |
| 3609159467713 | 100052  | 0    | S         | 120      | /system/bin/traced                            | traced          |
| 3609159567765 | 179480  | 0    | R+        | 120      | /system/bin/traced_probes                     | traced_probes   |
| 3609159594276 | 231979  | 1    | S         | 96       | /vendor/bin/hw/android.hardware.audio.service | writer          |







```sql
select process.name as process, thread.name as thread, sum(dur) as cpu_dur 
            from sched_slice 
            inner join thread using(utid) 
            inner join process using(upid) group by utid order by cpu_dur desc
```

| process                    | thread         | cpu_dur      |
| -------------------------- | -------------- | ------------ |
| NULL                       | swapper        | 340872882260 |
| com.ss.android.ugc.aweme   | main           | 13719472362  |
| com.ss.android.ugc.aweme   | RenderThread   | 10883488641  |
| /system/bin/surfaceflinger | surfaceflinger | 6254911462   |





### python



#### colab

不需要设置config，colab会自动从谷歌官网下载 Trace Processor：trace_processor_shell.exe

#### windows

（1）翻墙，脚本运行时自动从谷歌官网下载 Trace Processor：trace_processor_shell.exe**

（2）使用本地 Trace Processor，

- 手动下载`https://commondatastorage.googleapis.com/perfetto-luci-artifacts/v47.0/windows-amd64/trace_processor_shell.exe`，或者从` https://github.com/google/perfetto/releases`下载 windows-amd64.zip，解压之后获取trace_processor_shell.exe

- 下载完成后，将trace_processor_shell.exe的路径设置为 `TraceProcessorConfig` 的 `bin_path` 参数，从而避免从远程下载

  

```shell
config = TraceProcessorConfig(bin_path='path_to_local_trace_processor_shell.exe')
tp = TraceProcessor(config=config)
```
