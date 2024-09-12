## 简介[1]





## 1.Config

`config_janks.pbtx`

```.pbtx
buffers: {
    size_kb: 522240
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
        name: "android.surfaceflinger.frametimeline"
    }
}
duration_ms: 60000
```



## 2. SQL

### 所有信息

```sql
SELECT 
    (SELECT COUNT(*) FROM (
        -- total_cnt
        SELECT 
            ts, 
            AVG(dur) AS avg_dur, 
            surface_frame_token AS app_token, 
            display_frame_token, 
            jank_type, 
            on_time_finish, 
            present_type, 
            layer_name, 
            process.name 
        FROM actual_frame_timeline_slice 
        LEFT JOIN process USING (upid) 
        GROUP BY display_frame_token
    ) AS total_cnt
    ) AS total_cnt,

    (SELECT COUNT(*) FROM (
        -- total_jank_cnt
        SELECT 
            ts, 
            AVG(dur) AS avg_dur, 
            surface_frame_token AS app_token, 
            display_frame_token, 
            jank_type, 
            on_time_finish, 
            present_type, 
            layer_name, 
            process.name 
        FROM actual_frame_timeline_slice 
        LEFT JOIN process USING (upid) 
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
    ) AS total_jank_cnt
    ) AS total_jank_cnt,

    (SELECT COUNT(*) FROM (
        -- altert_count
        SELECT 
            ts, 
            AVG(dur) AS avg_dur, 
            surface_frame_token AS app_token, 
            display_frame_token, 
            jank_type, 
            on_time_finish, 
            present_type, 
            layer_name, 
            process.name 
        FROM actual_frame_timeline_slice 
        LEFT JOIN process USING (upid) 
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
        HAVING AVG(dur) < 100000000
        ORDER BY ts
    ) AS altert_count
    ) AS altert_count,

    (SELECT COUNT(*) FROM (
        -- jank_count
        SELECT 
            ts, 
            AVG(dur) AS avg_dur, 
            surface_frame_token AS app_token, 
            display_frame_token, 
            jank_type, 
            on_time_finish, 
            present_type, 
            layer_name, 
            process.name 
        FROM actual_frame_timeline_slice 
        LEFT JOIN process USING (upid) 
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
        HAVING AVG(dur) >= 100000000
        ORDER BY ts
    ) AS jank_count
    ) AS jank_count,

    -- p99_latency
    (WITH frame_stats AS (
        SELECT 
            display_frame_token, 
            AVG(dur) AS avg_dur
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING (upid)
        GROUP BY display_frame_token
    ),
    total_count AS (
        SELECT COUNT(*) AS total_count
        FROM frame_stats
    ),
    ranked_frames AS (
        SELECT 
            display_frame_token, 
            avg_dur,
            ROW_NUMBER() OVER (ORDER BY avg_dur DESC) AS row_num,
            total_count.total_count
        FROM frame_stats, total_count
    ),
    p99_frames AS (
        SELECT avg_dur
        FROM ranked_frames
        WHERE row_num <= CAST(total_count * 0.01 AS INT)
    )
    SELECT AVG(avg_dur) AS avg_p99_dur
    FROM p99_frames
    ) AS avg_p99_dur,

    -- avg_jank_latency
    (SELECT AVG(avg_dur) AS average_avg_dur
    FROM (
        SELECT AVG(dur) AS avg_dur
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING (upid)
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
    ) AS subquery
    ) AS avg_jank_latency,

    -- avg_normal_latency
    (SELECT AVG(avg_dur) AS average_avg_dur
    FROM (
        SELECT AVG(dur) AS avg_dur
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING (upid)
        WHERE on_time_finish = 1
        GROUP BY display_frame_token
    ) AS subquery
    ) AS avg_normal_latency;

```



