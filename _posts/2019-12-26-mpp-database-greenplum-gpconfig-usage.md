---
layout: post
title: GP参数修改gpconfig的使用:| greenplum
category: greenplum
tag: [greenplum]
---

这篇文章主要介绍了GP参数修改gpconfig的使用相关。
本文分为以下几个部分：
1. GPDB介绍
2. gpconfig使用


# 1.GPDB介绍
由于GPDB是分布式数据库，其参数文件分布于各个节点，如果每次参数修改都要到每个节点的配置文件上进行操作，那么效率太低，使得系统可维护性变差。所以，从4.0开始GREENPLUM就提供工具gpconfig，通过主节点就可以对所有节点的参数文件进行批量修改，极大方便了DBA的工作。

# 2.gpconfig使用

## 2.1 修改参数
比如要修改最大连接数的参数，可以以gpadmin用户登录master，执行如下命令:

```
gpconfig -c max_connection -v 750 -m 150
```

说明如下：

- (1) -c 指定要参数;
- (2) -v 指定segment 参数的设置值，如果没有-m参数，它也指定master上参数的设置，-m 如果希望master参数不同于segment，那么通过该参数独立指定master的参数值。

## 2.2 查询参数

可以使用指令gpconfig -s 查询参数，例如查看节点的最大连接数：

```
gpconfig -s max_connections 
```

## 2.3 注意事项

gpconfig只能在系统启动的情况下调用，所以如果参数修改不合适，导致系统无法启动时，我们可以用下列方法处理:
- 1、先把master的参数修改成正常的值
- 2、gpstart -m 仅启动master进入管理模式
- 3、gpconfig -r  <参数>   -- 把参数重置成默认值
- 4、gpstop -a -r -M fast

# 3.参考地址

【1】. https://yq.aliyun.com/articles/145445

# 4.greenplum6.2参数列表

