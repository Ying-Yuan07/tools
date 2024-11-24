# ftrace



SQL

**查询 ftrace 数据**
Perfetto 将 ftrace 数据以 `raw` 表的形式存储。以下是一些常用的 SQL 查询示例：

- **查看 raw 表的内容**

  ```
  
  SELECT * FROM raw LIMIT 100;
  ```

  该查询会显示 `raw` 表中的前 100 行数据，其中包括事件类型和详细内容。

- **过滤特定事件类型**
  如果想要查看特定的 ftrace 事件类型，比如 `sched_switch`，可以使用以下查询：

  ```
  SELECT * FROM raw WHERE name = 'swap_readpage_urgc_2' LIMIT 100;
  ```

  该查询会显示所有 `sched_switch` 事件。

#### Query result (100 rows) - 3.9ms

SELECT * FROM raw WHERE name = 'swap_readpage_urgc_2' LIMIT 100;

Copy queryCopy result (.tsv)

| id    | type                     | ts             | name                 | cpu  | utid | arg_set_id | common_flags | ucpu |
| ----- | ------------------------ | -------------- | -------------------- | ---- | ---- | ---------- | ------------ | ---- |
| 23484 | __intrinsic_ftrace_event | 45034315581688 | swap_readpage_urgc_2 | 0    | 183  | 5472       | 0            | 0    |
| 23490 | __intrinsic_ftrace_event | 45034315616376 | swap_readpage_urgc_2 | 0    | 183  | 5475       | 0            | 0    |
| 23493 | __intrinsic_ftrace_event | 45034315644553 | swap_readpage_urgc_2 | 0    | 183  | 5478       | 0            | 0    |
| 23495 | __intrinsic_ftrace_event | 45034315677261 | swap_readpage_urgc_2 | 0    | 183  | 5479       | 0            | 0    |

`id`: 事件的唯一标识符。

`type`: 事件类型，`__intrinsic_ftrace_event` 代表这是一个原始的 ftrace 事件。

`ts`: 时间戳，表示事件发生的时间。

`name`: 事件名称，例如 `swap_readpage_urgc_2` 可能表示一个页交换（swap）读取的函数。

`cpu`: 事件发生的 CPU 核心编号。

`utid`: 用户线程 ID。

`arg_set_id`: 表示事件参数集的标识符，可以用于关联更多的事件参数信息。

`common_flags`: 通用标志信息。

`ucpu`: 用户 CPU 的标识。



如果你希望进一步查询有关特定 `arg_set_id` 的参数信息，可以通过 `args` 表来关联。以下是一个示例查询，用于获取特定 `arg_set_id` 对应的参数值：

```
SELECT * 
FROM args 
WHERE arg_set_id IN (5472, 5475);
```

| id    | type             | arg_set_id | flat_key         | key              | int_value  | string_value | real_value | value_type | display_value |
| ----- | ---------------- | ---------- | ---------------- | ---------------- | ---------- | ------------ | ---------- | ---------- | ------------- |
| 24136 | __intrinsic_args | 5472       | comm             | comm             | NULL       |              | NULL       | string     |               |
| 24137 | __intrinsic_args | 5472       | p_uid            | p_uid            | 10096      | NULL         | NULL       | int        | 10096         |
| 24138 | __intrinsic_args | 5472       | tgid             | tgid             | 20110      | NULL         | NULL       | int        | 20110         |
| 24139 | __intrinsic_args | 5472       | pid              | pid              | 20396      | NULL         | NULL       | int        | 20396         |
| 24140 | __intrinsic_args | 5472       | address          | address          | 1885790208 | NULL         | NULL       | int        | 1885790208    |
| 24141 | __intrinsic_args | 5472       | swap_info_index  | swap_info_index  | 0          | NULL         | NULL       | int        | 0             |
| 24142 | __intrinsic_args | 5472       | swap_slot_offset | swap_slot_offset | 22939      | NULL         | NULL       | int        | 22939         |



统计一段时间内的符合uid与pid的记录

```sql
SELECT COUNT(DISTINCT raw.arg_set_id) AS total_count
FROM raw
JOIN args ON raw.arg_set_id = args.arg_set_id
WHERE raw.name = 'swap_readpage_urgc_2'
  AND raw.ts BETWEEN 45044154678000 AND 45045287781111
  AND EXISTS (SELECT 1 FROM args a WHERE a.arg_set_id = raw.arg_set_id AND a.key = 'p_uid' AND a.int_value = 10096)
  AND EXISTS (SELECT 1 FROM args a WHERE a.arg_set_id = raw.arg_set_id AND a.key = 'pid' AND a.int_value = 20121)
  AND args.key IN ('comm', 'p_uid', 'pid', 'swap_info_index', 'swap_slot_offset', 'tgid');
```