| total_cnt | total_jank_cnt | altert_count | jank_count | p99_latency(ns) | avg_jank_latency(ns) | avg_normal_latency(ns) |
| --------- | -------------- | ------------ | ---------- | --------------- | -------------------- | ---------------------- |
| 2580      | 52             | 30           | 22         | 611528690       | 119614205.1923077    | 10393380.784827318     |

- **total_cnt**: total frames count
- **total_jank_cnt**: altert_count+jank_count
- **altert_count**: total altert(16.6-100ms) frames count
- **jank_count**: total jank(>100ms) frames count





```sql
SELECT 
    (SELECT COUNT(*) FROM (
        -- total_cnt
        SELECT 
            ts, 
            AVG(dur) AS avg_dur, 
            surface_frame_token AS app_token, 
            display_frame_token, 
            jank_type, 
            on_time_finish, 
            present_type, 
            layer_name, 
            process.name 
        FROM actual_frame_timeline_slice 
        LEFT JOIN process USING (upid) 
        GROUP BY display_frame_token
    ) AS total_cnt
    ) AS total_cnt,

    (SELECT COUNT(*) FROM (
        -- total_jank_cnt
        SELECT 
            ts, 
            AVG(dur) AS avg_dur, 
            surface_frame_token AS app_token, 
            display_frame_token, 
            jank_type, 
            on_time_finish, 
            present_type, 
            layer_name, 
            process.name 
        FROM actual_frame_timeline_slice 
        LEFT JOIN process USING (upid) 
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
    ) AS total_jank_cnt
    ) AS total_jank_cnt,

    (SELECT COUNT(*) FROM (
        -- altert_count
        SELECT 
            ts, 
            AVG(dur) AS avg_dur, 
            surface_frame_token AS app_token, 
            display_frame_token, 
            jank_type, 
            on_time_finish, 
            present_type, 
            layer_name, 
            process.name 
        FROM actual_frame_timeline_slice 
        LEFT JOIN process USING (upid) 
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
        HAVING AVG(dur) < 100000000
        ORDER BY ts
    ) AS altert_count
    ) AS altert_count,

    (SELECT COUNT(*) FROM (
        -- jank_count
        SELECT 
            ts, 
            AVG(dur) AS avg_dur, 
            surface_frame_token AS app_token, 
            display_frame_token, 
            jank_type, 
            on_time_finish, 
            present_type, 
            layer_name, 
            process.name 
        FROM actual_frame_timeline_slice 
        LEFT JOIN process USING (upid) 
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
        HAVING AVG(dur) >= 100000000
        ORDER BY ts
    ) AS jank_count
    ) AS jank_count,

    -- p99_latency
    (WITH frame_stats AS (
        SELECT 
            display_frame_token, 
            AVG(dur) AS avg_dur
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING (upid)
        GROUP BY display_frame_token
    ),
    total_count AS (
        SELECT COUNT(*) AS total_count
        FROM frame_stats
    ),
    ranked_frames AS (
        SELECT 
            display_frame_token, 
            avg_dur,
            ROW_NUMBER() OVER (ORDER BY avg_dur DESC) AS row_num,
            total_count.total_count
        FROM frame_stats, total_count
    ),
    p99_frames AS (
        SELECT avg_dur
        FROM ranked_frames
        WHERE row_num <= CAST(total_count * 0.01 AS INT)
    )
    SELECT AVG(avg_dur) AS avg_p99_dur
    FROM p99_frames
    ) AS avg_p99_dur,

    -- avg_jank_latency
    (SELECT AVG(avg_dur) AS average_avg_dur
    FROM (
        SELECT AVG(dur) AS avg_dur
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING (upid)
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
    ) AS subquery
    ) AS avg_jank_latency,

    -- avg_normal_latency
    (SELECT AVG(avg_dur) AS average_avg_dur
    FROM (
        SELECT AVG(dur) AS avg_dur
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING (upid)
        WHERE on_time_finish = 1
        GROUP BY display_frame_token
    ) AS subquery
    ) AS avg_normal_latency;

```