```
                         name                         |                                     setting                                      |                                                                          description                                               
                            
------------------------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------
 application_name                                     | psql                                                                             | Sets the application name to be reported in statistics and logs.
 archive_mode                                         | off                                                                              | Allows archiving of WAL files using archive_command.
 array_nulls                                          | on                                                                               | Enable input of NULL elements in arrays.
 authentication_timeout                               | 1min                                                                             | Sets the maximum allowed time to complete client authentication.
 autovacuum_max_workers                               | 3                                                                                | Sets the maximum number of simultaneously running autovacuum worker processes.
 autovacuum_multixact_freeze_max_age                  | 400000000                                                                        | Multixact age at which to autovacuum a table to prevent multixact wraparound.
 autovacuum_work_mem                                  | -1                                                                               | Sets the maximum memory to be used by each autovacuum worker process.
 backslash_quote                                      | safe_encoding                                                                    | Sets whether "\'" is allowed in string literals.
 block_size                                           | 32768                                                                            | Shows the size of a disk block.
 bonjour                                              | off                                                                              | Enables advertising the server via Bonjour.
 bonjour_name                                         |                                                                                  | Sets the Bonjour service name.
 bytea_output                                         | hex                                                                              | Sets the output format for bytea.
 check_function_bodies                                | on                                                                               | Check function bodies during CREATE FUNCTION.
 checkpoint_completion_target                         | 0.5                                                                              | Time spent flushing dirty buffers during checkpoint, as fraction of checkpoint interval.
 client_encoding                                      | UTF8                                                                             | Sets the client's character set encoding.
 client_min_messages                                  | notice                                                                           | Sets the message levels that are sent to the client.
 cpu_index_tuple_cost                                 | 0.005                                                                            | Sets the planner's estimate of the cost of processing each index entry during an index scan.
 cpu_operator_cost                                    | 0.0025                                                                           | Sets the planner's estimate of the cost of processing each operator or function call.
 cpu_tuple_cost                                       | 0.01                                                                             | Sets the planner's estimate of the cost of processing each tuple (row).
 create_restartpoint_on_ckpt_record_replay            | off                                                                              | create a restartpoint only on mirror immediately after replaying a checkpoint record.
 cursor_tuple_fraction                                | 1                                                                                | Sets the planner's estimate of the fraction of a cursor's rows that will be retrieved.
 data_checksums                                       | on                                                                               | Shows whether data checksums are turned on for this cluster.
 data_sync_retry                                      | off                                                                              | Whether to continue running after a failure to sync data files.
 DateStyle                                            | ISO, MDY                                                                         | Sets the display format for date and time values.
 db_user_namespace                                    | off                                                                              | Enables per-database user names.
 deadlock_timeout                                     | 1s                                                                               | Sets the time to wait on a lock before checking for deadlock.
 debug_assertions                                     | off                                                                              | Turns on various assertion checks.
 debug_pretty_print                                   | on                                                                               | Indents parse and plan tree displays.
 debug_print_parse                                    | off                                                                              | Logs each query's parse tree.
 debug_print_plan                                     | off                                                                              | Logs each query's execution plan.
 debug_print_prelim_plan                              | off                                                                              | Prints the preliminary execution plan to server log.
 debug_print_rewritten                                | off                                                                              | Logs each query's rewritten parse tree.
 debug_print_slice_table                              | off                                                                              | Prints the slice table to server log.
 default_statistics_target                            | 100                                                                              | Sets the default statistics target.
 default_tablespace                                   |                                                                                  | Sets the default tablespace to create tables and indexes in.
 default_text_search_config                           | pg_catalog.english                                                               | Sets default text search configuration.
 default_transaction_deferrable                       | off                                                                              | Sets the default deferrable status of new transactions.
 default_transaction_isolation                        | read committed                                                                   | Sets the transaction isolation level of each new transaction.
 default_transaction_read_only                        | off                                                                              | Sets the default read-only status of new transactions.
 dynamic_library_path                                 | $libdir                                                                          | Sets the path for dynamically loadable modules.
 dynamic_shared_memory_type                           | posix                                                                            | Selects the dynamic shared memory implementation used.
 effective_cache_size                                 | 16GB                                                                             | Sets the planner's assumption about the total size of the data caches.
 effective_io_concurrency                             | 1                                                                                | Number of simultaneous requests that can be handled efficiently by the disk subsystem.
 enable_bitmapscan                                    | on                                                                               | Enables the planner's use of bitmap-scan plans.
 enable_groupagg                                      | on                                                                               | Enables the planner's use of grouping aggregation plans.
 enable_hashagg                                       | on                                                                               | Enables the planner's use of hashed aggregation plans.
 enable_hashjoin                                      | on                                                                               | Enables the planner's use of hash join plans.
 enable_indexonlyscan                                 | on                                                                               | Enables the planner's use of index-only-scan plans.
 enable_indexscan                                     | on                                                                               | Enables the planner's use of index-scan plans.
 enable_material                                      | on                                                                               | Enables the planner's use of materialization.
 enable_mergejoin                                     | off                                                                              | Enables the planner's use of merge join plans.
 enable_nestloop                                      | off                                                                              | Enables the planner's use of nested-loop join plans.
 enable_seqscan                                       | on                                                                               | Enables the planner's use of sequential-scan plans.
 enable_sort                                          | on                                                                               | Enables the planner's use of explicit sort steps.
 enable_tidscan                                       | on                                                                               | Enables the planner's use of TID scan plans.
 escape_string_warning                                | on                                                                               | Warn about backslash escapes in ordinary string literals.
 event_source                                         | PostgreSQL                                                                       | Sets the application name used to identify PostgreSQL messages in the event log.
 exit_on_error                                        | off                                                                              | Terminate session on any error.
 explain_memory_verbosity                             | suppress                                                                         | Experimental feature: show memory account usage in EXPLAIN ANALYZE.
 extra_float_digits                                   | 0                                                                                | Sets the number of digits displayed for floating-point values.
 from_collapse_limit                                  | 20                                                                               | Sets the FROM-list size beyond which subqueries are not collapsed.
 geqo_seed                                            | 1                                                                                | Unused. Syntax check only for PostgreSQL compatibility.
 gin_fuzzy_search_limit                               | 0                                                                                | Sets the maximum allowed result for exact search by GIN.
 gp_adjust_selectivity_for_outerjoins                 | on                                                                               | Adjust selectivity of null tests over outer joins.
 gp_appendonly_compaction_threshold                   | 10                                                                               | Threshold of the ratio of dirty data in a segment file over which the file will be compacted during lazy vacuum.
 gp_autostats_mode                                    | on_no_stats                                                                      | Sets the autostats mode.
 gp_autostats_mode_in_functions                       | none                                                                             | Sets the autostats mode for statements in procedural language functions.
 gp_autostats_on_change_threshold                     | 2147483647                                                                       | Threshold for number of tuples added to table by CTAS or Insert-to to trigger autostats in on_change mode. See gp_autostats_mode.
 gp_cached_segworkers_threshold                       | 5                                                                                | Sets the maximum number of segment workers to cache between statements.
 gp_command_count                                     | 1                                                                                | Shows the number of commands received from the client in this session.
 gp_connection_send_timeout                           | 3600                                                                             | Timeout for sending data to unresponsive clients (seconds)
 gp_contentid                                         | -1                                                                               | The contentid used by this server.
 gp_create_table_random_default_distribution          | off                                                                              | Set the default distribution of a table to RANDOM.
 gp_dbid                                              | 1                                                                                | The dbid used by this server.
 gp_debug_linger                                      | 0                                                                                | Number of seconds for QD/QE process to linger upon fatal internal error.
 gp_default_storage_options                           | appendonly=false,blocksize=32768,compresstype=none,checksum=true,orientation=row | default options for appendonly storage.
 gp_dynamic_partition_pruning                         | on                                                                               | This guc enables plans that can dynamically eliminate scanning of partitions.
 gp_enable_agg_distinct                               | on                                                                               | Enable 2-phase aggregation to compute a single distinct-qualified aggregate.
 gp_enable_agg_distinct_pruning                       | on                                                                               | Enable 3-phase aggregation and join to compute distinct-qualified aggregates.
 gp_enable_direct_dispatch                            | on                                                                               | Enable dispatch for single-row-insert targetted mirror-pairs.
 gp_enable_exchange_default_partition                 | off                                                                              | Allow DDL that will exchange default partitions.
 gp_enable_fast_sri                                   | on                                                                               | Enable single-slice single-row inserts.
 gp_enable_global_deadlock_detector                   | off                                                                              | Enables the Global Deadlock Detector.
 gp_enable_gpperfmon                                  | off                                                                              | Enable gpperfmon monitoring.
 gp_enable_groupext_distinct_gather                   | on                                                                               | Enable gathering data to a single node to compute distinct-qualified aggregates on grouping extention queries.
 gp_enable_groupext_distinct_pruning                  | on                                                                               | Enable 3-phase aggregation and join to compute distinct-qualified aggregates on grouping extention queries.
 gp_enable_minmax_optimization                        | on                                                                               | Enables the planner's use of index scans with limit to implement MIN/MAX.
 gp_enable_multiphase_agg                             | on                                                                               | Enables the planner's use of two- or three-stage parallel aggregation plans.
 gp_enable_predicate_propagation                      | on                                                                               | When two expressions are equivalent (such as with equijoined keys) then the planner applies predicates on one expression to the oth
er expression.
 gp_enable_preunique                                  | on                                                                               | Enable 2-phase duplicate removal.
 gp_enable_query_metrics                              | off                                                                              | Enable all query metrics collection.
 gp_enable_relsize_collection                         | off                                                                              | This guc enables relsize collection when stats are not present. If disabled and stats are not present a default value is used.
 gp_enable_sort_distinct                              | on                                                                               | Enable duplicate removal to be performed while sorting.
 gp_enable_sort_limit                                 | on                                                                               | Enable LIMIT operation to be performed while sorting.
 gp_external_enable_exec                              | on                                                                               | Enable selecting from an external table with an EXECUTE clause.
 gp_external_enable_filter_pushdown                   | on                                                                               | Enable passing of query constraints to external table providers
 gp_external_max_segs                                 | 64                                                                               | Maximum number of segments that connect to a single gpfdist URL.
 gp_fts_mark_mirror_down_grace_period                 | 30s                                                                              | Time (in seconds) allowed to mirror after disconnection, to reconnect before being marked as down in configuration by FTS.
 gp_fts_probe_interval                                | 1min                                                                             | A complete probe of all segments starts each time a timer with this period expires.
 gp_fts_probe_retries                                 | 5                                                                                | Number of retries for FTS to complete probing a segment.
 gp_fts_probe_timeout                                 | 20s                                                                              | Maximum time (in seconds) allowed for FTS to complete probing a segment.
 gp_global_deadlock_detector_period                   | 2min                                                                             | Sets the executing period of global deadlock detector backend.
 gp_gpperfmon_send_interval                           | 1                                                                                | Interval in seconds between sending messages to gpperfmon.
 gp_hashjoin_tuples_per_bucket                        | 5                                                                                | Target density of hashtable used by Hashjoin during execution
 gp_initial_bad_row_limit                             | 1000                                                                             | Stops processing when number of the first bad rows exceeding this value
 gp_instrument_shmem_size                             | 5MB                                                                              | Sets the size of shmem allocated for instrumentation.
 gp_interconnect_cache_future_packets                 | on                                                                               | Control whether future packets are cached.
 gp_interconnect_debug_retry_interval                 | 10                                                                               | Sets the interval by retry times to record a debug message for retry.
 gp_interconnect_default_rtt                          | 20ms                                                                             | Sets the default rtt (in ms) for UDP interconnect
 gp_interconnect_fc_method                            | loss                                                                             | Sets the flow control method used for UDP interconnect.
 gp_interconnect_min_retries_before_timeout           | 100                                                                              | Sets the min retries before reporting a transmit timeout in the interconnect.
 gp_interconnect_min_rto                              | 20ms                                                                             | Sets the min rto (in ms) for UDP interconnect
 gp_interconnect_queue_depth                          | 4                                                                                | Sets the maximum size of the receive queue for each connection in the UDP interconnect
 gp_interconnect_setup_timeout                        | 2h                                                                               | Timeout (in seconds) on interconnect setup that occurs at query start
 gp_interconnect_snd_queue_depth                      | 2                                                                                | Sets the maximum size of the send queue for each connection in the UDP interconnect
 gp_interconnect_tcp_listener_backlog                 | 128                                                                              | Size of the listening queue for each TCP interconnect socket
 gp_interconnect_timer_checking_period                | 20ms                                                                             | Sets the timer checking period (in ms) for UDP interconnect
 gp_interconnect_timer_period                         | 5ms                                                                              | Sets the timer period (in ms) for UDP interconnect
 gp_interconnect_transmit_timeout                     | 1h                                                                               | Timeout (in seconds) on interconnect to transmit a packet
 gp_interconnect_type                                 | udpifc                                                                           | Sets the protocol used for inter-node communication.
 gp_log_format                                        | csv                                                                              | Sets the format for log files.
 gp_max_local_distributed_cache                       | 1024                                                                             | Sets the number of local-distributed transactions to cache for optimizing visibility processing by backends.
 gp_max_packet_size                                   | 8192                                                                             | Sets the max packet size for the Interconnect.
 gp_max_partition_level                               | 0                                                                                | Sets the maximum number of levels allowed when creating a partitioned table.
 gp_max_plan_size                                     | 0                                                                                | Sets the maximum size of a plan to be dispatched.
 gp_max_slices                                        | 0                                                                                | Maximum slices for a single query
 gp_motion_cost_per_row                               | 0                                                                                | Sets the planner's estimate of the cost of moving a row between worker processes.
 gp_reject_percent_threshold                          | 300                                                                              | Reject limit in percent starts calculating after this number of rows processed
 gp_reraise_signal                                    | on                                                                               | Do we attempt to dump core when a serious problem occurs.
 gp_resgroup_memory_policy                            | eager_free                                                                       | Sets the policy for memory allocation of queries.
 gp_resource_group_bypass                             | off                                                                              | If the value is true, the query in this session will not be limited by resource group.
 gp_resource_group_cpu_limit                          | 0.9                                                                              | Maximum percentage of CPU resources assigned to a cluster.
 gp_resource_group_cpu_priority                       | 10                                                                               | Sets the cpu priority for postgres processes when resource group is enabled.
 gp_resource_group_memory_limit                       | 0.7                                                                              | Maximum percentage of memory resources assigned to a cluster.
 gp_resource_manager                                  | queue                                                                            | Sets the type of resource manager.
 gp_resqueue_memory_policy                            | eager_free                                                                       | Sets the policy for memory allocation of queries.
 gp_resqueue_priority                                 | on                                                                               | Enables priority scheduling.
 gp_resqueue_priority_cpucores_per_segment            | 4                                                                                | Number of processing units associated with a segment.
 gp_resqueue_priority_sweeper_interval                | 1000                                                                             | Frequency (in ms) at which sweeper process re-evaluates CPU shares.
 gp_role                                              | dispatch                                                                         | Sets the role for the session.
 gp_safefswritesize                                   | 0                                                                                | Minimum FS safe write size.
 gp_segment_connect_timeout                           | 10min                                                                            | Maximum time (in seconds) allowed for a new worker process to start or a mirror to respond.
 gp_segments_for_planner                              | 0                                                                                | If >0, number of segment dbs for the planner to assume in its cost and size estimates.
 gp_server_version                                    | 6.2.1 build commit:d90ac1a1b983b913b3950430d4d9e47ee8827fd4                      | Shows the Greenplum server version.
 gp_server_version_num                                | 60000                                                                            | Shows the Greenplum server version as an integer.
 gp_session_id                                        | 314                                                                              | Global ID used to uniquely identify a particular session in an Greenplum Database array
 gp_set_proc_affinity                                 | off                                                                              | On postmaster startup, attempt to bind postmaster to a processor
 gp_statistics_pullup_from_child_partition            | on                                                                               | This guc enables the planner to utilize statistics from partitions in planning queries on the parent.
 gp_statistics_use_fkeys                              | on                                                                               | This guc enables the planner to utilize statistics derived from foreign key relationships.
 gp_subtrans_warn_limit                               | 16777216                                                                         | Sets the warning limit on number of subtransactions in a transaction.
 gp_udp_bufsize_k                                     | 0                                                                                | Sets recv buf size of UDP interconnect, for testing.
 gp_vmem_idle_resource_timeout                        | 18s                                                                              | Sets the time a session can be idle (in milliseconds) before we release gangs on the segment DBs to free resources.
 gp_vmem_protect_limit                                | 8192                                                                             | Virtual memory limit (in MB) of Greenplum memory protection.
 gp_vmem_protect_segworker_cache_limit                | 500                                                                              | Max virtual memory limit (in MB) for a segworker to be cachable.
 gp_workfile_compression                              | off                                                                              | Enables compression of temporary files.
 gp_workfile_limit_files_per_query                    | 100000                                                                           | Maximum number of workfiles allowed per query per segment.
 gp_workfile_limit_per_query                          | 0                                                                                | Maximum disk space (in KB) used for workfiles per query per segment.
 gp_workfile_limit_per_segment                        | 0                                                                                | Maximum disk space (in KB) used for workfiles per segment.
 gpperfmon_log_alert_level                            | none                                                                             | Specify the log alert level used by gpperfmon.
 gpperfmon_port                                       | 8888                                                                             | Sets the port number of gpperfmon.
 hot_standby                                          | off                                                                              | Allows connections and queries during recovery.
 hot_standby_feedback                                 | off                                                                              | Allows feedback from a hot standby to the primary that will avoid query conflicts.
 huge_pages                                           | try                                                                              | Use of huge pages on Linux.
 ignore_checksum_failure                              | off                                                                              | Continues processing after a checksum failure.
 integer_datetimes                                    | on                                                                               | Datetimes are integer based.
 IntervalStyle                                        | postgres                                                                         | Sets the display format for interval values.
 join_collapse_limit                                  | 20                                                                               | Sets the FROM-list size beyond which JOIN constructs are not flattened.
 krb_caseins_users                                    | off                                                                              | Sets whether Kerberos and GSSAPI user names should be treated as case-insensitive.
 krb_server_keyfile                                   | FILE:/usr/local/greenplum-db-oss/etc/postgresql/krb5.keytab                      | Sets the location of the Kerberos server key file.
 lc_collate                                           | en_US.utf8                                                                       | Shows the collation order locale.
 lc_ctype                                             | en_US.utf8                                                                       | Shows the character classification and case conversion locale.
 lc_messages                                          | en_US.utf8                                                                       | Sets the language in which messages are displayed.
 lc_monetary                                          | en_US.utf8                                                                       | Sets the locale for formatting monetary amounts.
 lc_numeric                                           | en_US.utf8                                                                       | Sets the locale for formatting numbers.
 lc_time                                              | en_US.utf8                                                                       | Sets the locale for formatting date and time values.
 listen_addresses                                     | *                                                                                | Sets the host name or IP address(es) to listen to.
 lo_compat_privileges                                 | off                                                                              | Enables backward compatibility mode for privilege checks on large objects.
 local_preload_libraries                              |                                                                                  | Lists unprivileged shared libraries to preload into each backend.
 lock_timeout                                         | 0                                                                                | Sets the maximum allowed duration of any wait for a lock.
 log_autostats                                        | off                                                                              | Logs details of auto-stats issued ANALYZEs.
 log_autovacuum_min_duration                          | -1                                                                               | Sets the minimum execution time above which autovacuum actions will be logged.
 log_checkpoints                                      | off                                                                              | Logs each checkpoint.
 log_connections                                      | off                                                                              | Logs each successful connection.
 log_disconnections                                   | off                                                                              | Logs end of a session, including duration.
 log_dispatch_stats                                   | off                                                                              | Writes dispatcher performance statistics to the server log.
 log_duration                                         | off                                                                              | Logs the duration of each completed SQL statement.
 log_error_verbosity                                  | default                                                                          | Sets the verbosity of logged messages.
 log_executor_stats                                   | off                                                                              | Writes executor performance statistics to the server log.
 log_file_mode                                        | 0600                                                                             | Sets the file permissions for log files.
 log_hostname                                         | off                                                                              | Logs the host name in the connection logs.
 log_lock_waits                                       | off                                                                              | Logs long lock waits.
 log_min_duration_statement                           | -1                                                                               | Sets the minimum execution time above which statements will be logged.
 log_min_error_statement                              | error                                                                            | Causes all statements generating error at or above this level to be logged.
 log_min_messages                                     | warning                                                                          | Sets the message levels that are logged.
 log_parser_stats                                     | off                                                                              | Writes parser performance statistics to the server log.
 log_planner_stats                                    | off                                                                              | Writes planner performance statistics to the server log.
 log_rotation_age                                     | 1d                                                                               | Automatic log file rotation will occur after N minutes.
 log_rotation_size                                    | 1GB                                                                              | Automatic log file rotation will occur after N kilobytes.
 log_statement                                        | all                                                                              | Sets the type of statements logged.
 log_statement_stats                                  | off                                                                              | Writes cumulative performance statistics to the server log.
 log_temp_files                                       | -1                                                                               | Log the use of temporary files larger than this number of kilobytes.
 log_timezone                                         | PRC                                                                              | Sets the time zone to use in log messages.
 log_truncate_on_rotation                             | off                                                                              | Truncate existing log files of same name during log rotation.
 logging_collector                                    | on                                                                               | Start a subprocess to capture stderr output and/or csvlogs into log files.
 maintenance_work_mem                                 | 64MB                                                                             | Sets the maximum memory to be used for maintenance operations.
 max_appendonly_tables                                | 10000                                                                            | Maximum number of different (unrelated) append only tables that can participate in writing data concurrently.
 max_connections                                      | 250                                                                              | Sets the maximum number of concurrent connections.
 max_files_per_process                                | 1000                                                                             | Sets the maximum number of simultaneously open files for each server process.
 max_function_args                                    | 100                                                                              | Shows the maximum number of function arguments.
 max_identifier_length                                | 63                                                                               | Shows the maximum identifier length.
 max_index_keys                                       | 32                                                                               | Shows the maximum number of index keys.
 max_locks_per_transaction                            | 128                                                                              | Sets the maximum number of locks per transaction.
 max_pred_locks_per_transaction                       | 64                                                                               | Sets the maximum number of predicate locks per transaction.
 max_prepared_transactions                            | 250                                                                              | Sets the maximum number of simultaneously prepared transactions.
 max_replication_slots                                | 10                                                                               | Sets the maximum number of simultaneously defined replication slots.
 max_resource_portals_per_transaction                 | 64                                                                               | Maximum number of resource queues.
 max_resource_queues                                  | 9                                                                                | Maximum number of resource queues.
 max_stack_depth                                      | 2MB                                                                              | Sets the maximum stack depth, in kilobytes.
 max_standby_archive_delay                            | 30s                                                                              | Sets the maximum delay before canceling queries when a hot standby server is processing archived WAL data.
 max_standby_streaming_delay                          | 30s                                                                              | Sets the maximum delay before canceling queries when a hot standby server is processing streamed WAL data.
 max_statement_mem                                    | 2000MB                                                                           | Sets the maximum value for statement_mem setting.
 max_wal_senders                                      | 10                                                                               | Sets the maximum number of simultaneously running WAL sender processes.
 max_worker_processes                                 | 14                                                                               | Maximum number of concurrent worker processes.
 memory_spill_ratio                                   | 20                                                                               | Sets the memory_spill_ratio for resource group.
 optimizer                                            | on                                                                               | Enable Pivotal Query Optimizer.
 optimizer_analyze_root_partition                     | on                                                                               | Enable statistics collection on root partitions during ANALYZE
 optimizer_control                                    | on                                                                               | Allow/disallow turning the optimizer on or off.
 optimizer_enable_associativity                       | off                                                                              | Enables Join Associativity in optimizer
 optimizer_join_arity_for_associativity_commutativity | 18                                                                               | Maximum number of children n-ary-join have without disabling commutativity and associativity transform
 optimizer_join_order                                 | exhaustive                                                                       | Set optimizer join heuristic model.
 optimizer_join_order_threshold                       | 10                                                                               | Maximum number of join children to use dynamic programming based join ordering algorithm.
 optimizer_mdcache_size                               | 16MB                                                                             | Sets the size of MDCache.
 optimizer_metadata_caching                           | on                                                                               | This guc enables the optimizer to cache and reuse metadata.
 optimizer_minidump                                   | onerror                                                                          | Generate optimizer minidump.
 optimizer_parallel_union                             | off                                                                              | Enable parallel execution for UNION/UNION ALL queries.
 password_encryption                                  | on                                                                               | Encrypt passwords.
 password_hash_algorithm                              | MD5                                                                              | The cryptograph hash algorithm to apply to passwords before storing them.
 pljava_classpath                                     |                                                                                  | classpath used by the the JVM
 pljava_classpath_insecure                            | off                                                                              | Allow pljava_classpath to be set by user per session
 pljava_release_lingering_savepoints                  | off                                                                              | If true, lingering savepoints will be released on function exit; if false, they will be rolled back
 pljava_statement_cache_size                          | 0                                                                                | Size of the prepared statement MRU cache
 pljava_vmoptions                                     |                                                                                  | Options sent to the JVM when it is created
 port                                                 | 5432                                                                             | Sets the TCP port the server listens on.
 quote_all_identifiers                                | off                                                                              | When generating SQL fragments, quote all identifiers.
 random_page_cost                                     | 100                                                                              | Sets the planner's estimate of the cost of a nonsequentially fetched disk page.
 readable_external_table_timeout                      | 0                                                                                | Cancel the query if no data read within N seconds.
 repl_catchup_within_range                            | 1                                                                                | Sets the maximum number of xlog segments allowed to lag when the backends can start blocking despite the WAL sender being in catchu
p phase. (Master Mirroring)
 resource_cleanup_gangs_on_wait                       | on                                                                               | Enable idle gang cleanup before resource lockwait.
 resource_scheduler                                   | on                                                                               | Enable resource scheduling.
 resource_select_only                                 | off                                                                              | Enable resource locking of SELECT only.
 restart_after_crash                                  | on                                                                               | Reinitialize server after backend crash.
 runaway_detector_activation_percent                  | 90                                                                               | The runaway detector activates if the used vmem exceeds this percentage of the vmem quota. Set to 100 to disable runaway detection.
 search_path                                          | "$user",public                                                                   | Sets the schema search order for names that are not schema-qualified.
 segment_size                                         | 1GB                                                                              | Shows the number of pages per disk file.
 seq_page_cost                                        | 1                                                                                | Sets the planner's estimate of the cost of a sequentially fetched disk page.
 server_encoding                                      | UTF8                                                                             | Sets the server (database) character set encoding.
 server_version                                       | 9.4.24                                                                           | Shows the server version.
 server_version_num                                   | 90424                                                                            | Shows the server version as an integer.
 session_preload_libraries                            |                                                                                  | Lists shared libraries to preload into each backend.
 session_replication_role                             | origin                                                                           | Sets the session's behavior for triggers and rewrite rules.
 shared_buffers                                       | 125MB                                                                            | Sets the number of shared memory buffers used by the server.
 shared_preload_libraries                             |                                                                                  | Lists shared libraries to preload into server.
 ssl                                                  | off                                                                              | Enables SSL connections.
 ssl_ca_file                                          |                                                                                  | Location of the SSL certificate authority file.
 ssl_cert_file                                        | server.crt                                                                       | Location of the SSL server certificate file.
 ssl_ciphers                                          | HIGH:MEDIUM:+3DES:!aNULL                                                         | Sets the list of allowed SSL ciphers.
 ssl_crl_file                                         |                                                                                  | Location of the SSL certificate revocation list file.
 ssl_ecdh_curve                                       | prime256v1                                                                       | Sets the curve to use for ECDH.
 ssl_key_file                                         | server.key                                                                       | Location of the SSL server private key file.
 ssl_prefer_server_ciphers                            | on                                                                               | Give priority to server ciphersuite order.
 standard_conforming_strings                          | on                                                                               | Causes '...' strings to treat backslashes literally.
 statement_mem                                        | 125MB                                                                            | Sets the memory to be reserved for a statement.
 statement_timeout                                    | 0                                                                                | Sets the maximum allowed duration of any statement.
 stats_queue_level                                    | off                                                                              | Collects resource queue-level statistics on database activity.
 superuser_reserved_connections                       | 3                                                                                | Sets the number of connection slots reserved for superusers (including reserved FTS connections).
 synchronize_seqscans                                 | on                                                                               | Enable synchronized sequential scans.
 synchronous_commit                                   | on                                                                               | Sets the current transaction's synchronization level.
 synchronous_standby_names                            |                                                                                  | List of names of potential synchronous standbys.
 tcp_keepalives_count                                 | 0                                                                                | Maximum number of TCP keepalive retransmits.
 tcp_keepalives_idle                                  | 0                                                                                | Time between issuing TCP keepalives.
 tcp_keepalives_interval                              | 0                                                                                | Time between TCP keepalive retransmits.
 temp_buffers                                         | 32MB                                                                             | Sets the maximum number of temporary buffers used by each session.
 temp_file_limit                                      | -1                                                                               | Limits the total size of all temporary files used by each session.
 temp_tablespaces                                     |                                                                                  | Sets the tablespace(s) to use for temporary tables and sort files.
 TimeZone                                             | PRC                                                                              | Sets the time zone for displaying and interpreting time stamps.
 timezone_abbreviations                               | Default                                                                          | Selects a file of time zone abbreviations.
 trace_recovery_messages                              | log                                                                              | Enables logging of recovery-related debugging information.
 track_activities                                     | on                                                                               | Collects information about executing commands.
 track_activity_query_size                            | 1024                                                                             | Sets the size reserved for pg_stat_activity.query, in bytes.
 track_counts                                         | on                                                                               | Collects statistics on database activity.
 track_functions                                      | none                                                                             | Collects function-level statistics on database activity.
 track_io_timing                                      | off                                                                              | Collects timing statistics for database I/O activity.
 transaction_deferrable                               | off                                                                              | Whether to defer a read-only serializable transaction until it can be executed with no possible serialization failures.
 transaction_isolation                                | read committed                                                                   | Sets the current transaction's isolation level.
 transaction_read_only                                | off                                                                              | Sets the current transaction's read-only status.
 transform_null_equals                                | off                                                                              | Treats "expr=NULL" as "expr IS NULL".
 unix_socket_directories                              | /tmp                                                                             | Sets the directories where Unix-domain sockets will be created.
 unix_socket_group                                    |                                                                                  | Sets the owning group of the Unix-domain socket.
 unix_socket_permissions                              | 0777                                                                             | Sets the access permissions of the Unix-domain socket.
 update_process_title                                 | on                                                                               | Updates the process title to show the active SQL command.
 vacuum_cost_delay                                    | 0                                                                                | Vacuum cost delay in milliseconds.
 vacuum_cost_limit                                    | 200                                                                              | Vacuum cost amount available before napping.
 vacuum_cost_page_dirty                               | 20                                                                               | Vacuum cost for a page dirtied by vacuum.
 vacuum_cost_page_hit                                 | 1                                                                                | Vacuum cost for a page found in the buffer cache.
 vacuum_cost_page_miss                                | 10                                                                               | Vacuum cost for a page not found in the buffer cache.
 vacuum_defer_cleanup_age                             | 0                                                                                | Number of transactions by which VACUUM and HOT cleanup should be deferred, if any.
 vacuum_freeze_min_age                                | 50000000                                                                         | Minimum age at which VACUUM should freeze a table row.
 vacuum_freeze_table_age                              | 150000000                                                                        | Age at which VACUUM should scan whole table to freeze tuples.
 vacuum_multixact_freeze_min_age                      | 5000000                                                                          | Minimum age at which VACUUM should freeze a MultiXactId in a table row.
 vacuum_multixact_freeze_table_age                    | 150000000                                                                        | Multixact age at which VACUUM should scan whole table to freeze tuples.
 wal_block_size                                       | 32768                                                                            | Shows the block size in the write ahead log.
 wal_buffers                                          | 4000kB                                                                           | Sets the number of disk-page buffers in shared memory for WAL.
 wal_keep_segments                                    | 5                                                                                | Sets the number of WAL files held for standby servers.
 wal_level                                            | archive                                                                          | Set the level of information written to the WAL.
 wal_log_hints                                        | off                                                                              | Writes full pages to WAL when first modified after a checkpoint, even for a non-critical modifications.
 wal_receiver_status_interval                         | 10s                                                                              | Sets the maximum interval between WAL receiver status reports to the primary.
 wal_receiver_timeout                                 | 1min                                                                             | Sets the maximum wait time to receive data from the primary.
 wal_segment_size                                     | 64MB                                                                             | Shows the number of pages per write ahead log segment.
 wal_sender_timeout                                   | 1min                                                                             | Sets the maximum time to wait for WAL replication.
 wal_writer_delay                                     | 200ms                                                                            | WAL writer sleep time between WAL flushes.
 work_mem                                             | 32MB                                                                             | Sets the maximum memory to be used for query workspaces.
 writable_external_table_bufsize                      | 64kB                                                                             | Buffer size in kilobytes for writable external table before writing data to gpfdist.
 xmlbinary                                            | base64                                                                           | Sets how binary values are to be encoded in XML.
 xmloption                                            | content                                                                          | Sets whether XML data in implicit parsing and serialization operations is to be considered as documents or content fragments.
(324 rows)
```