### P99 

#### P99_list

```SQL
WITH frame_stats AS (
    SELECT 
        display_frame_token, 
        AVG(dur) AS avg_dur, 
        surface_frame_token AS app_token, 
        jank_type, 
        on_time_finish, 
        present_type, 
        layer_name, 
        process.name
    FROM actual_frame_timeline_slice
    LEFT JOIN process USING (upid)
    GROUP BY display_frame_token
),
total_count AS (
    SELECT COUNT(*) AS total_count
    FROM frame_stats
),
ranked_frames AS (
    SELECT 
        display_frame_token, 
        avg_dur, 
        app_token, 
        jank_type, 
        on_time_finish, 
        present_type, 
        layer_name, 
        name,
        ROW_NUMBER() OVER (ORDER BY avg_dur DESC) AS row_num,
        total_count.total_count
    FROM frame_stats, total_count
)
SELECT 
    display_frame_token, 
    avg_dur, 
    app_token, 
    jank_type, 
    on_time_finish, 
    present_type, 
    layer_name, 
    name
FROM ranked_frames
WHERE row_num <= CAST(total_count * 0.01 AS INT)
ORDER BY avg_dur DESC;
```

#### P99_avg

```sql
WITH frame_stats AS (
    SELECT 
        display_frame_token, 
        AVG(dur) AS avg_dur, 
        surface_frame_token AS app_token, 
        jank_type, 
        on_time_finish, 
        present_type, 
        layer_name, 
        process.name
    FROM actual_frame_timeline_slice
    LEFT JOIN process USING (upid)
    GROUP BY display_frame_token
),
total_count AS (
    SELECT COUNT(*) AS total_count
    FROM frame_stats
),
ranked_frames AS (
    SELECT 
        display_frame_token, 
        avg_dur, 
        app_token, 
        jank_type, 
        on_time_finish, 
        present_type, 
        layer_name, 
        name,
        ROW_NUMBER() OVER (ORDER BY avg_dur DESC) AS row_num,
        total_count.total_count
    FROM frame_stats, total_count
),
p99_frames AS (
    SELECT avg_dur
    FROM ranked_frames
    WHERE row_num <= CAST(total_count * 0.01 AS INT)
)
SELECT AVG(avg_dur) AS avg_p99_dur
FROM p99_frames;
```















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



```py
from perfetto.trace_processor import TraceProcessorConfig, TraceProcessor

# Enable verbose mode for detailed logs
# config = TraceProcessorConfig(verbose=True)

config = TraceProcessorConfig(bin_path='D:\\tools\\Android\\perfetto-windows-amd64\\trace_processor_shell.exe')

# Initialize the trace processor with the trace file
tp = TraceProcessor(trace='test.perfetto-trace', config=config)


SQL = '''
SELECT
    (SELECT COUNT(*) FROM (
        -- total_cnt
        SELECT ts, avg(dur) as avg_dur, surface_frame_token as app_token, display_frame_token, jank_type, on_time_finish, present_type, layer_name, process.name
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING(upid)
        GROUP BY display_frame_token
    ) AS total_cnt
    ) AS total_cnt,
    (SELECT COUNT(*) FROM (
        -- total_jank_cnt
        SELECT ts, avg(dur) as avg_dur, surface_frame_token as app_token, display_frame_token, jank_type, on_time_finish, present_type, layer_name, process.name
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING(upid)
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
    ) AS total_jank_cnt
    ) AS total_jank_cnt,
    (SELECT COUNT(*) FROM (
        -- altert_count
        SELECT ts, AVG(dur) AS avg_dur, surface_frame_token AS app_token, display_frame_token, jank_type, on_time_finish, present_type, layer_name, process.name
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING(upid)
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
        HAVING AVG(dur) < 100000000
        ORDER BY ts
    ) AS altert_count
    ) AS altert_count,
    (SELECT COUNT(*) FROM (
        -- jank_count
        SELECT ts, AVG(dur) AS avg_dur, surface_frame_token AS app_token, display_frame_token, jank_type, on_time_finish, present_type, layer_name, process.name
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING(upid)
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
        HAVING AVG(dur) >= 100000000
        ORDER BY ts
    ) AS jank_count
    ) AS jank_count,
     -- p99_latency
    (select  avg(dur) as avg_dur from actual_frame_timeline_slice
    left join process using(upid)
    where on_time_finish = 0
    group by display_frame_token
    order by avg_dur desc
    limit 1) as p99_latency,
    -- avg_jank_latency
    (SELECT AVG(avg_dur) AS average_avg_dur
    FROM (
        SELECT AVG(dur) AS avg_dur
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING (upid)
        WHERE on_time_finish = 0
        GROUP BY display_frame_token
    ) AS subquery
    ) AS avg_jank_latency,
     -- avg_normal_latency
    (SELECT AVG(avg_dur) AS average_avg_dur
    FROM (
        SELECT AVG(dur) AS avg_dur
        FROM actual_frame_timeline_slice
        LEFT JOIN process USING (upid)
        WHERE on_time_finish = 1
        GROUP BY display_frame_token
    ) AS subquery
    ) AS avg_normal_latency
'''


if __name__ == '__main__':
    # Query the 'slice' table
    qr_it = tp.query(SQL)
    for idx, row in enumerate(qr_it):
        row = '%s' % row
        row = eval(row)
        print(row)
```



```shell
Connected to pydev debugger (build 203.8084.11)
{'total_cnt': 1463, 'total_jank_cnt': 37, 'altert_count': 31, 'jank_count': 6, '
p99_latency': 335594593.0, 'avg_jank_latency': 76723447.72972973, 'avg_normal_la
tency': 10356563.05468216}
```





P95

```SQL
WITH ranked_frames AS (
    SELECT 
        display_frame_token, 
        AVG(dur) AS avg_dur, 
        surface_frame_token AS app_token, 
        jank_type, 
        on_time_finish, 
        present_type, 
        layer_name, 
        process.name,
        PERCENT_RANK() OVER (ORDER BY AVG(dur) DESC) AS percentile_rank
    FROM actual_frame_timeline_slice
    LEFT JOIN process USING (upid)
    WHERE on_time_finish = 0
    GROUP BY display_frame_token
)
SELECT 
    display_frame_token, 
    avg_dur, 
    app_token, 
    jank_type, 
    on_time_finish, 
    present_type, 
    layer_name, 
    name
FROM ranked_frames
WHERE percentile_rank >= 0.95
ORDER BY avg_dur DESC;

```



| display_frame_token | avg_dur  | app_token | jank_type                        | on_time_finish | present_type    | layer_name                                        | name |
| ------------------- | -------- | --------- | -------------------------------- | -------------- | --------------- | ------------------------------------------------- | ---- |
| 19981               | 21442499 | 19977     | Display HAL, App Deadline Missed | 0              | Late Present    | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 20252               | 20743765 | 20249     | None                             | 0              | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |



| 21919 | 6714050.5 | 21916 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| ----- | --------- | ----- | ---- | ---- | --------------- | ------------------------------------------------- | ---- |
| 21955 | 6712927.5 | 21952 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 20833 | 6708648   | 20830 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 22005 | 6706112   | 22002 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 20483 | 6704527.5 | 20480 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 20489 | 6701972.5 | 20486 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 22011 | 6696732.5 | 22008 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 21216 | 6696647.5 | 21213 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 21969 | 6696238   | 21966 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 21951 | 6693604   | 21948 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 20499 | 6688138   | 20496 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |
| 22001 | 6682210.5 | 21998 | None | 1    | On-time Present | TX - com.youku.phone/com.youku.v2.HomePageEntry#0 | NULL |













## refs

[1] https://perfetto.dev/docs/data-sources/frametimeline

