---
displayed_sidebar: "English"
---

# Parameter configuration

This topic describes FE, BE, and system parameters. It also provides suggestions on how to configure and tune these parameters.

## FE configuration items

FE parameters are classified into dynamic parameters and static parameters.

- Dynamic parameters can be configured and adjusted by running SQL commands, which is very convenient. But the configurations become invalid if you restart your FE. Therefore, we recommend that you also modify the configuration items in the `fe.conf` file to prevent the loss of modifications.

- Static parameters can only be configured and adjusted in the FE configuration file **fe.conf**. **After you modify this file, you must restart your FE for the changes to take effect.**

Whether a parameter is a dynamic parameter is indicated by the `IsMutable` column in the output of [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md). `TRUE` indicates a dynamic parameter.

Note that both dynamic and static FE parameters can be configured in the **fe.conf** file.

### View FE configuration items

After your FE is started, you can run the ADMIN SHOW FRONTEND CONFIG command on your MySQL client to check the parameter configurations. If you want to query the configuration of a specific parameter, run the following command:

```SQL
 ADMIN SHOW FRONTEND CONFIG [LIKE "pattern"];
 ```

 For detailed description of the returned fields, see [ADMIN SHOW CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SHOW_CONFIG.md).

> **NOTE**
>
> You must have administrator's privilege to run cluster administration-related commands.

### Configure FE dynamic parameters

You can configure or modify the settings of FE dynamic parameters using [ADMIN SET FRONTEND CONFIG](../sql-reference/sql-statements/Administration/ADMIN_SET_CONFIG.md).

```SQL
ADMIN SET FRONTEND CONFIG ("key" = "value");
```

> **NOTE**
>
> The configurations will be restored to the default values in the `fe.conf` file after the FE restarts. Therefore, we recommend that you also modify the configuration items in `fe.conf` to prevent the loss of modifications.

#### Logging

##### qe_slow_log_ms

- Unit: ms
- Default: 5000
- Description: The threshold used to determine whether a query is a slow query. If the response time of a query exceeds this threshold, it is recorded as a slow query in `fe.audit.log`.

#### Metadata and cluster management

##### catalog_try_lock_timeout_ms

- **Unit**: ms
- **Default**: 5000
- **Description**: The timeout duration to obtain the global lock.

##### edit_log_roll_num

- **Unit**: -
- **Default**: 50000
- **Description**: The maximum number of metadata log entries that can be written before a log file is created for these log entries. This parameter is used to control the size of log files. The new log file is written to the BDBJE database.

##### ignore_unknown_log_id

- **Unit**: -
- **Default**: FALSE
- **Description**: Whether to ignore an unknown log ID. When an FE is rolled back, the BEs of the earlier version may be unable to recognize some log IDs. If the value is `TRUE`, the FE ignores unknown log IDs. If the value is `FALSE`, the FE exits.

##### ignore_materialized_view_error

- **Unit**: -
- **Default**: FALSE
- **Description**: Whether FE ignores the metadata exception caused by materialized view errors. If FE fails to start due to the metadata exception caused by materialized view errors, you can set this parameter to `true` to allow FE to ignore the exception. This parameter is supported from v2.5.10 onwards.

##### ignore_meta_check

- **Unit**: -
- **Default**: FALSE
- **Description**: Whether non-leader FEs ignore the metadata gap from the leader FE. If the value is TRUE, non-leader FEs ignore the metadata gap from the leader FE and continue providing data reading services. This parameter ensures continuous data reading services even when you stop the leader FE for a long period of time. If the value is FALSE, non-leader FEs do not ignore the metadata gap from the leader FE and stop providing data reading services.

##### meta_delay_toleration_second

- **Unit**: s
- **Default**: 300
- **Description**: The maximum duration by which the metadata on the follower and observer FEs can lag behind that on the leader FE. Unit: seconds. If this duration is exceeded, the non-leader FEs stops providing services.

##### drop_backend_after_decommission

- **Unit**: -
- **Default**: TRUE
- **Description**: Whether to delete a BE after the BE is decommissioned. `TRUE` indicates that the BE is deleted immediately after it is decommissioned. `FALSE` indicates that the BE is not deleted after it is decommissioned.

##### enable_collect_query_detail_info

- **Unit**: -
- **Default**: FALSE
- **Description**: Whether to collect the profile of a query. If this parameter is set to `TRUE`, the system collects the profile of the query. If this parameter is set to `FALSE`, the system does not collect the profile of the query.

##### enable_background_refresh_connector_metadata

- **Unit**: -
- **Default**: `true` in v3.0<br />`false` in v2.5
- **Description**: Whether to enable the periodic Hive metadata cache refresh. After it is enabled, StarRocks polls the metastore (Hive Metastore or AWS Glue) of your Hive cluster, and refreshes the cached metadata of the frequently accessed Hive catalogs to perceive data changes. `true` indicates to enable the Hive metadata cache refresh, and `false` indicates to disable it. This parameter is supported from v2.5.5 onwards.

##### background_refresh_metadata_interval_millis

- **Unit**: ms
- **Default**: 600000
- **Description**: The interval between two consecutive Hive metadata cache refreshes. This parameter is supported from v2.5.5 onwards.

##### background_refresh_metadata_time_secs_since_last_access_secs

- **Unit**: s
- **Default**: 86400
- **Description**: The expiration time of a Hive metadata cache refresh task. For the Hive catalog that has been accessed, if it has not been accessed for more than the specified time, StarRocks stops refreshing its cached metadata. For the Hive catalog that has not been accessed, StarRocks will not refresh its cached metadata. This parameter is supported from v2.5.5 onwards.

##### enable_statistics_collect_profile

- **Unit**: N/A
- **Default**: false
- **Description**: Whether to generate profiles for statistics queries. You can set this item to `true` to allow StarRocks to generate query profiles for queries on system statistics. This parameter is supported from v3.1.5 onwards.

#### User, role, privilege

##### privilege_max_total_roles_per_user

- **Unit**: -
- **Default**: 64
- **Description**: The maximum number of roles a user can have.
- **Introduced in**: v3.0.0

##### privilege_max_role_depth

- **Unit**: -
- **Default**: 16
- **Description**: The maximum role depth (level of inheritance) of a role.
- **Introduced in**: v3.0.0

#### Query engine

##### max_allowed_in_element_num_of_delete

- Unit: -
- Default: 10000
- Description: The maximum number of elements allowed for the IN predicate in a DELETE statement.

##### enable_materialized_view

- Unit: -
- Default: TRUE
- Description: Whether to enable the creation of materialized views.

##### enable_decimal_v3

- Unit: -
- Default: TRUE
- Description: Whether to support the DECIMAL V3 data type.

##### expr_children_limit

- Unit: -
- Default: 10000
- Description: The maximum number of child expressions allowed in an expression.

##### enable_sql_blacklist

- Unit: -
- Default: FALSE
- Description: Whether to enable blacklist check for SQL queries. When this feature is enabled, queries in the blacklist cannot be executed.

##### dynamic_partition_check_interval_seconds

- Unit: s
- Default: 600
- Description: The interval at which new data is checked. If new data is detected, StarRocks automatically creates partitions for the data.

##### dynamic_partition_enable

- Unit: -
- Default: TRUE
- Description: Whether to enable the dynamic partitioning feature. When this feature is enabled, StarRocks dynamically creates partitions for new data and automatically deletes expired partitions to ensure the freshness of data.

##### http_slow_request_threshold_ms

- Unit: ms
- Default: 5000
- Description: If the response time for an HTTP request exceeds the value specified by this parameter, a log is generated to track this request.
- Introduced in: 2.5.15, 3.1.5

##### max_partitions_in_one_batch

- Unit: -
- Default: 4096
- Description: The maximum number of partitions that can be created when you bulk create partitions.

##### max_query_retry_time

- Unit: -
- Default: 2
- Description: The maximum number of query retries on an FE.

##### max_create_table_timeout_second

- Unit: s
- Default: 600
- Description: The maximum timeout duration for creating a table.

##### create_table_max_serial_replicas

- Unit: -
- Default: 128
- Description: The maximum number of replicas to create serially. If actual replica count exceeds this, replicas will be created concurrently. Try to reduce this config if table creation is taking a long time to complete.

##### max_running_rollup_job_num_per_table

- Unit: -
- Default: 1
- Description: The maximum number of rollup jobs can run in parallel for a table.

##### max_planner_scalar_rewrite_num

- Unit: -
- Default: 100000
- Description: The maximum number of times that the optimizer can rewrite a scalar operator.

##### enable_statistic_collect

- Unit: -
- Default: TRUE
- Description: Whether to collect statistics for the CBO. This feature is enabled by default.

##### enable_collect_full_statistic

- Unit: -
- Default: TRUE
- Description: Whether to enable automatic full statistics collection. This feature is enabled by default.

##### statistic_auto_collect_ratio

- Unit: -
- Default: 0.8
- Description: The threshold for determining whether the statistics for automatic collection are healthy. If statistics health is below this threshold, automatic collection is triggered.

##### statistic_max_full_collect_data_size

- Unit: LONG
- Default: 107374182400
- Description: The size, in bytes, of the largest partition for the automatic collection of statistics. If a partition exceeds this value, then sampled collection is performed instead of full.

##### statistic_collect_max_row_count_per_query

- Unit: INT
- Default: 5000000000
- Description: The maximum number of rows to query for a single analyze task. An analyze task will be split into multiple queries if this value is exceeded.

##### statistic_collect_interval_sec

- Unit: s
- Default: 300
- Description: The interval for checking data updates during automatic collection.

##### statistic_auto_analyze_start_time

- Unit: STRING
- Default: 00:00:00
- Description: The start time of automatic collection. Value range: `00:00:00` - `23:59:59`.

##### statistic_auto_analyze_end_time

- Unit: STRING
- Default: 23:59:59
- Description: The end time of automatic collection. Value range: `00:00:00` - `23:59:59`.

##### statistic_sample_collect_rows

- Unit: -
- Default: 200000
- Description: The minimum number of rows to collect for sampled collection. If the parameter value exceeds the actual number of rows in your table, full collection is performed.

##### histogram_buckets_size

- Unit: -
- Default: 64
- Description: The default bucket number for a histogram.

##### histogram_mcv_size

- Unit: -
- Default: 100
- Description: The number of most common values (MCV) for a histogram.

##### histogram_sample_ratio

- Unit: -
- Default: 0.1
- Description: The sampling ratio for a histogram.

##### histogram_max_sample_row_count

- Unit: -
- Default: 10000000
- Description: The maximum number of rows to collect for a histogram.

##### statistics_manager_sleep_time_sec

- Unit: s
- Default: 60
- Description: The interval at which metadata is scheduled. The system performs the following operations based on this interval:
  - Create tables for storing statistics.
  - Delete statistics that have been deleted.
  - Delete expired statistics.

##### statistic_update_interval_sec

- Unit: s
- Default: `24 * 60 * 60`
- Description: The interval at which the cache of statistical information is updated. Unit: seconds.

##### statistic_analyze_status_keep_second

- Unit: s
- Default: 259200
- Description: The duration to retain the history of collection tasks. The default value is 3 days.

##### statistic_collect_concurrency

- Unit: -
- Default: 3
- Description: The maximum number of manual collection tasks that can run in parallel. The value defaults to 3, which means you can run a maximum of three manual collection tasks in parallel. If the value is exceeded, incoming tasks will be in the PENDING state, waiting to be scheduled.

#### enable_statistic_collect_on_first_load

- Unit: -
- Default: true
- Description: Whether to automatically collect statistics when data is loaded into a table for the first time. If a table has multiple partitions, any data loading into an empty partition of this table will trigger automatic statistics collection on this partition. If new tables are frequently created and data is frequently loaded, the memory and CPU overhead will increase.
- Introduced in: v3.1

##### enable_local_replica_selection

- Unit: -
- Default: FALSE
- Description: Whether to select local replicas for queries. Local replicas reduce the network transmission cost. If this parameter is set to TRUE, the CBO preferentially selects tablet replicas on BEs that have the same IP address as the current FE. If this parameter is set to `FALSE`, both local replicas and non-local replicas can be selected. The default value is FALSE.

##### max_distribution_pruner_recursion_depth

- Unit: -
- Default: 100
- Description: The maximum recursion depth allowed by the partition pruner. Increasing the recursion depth can prune more elements but also increases CPU consumption.

##### enable_udf

- Unit: -
- Default: FALSE
- Description: Whether to enable UDF.

#### Loading and unloading

##### max_broker_load_job_concurrency

- **Unit**: -
- **Default**: 5
- **Description**: The maximum number of concurrent Broker Load jobs allowed within the StarRocks cluster. This parameter is valid only for Broker Load. The value of this parameter must be less than the value of `max_running_txn_num_per_db`. From v2.5 onwards, the default value is changed from `10` to `5`. The alias of this parameter is `async_load_task_pool_size`.

##### load_straggler_wait_second

- **Unit**: s
- **Default**: 300
- **Description**: The maximum loading lag that can be tolerated by a BE replica. If this value is exceeded, cloning is performed to clone data from other replicas. Unit: seconds.

##### desired_max_waiting_jobs

- **Unit**: -
- **Default**: 1024
- **Description**: The maximum number of pending jobs in an FE. The number refers to all jobs, such as table creation, loading, and schema change jobs. If the number of pending jobs in an FE reaches this value, the FE will reject new load requests. This parameter takes effect only for asynchronous loading. From v2.5 onwards, the default value is changed from 100 to 1024.

##### max_load_timeout_second

- **Unit**: s
- **Default**: 259200
- **Description**: The maximum timeout duration allowed for a load job. The load job fails if this limit is exceeded. This limit applies to all types of load jobs.

##### min_load_timeout_second

- **Unit**: s
- **Default**: 1
- **Description**: The minimum timeout duration allowed for a load job. This limit applies to all types of load jobs. Unit: seconds.

##### max_running_txn_num_per_db

- **Unit**: -
- **Default**: 1000
- **Description**: The maximum number of load transactions allowed to be running for each database within a StarRocks cluster. The default value is `1000`. From v3.1 onwards, the default value is changed to `1000` from `100`. When the actual number of load transactions running for a database exceeds the value of this parameter, new load requests will not be processed. New requests for synchronous load jobs will be denied, and new requests for asynchronous load jobs will be placed in queue. We do not recommend you increase the value of this parameter because this will increase system load.

##### load_parallel_instance_num

- **Unit**: -
- **Default**: 1
- **Description**: The maximum number of concurrent loading instances for each load job on a BE.

##### disable_load_job

- **Unit**: -
- **Default**: FALSE
- **Description**: Whether to disable loading when the cluster encounters an error. This prevents any loss caused by cluster errors. The default value is `FALSE`, indicating that loading is not disabled. `TRUE` indicates loading is disabled and the cluster is in read-only state.

##### history_job_keep_max_second

- **Unit**: s
- **Default**: 604800
- **Description**: The maximum duration a historical job can be retained, such as schema change jobs, in seconds.

##### label_keep_max_num

- **Unit**: -
- **Default**: 1000
- **Description**: The maximum number of load jobs that can be retained within a period of time. If this number is exceeded, the information of historical jobs will be deleted.

##### label_keep_max_second

- **Unit**: s
- **Default**: 259200
- **Description**: The maximum duration in seconds to keep the labels of load jobs that have been completed and are in the FINISHED or CANCELLED state. The default value is 3 days. After this duration expires, the labels will be deleted. This parameter applies to all types of load jobs. A value too large consumes a lot of memory.

##### max_routine_load_job_num

- **Unit**: -
- **Default**: 100
- **Description**: The maximum number of Routine Load jobs in a StarRocks cluster. This parameter is deprecated since v3.1.0.

##### max_routine_load_task_concurrent_num

- **Unit**: -
- **Default**: 5
- **Description**: The maximum number of concurrent tasks for each Routine Load job.

##### max_routine_load_task_num_per_be

- **Unit**: -
- **Default**: 16
- **Description**: The maximum number of concurrent Routine Load tasks on each BE. Since v3.1.0, the default value for this parameter is increased to 16 from 5, and no longer needs to be less than or equal to the value of BE static parameter `routine_load_thread_pool_size` (deprecated).

##### max_routine_load_batch_size

- **Unit**: Byte
- **Default**: 4294967296
- **Description**: The maximum amount of data that can be loaded by a Routine Load task, in bytes.

##### routine_load_task_consume_second

- **Unit**: s
- **Default**: 15
- **Description**: The maximum time for each Routine Load task within the cluster to consume data. Since v3.1.0, Routine Load job supports a new parameter `task_consume_second` in [job_properties](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties). This parameter applies to individual load tasks within a Routine Load job, which is more flexible.

##### routine_load_task_timeout_second

- **Unit**: s
- **Default**: 60
- **Description**: The timeout duration for each Routine Load task within the cluster. Since v3.1.0, Routine Load job supports a new parameter `task_timeout_second` in [job_properties](../sql-reference/sql-statements/data-manipulation/CREATE_ROUTINE_LOAD.md#job_properties). This parameter applies to individual load tasks within a Routine Load job, which is more flexible.

#### routine_load_unstable_threshold_second
- **Unit**: s
- **Default**: 3600
- **Description**: Routine Load job is set in the UNSTABLE state if any task within the Routine Load job lags. To be specific, the difference between the timestamp of the message being consumed the current time exceeds this threshold. Also, unconsumed messages exist in the data source.

##### max_tolerable_backend_down_num

- **Unit**: -
- **Default**: 0
- **Description**: The maximum number of faulty BE nodes allowed. If this number is exceeded, Routine Load jobs cannot be automatically recovered.

##### period_of_auto_resume_min

- **Unit**: Min
- **Default**: 5
- **Description**: The interval at which Routine Load jobs are automatically recovered.

##### spark_load_default_timeout_second

- **Unit**: s
- **Default**: 86400
- **Description**: The timeout duration for each Spark Load job, in seconds.

##### spark_home_default_dir

- **Unit**: -
- **Default**: StarRocksFE.STARROCKS_HOME_DIR + "/lib/spark2x"
- **Description**: The root directory of a Spark client.

##### stream_load_default_timeout_second

- **Unit**: s
- **Default**: 600
- **Description**: The default timeout duration for each Stream Load job, in seconds.

##### max_stream_load_timeout_second

- **Unit**: s
- **Default**: 259200
- **Description**: The maximum allowed timeout duration for a Stream Load job, in seconds.

##### insert_load_default_timeout_second

- **Unit**: s
- **Default**: 3600
- **Description**: The timeout duration for the INSERT INTO statement that is used to load data, in seconds.

##### broker_load_default_timeout_second

- **Unit**: s
- **Default**: 14400
- **Description**: The timeout duration for a Broker Load job, in seconds.

##### min_bytes_per_broker_scanner

- **Unit**: Byte
- **Default**: 67108864
- **Description**: The minimum allowed amount of data that can be processed by a Broker Load instance, in bytes.

##### max_broker_concurrency

- **Unit**: -
- **Default**: 100
- **Description**: The maximum number of concurrent instances for a Broker Load task. This parameter is deprecated from v3.1 onwards.

##### export_max_bytes_per_be_per_task

- **Unit**: Byte
- **Default**: 268435456
- **Description**: The maximum amount of data that can be exported from a single BE by a single data unload task, in bytes.

##### export_running_job_num_limit

- **Unit**: -
- **Default**: 5
- **Description**: The maximum number of data exporting tasks that can run in parallel.

##### export_task_default_timeout_second

- **Unit**: s
- **Default**: 7200
- **Description**: The timeout duration for a data exporting task, in seconds.

##### empty_load_as_error

- **Unit**: -
- **Default**: TRUE
- **Description**: Whether to return an error message "all partitions have no load data" if no data is loaded. Values:
  - TRUE: If no data is loaded, the system displays a failure message and returns an error "all partitions have no load data".
  - FALSE: If no data is loaded, the system displays a success message and returns OK, instead of an error.

##### external_table_commit_timeout_ms

- **Unit**: ms
- **Default**: 10000
- **Description**: The timeout duration for committing (publishing) a write transaction to a StarRocks external table. The default value `10000` indicates a 10-second timeout duration.

##### enable_sync_publish

- **Unit**: -
- **Default**: TRUE
- **Description**: Whether to synchronously execute the apply task at the publish phase of a load transaction. This parameter is applicable only to Primary Key tables. Valid values:
  - `TRUE` (default): The apply task is synchronously executed at the publish phase of a load transaction. It means that the load transaction is reported as successful only after the apply task is completed, and the loaded data can truly be queried. When a task loads a large volume of data at a time or loads data frequently, setting this parameter to `true` can improve query performance and stability, but may increase load latency.
  - `FALSE`: The apply task is asynchronously executed at the publish phase of a load transaction. It means that the load transaction is reported as successful after the apply task is submitted, but the loaded data cannot be immediately queried. In this case, concurrent queries need to wait for the apply task to complete or time out before they can continue. When a task loads a large volume of data at a time or loads data frequently, setting this parameter to `false` may affect query performance and stability.
- **Introduced in**: v3.1.4

#### Storage

##### default_replication_num

- **Unit**: -
- **Default**: 3
- **Description**: `default_replication_num` sets the default number of replicas for each data partition when creating a table in StarRocks. This setting can be overridden when creating a table by specifying `replication_num=x` in the CREATE TABLE DDL.

##### enable_strict_storage_medium_check

- **Unit**: -
- **Default**: FALSE
- **Description**: Whether the FE strictly checks the storage medium of BEs when users create tables. If this parameter is set to `TRUE`, the FE checks the storage medium of BEs when users create tables and returns an error if the storage medium of the BE is different from the `storage_medium` parameter specified in the CREATE TABLE statement. For example, the storage medium specified in the CREATE TABLE statement is SSD but the actual storage medium of BEs is HDD. As a result, the table creation fails. If this parameter is `FALSE`, the FE does not check the storage medium of BEs when users create a table.

##### enable_auto_tablet_distribution

- **Unit**: -
- **Default**: TRUE
- **Description**: Whether to automatically set the number of buckets.
  - If this parameter is set to `TRUE`, you don't need to specify the number of buckets when you create a table or add a partition. StarRocks automatically determines the number of buckets.
  - If this parameter is set to `FALSE`, you need to manually specify the number of buckets when you create a table or add a partition. If you do not specify the bucket count when adding a new partition to a table, the new partition inherits the bucket count set at the creation of the table. However, you can also manually specify the number of buckets for the new partition. Starting from version 2.5.7, StarRocks supports setting this parameter.

##### storage_usage_soft_limit_percent

- **Unit**: %
- **Default**: 90
- **Description**: Soft limit of the storage usage percentage in a BE directory. If the storage usage (in percentage) of the BE storage directory exceeds this value and the remaining storage space is less than `storage_usage_soft_limit_reserve_bytes`, tablets cannot be cloned into this directory.

##### storage_usage_soft_limit_reserve_bytes

- **Unit**: Byte
- **Default**: `200 * 1024 * 1024 * 1024`
- **Description**: Soft limit of the remaining storage space in a BE directory. If the remaining storage space in the BE storage directory is less than this value and the storage usage (in percentage) exceeds `storage_usage_soft_limit_percent`, tablets cannot be cloned into this directory.

##### storage_usage_hard_limit_percent

- **Unit**: %
- **Default**: 95
- **Description**: Hard limit of the storage usage percentage in a BE directory. If the storage usage (in percentage) of the BE storage directory exceeds this value and the remaining storage space is less than `storage_usage_hard_limit_reserve_bytes`, Load and Restore jobs are rejected. You need to set this item together with the BE configuration item `storage_flood_stage_usage_percent` to allow the configurations to take effect.

##### storage_usage_hard_limit_reserve_bytes

- **Unit**: Byte
- **Default**: `100 * 1024 * 1024 * 1024`
- **Description**: Hard limit of the remaining storage space in a BE directory. If the remaining storage space in the BE storage directory is less than this value and the storage usage (in percentage) exceeds `storage_usage_hard_limit_percent`, Load and Restore jobs are rejected. You need to set this item together with the BE configuration item `storage_flood_stage_left_capacity_bytes` to allow the configurations to take effect.

##### catalog_trash_expire_second

- **Unit**: s
- **Default**: 86400
- **Description**: The longest duration the metadata can be retained after a database, table, or partition is dropped. If this duration expires, the data will be deleted and cannot be recovered through the [RECOVER](../sql-reference/sql-statements/data-definition/RECOVER.md) command.

##### alter_table_timeout_second

- **Unit**: s
- **Default**: 86400
- **Description**: The timeout duration for the schema change operation (ALTER TABLE). Unit: seconds.

##### recover_with_empty_tablet

- **Unit**: -
- **Default**: FALSE
- **Description**: Whether to replace a lost or corrupted tablet replica with an empty one. If a tablet replica is lost or corrupted, data queries on this tablet or other healthy tablets may fail. Replacing the lost or corrupted tablet replica with an empty tablet ensures that the query can still be executed. However, the result may be incorrect because data is lost. The default value is `FALSE`, which means lost or corrupted tablet replicas are not replaced with empty ones, and the query fails.

##### tablet_create_timeout_second

- **Unit**: s
- **Default**: 10
- **Description**: The timeout duration for creating a tablet, in seconds.

##### tablet_delete_timeout_second

- **Unit**: s
- **Default**: 2
- **Description**: The timeout duration for deleting a tablet, in seconds.

##### check_consistency_default_timeout_second

- **Unit**: s
- **Default**: 600
- **Description**: The timeout duration for a replica consistency check. You can set this parameter based on the size of your tablet.

##### tablet_sched_slot_num_per_path

- **Unit**: -
- **Default**: 8
- **Description**: The maximum number of tablet-related tasks that can run concurrently in a BE storage directory. The alias is `schedule_slot_num_per_path`. From v2.5 onwards, the default value of this parameter is changed from `4` to `8`.

##### tablet_sched_max_scheduling_tablets

- **Unit**: -
- **Default**: 10000
- **Description**: The maximum number of tablets that can be scheduled at the same time. If the value is exceeded, tablet balancing and repair checks will be skipped.

##### tablet_sched_disable_balance

- **Unit**: -
- **Default**: FALSE
- **Description**: Whether to disable tablet balancing. `TRUE` indicates that tablet balancing is disabled. `FALSE` indicates that tablet balancing is enabled. The alias is `disable_balance`.

##### tablet_sched_disable_colocate_balance

- **Unit**: -
- **Default**: FALSE
- **Description**: Whether to disable replica balancing for Colocate Table. `TRUE` indicates replica balancing is disabled. `FALSE` indicates replica balancing is enabled. The alias is `disable_colocate_balance`.

##### tablet_sched_max_balancing_tablets

- **Unit**: -
- **Default**: 500
- **Description**: The maximum number of tablets that can be balanced at the same time. If this value is exceeded, tablet re-balancing will be skipped. The alias is `max_balancing_tablets`.

##### tablet_sched_balance_load_disk_safe_threshold

- **Unit**: -
- **Default**: 0.5
- **Description**: The percentage threshold for determining whether the disk usage of BEs is balanced. If the disk usage of all BEs is lower than this value, it is considered balanced. If the disk usage is greater than this value and the difference between the highest and lowest BE disk usage is greater than 10%, the disk usage is considered unbalanced and a tablet re-balancing is triggered. The alias is `balance_load_disk_safe_threshold`.

##### tablet_sched_balance_load_score_threshold

- **Unit**: -
- **Default**: 0.1
- **Description**: The percentage threshold for determining whether the load of a BE is balanced. If a BE has a lower load than the average load of all BEs and the difference is greater than this value, this BE is in a low load state. On the contrary, if a BE has a higher load than the average load and the difference is greater than this value, this BE is in a high load state. The alias is `balance_load_score_threshold`.

##### tablet_sched_repair_delay_factor_second

- **Unit**: s
- **Default**: 60
- **Description**: The interval at which replicas are repaired, in seconds. The alias is `tablet_repair_delay_factor_second`.

##### tablet_sched_min_clone_task_timeout_sec

- **Unit**: s
- **Default**: 3 * 60
- **Description**: The minimum timeout duration for cloning a tablet, in seconds.

##### tablet_sched_max_clone_task_timeout_sec

- **Unit**: s
- **Default**: `2 * 60 * 60`
- **Description**: The maximum timeout duration for cloning a tablet, in seconds. The alias is `max_clone_task_timeout_sec`.

##### tablet_sched_max_not_being_scheduled_interval_ms

- **Unit**: ms
- **Default**: `15 * 60 * 100`
- **Description**: When the tablet clone tasks are being scheduled, if a tablet has not been scheduled for the specified time in this parameter, StarRocks gives it a higher priority to schedule it as soon as possible.

#### Shared-data specific

##### lake_compaction_score_selector_min_score

- **Default**: 10.0
- **Description**: The Compaction Score threshold that triggers Compaction operations. When the Compaction Score of a partition is greater than or equal to this value, the system performs Compaction on that partition.
- **Introduced in**: v3.1.0

The Compaction Score indicates whether a partition needs Compaction and is scored based on the number of files in the partition. Excessive number of files can impact query performance, so the system periodically performs Compaction to merge small files and reduce the file count. You can check the Compaction Score for a partition based on the `MaxCS` column in the result returned by running [SHOW PARTITIONS](../sql-reference/sql-statements/data-manipulation/SHOW_PARTITIONS.md).

##### lake_compaction_max_tasks

- **Default**: -1
- **Description**: The maximum number of concurrent Compaction tasks allowed.
- **Introduced in**: v3.1.0

The system calculates the number of Compaction tasks based on the number of tablets in a partition. For example, if a partition has 10 tablets, performing one Compaction on that partition creates 10 Compaction tasks. If the number of concurrently executing Compaction tasks exceeds this threshold, the system will not create new Compaction tasks. Setting this item to `0` disables Compaction, and setting it to `-1` allows the system to automatically calculate this value based on an adaptive strategy.

##### lake_compaction_history_size

- **Default**: 12
- **Description**: The number of recent successful Compaction task records to keep in the memory of the Leader FE node. You can view recent successful Compaction task records using the `SHOW PROC '/compactions'` command. Note that the Compaction history is stored in the FE process memory, and it will be lost if the FE process is restarted.
- **Introduced in**: v3.1.0

##### lake_compaction_fail_history_size

- **Default**: 12
- **Description**: The number of recent failed Compaction task records to keep in the memory of the Leader FE node. You can view recent failed Compaction task records using the `SHOW PROC '/compactions'` command. Note that the Compaction history is stored in the FE process memory, and it will be lost if the FE process is restarted.
- **Introduced in**: v3.1.0

##### lake_autovacuum_parallel_partitions

- **Default**: 8
- **Description**: The maximum number of partitions that can undergo AutoVacuum simultaneously. AutoVaccum is the Garbage Collection after Compactions.
- **Introduced in**: v3.1.0

##### lake_autovacuum_partition_naptime_seconds

- **Unit**: Second
- **Default**: 180
- **Description**: The minimum interval between AutoVacuum operations on the same partition.
- **Introduced in**: v3.1.0

##### lake_autovacuum_grace_period_minutes

- **Unit**: Minutes
- **Default**: 5
- **Description**: The time range for retaining historical data versions. Historical data versions within this time range are not automatically cleaned via AutoVacuum after Compactions. You need to set this value greater than the maximum query time to avoid that the data accessed by running queries get deleted before the queries finish.
- **Introduced in**: v3.1.0

##### lake_autovacuum_stale_partition_threshold

- **Unit**: Hours
- **Default**: 12
- **Description**: If a partition has no updates (loading, DELETE, or Compactions) within this time range, the system will not perform AutoVacuum on this partition.
- **Introduced in**: v3.1.0

#### Other FE dynamic parameters

##### plugin_enable

- **Unit**: -
- **Default**: TRUE
- **Description**: Whether plugins can be installed on FEs. Plugins can be installed or uninstalled only on the Leader FE.

##### max_small_file_number

- **Unit**: -
- **Default**: 100
- **Description**: The maximum number of small files that can be stored on an FE directory.

##### max_small_file_size_bytes

- **Unit**: Byte
- **Default**: 1024 * 1024
- **Description**: The maximum size of a small file, in bytes.

##### agent_task_resend_wait_time_ms

- **Unit**: ms
- **Default**: 5000
- **Description**: The duration the FE must wait before it can resend an agent task. An agent task can be resent only when the gap between the task creation time and the current time exceeds the value of this parameter. This parameter is used to prevent repetitive sending of agent tasks. Unit: ms.

##### backup_job_default_timeout_ms

- **Unit**: ms
- **Default**: 86400*1000
- **Description**: The timeout duration of a backup job. If this value is exceeded, the backup job fails.

##### report_queue_size

- **Unit**: -
- **Default**: 100
- **Description**: The maximum number of jobs that can wait in a report queue. The report is about disk, task, and tablet information of BEs. If too many report jobs are piling up in a queue, OOM will occur.

##### enable_experimental_mv

- **Unit**: -
- **Default**: TRUE
- **Description**: Whether to enable the asynchronous materialized view feature. TRUE indicates this feature is enabled. From v2.5.2 onwards, this feature is enabled by default. For versions earlier than v2.5.2, this feature is disabled by default.

##### authentication_ldap_simple_bind_base_dn

- **Unit**: -
- **Default**: Empty string
- **Description**: The base DN, which is the point from which the LDAP server starts to search for users' authentication information.

##### authentication_ldap_simple_bind_root_dn

- **Unit**: -
- **Default**: Empty string
- **Description**: The administrator DN used to search for users' authentication information.

##### authentication_ldap_simple_bind_root_pwd

- **Unit**: -
- **Default**: Empty string
- **Description**: The password of the administrator used to search for users' authentication information.

##### authentication_ldap_simple_server_host

- **Unit**: -
- **Default**: Empty string
- **Description**: The host on which the LDAP server runs.

##### authentication_ldap_simple_server_port

- **Unit**: -
- **Default**: 389
- **Description**: The port of the LDAP server.

##### authentication_ldap_simple_user_search_attr

- **Unit**: -
- **Default**: uid
- **Description**: The name of the attribute that identifies users in LDAP objects.

##### max_upload_task_per_be

- **Unit**: -
- **Default**: 0
- **Description**: In each BACKUP operation, the maximum number of upload tasks StarRocks assigned to a BE node. When this item is set to less than or equal to 0, no limit is imposed on the task number. This item is supported from v3.1.0 onwards.

##### max_download_task_per_be

- **Unit**: -
- **Default**: 0
- **Description**: In each RESTORE operation, the maximum number of download tasks StarRocks assigned to a BE node. When this item is set to less than or equal to 0, no limit is imposed on the task number. This item is supported from v3.1.0 onwards.

##### allow_system_reserved_names

- **Default**: FALSE
- Whether to allow users to create columns whose names initiated with `__op` and `__row`. To enable this feaure, set this paramter to `TRUE`. Please note that thess name formats are reserved for special purposes in StarRocks and creating such columns may result in undefined behavior. Therefore this feature is disabled by default. This item is supported from v3.1.5 onwards.

##### enable_mv_automatic_active_check

- **Default**: TRUE
- **Description**: Whether to enable the system to automatically check and re-activate the asynchronous materialized views that are set inactive because their base tables (views) had undergone Schema Change or had been dropped and re-created. Please note that this feature will not re-activate the materialized views that are manually set inactive by users. This item is supported from v3.1.6 onwards.

##### default_mv_refresh_immediate

- **Unit**: -
- **Default**: TRUE
- **Description**: Whether to refresh an asynchronous materialized view immediately after creation. When this item is set to `true`, newly created materialized view will be refreshed immediately.
- **Introduced in**: v3.1.7

### Configure FE static parameters

This section provides an overview of the static parameters that you can configure in the FE configuration file **fe.conf**. After you reconfigure these parameters for an FE, you must restart the FE for the changes to take effect.

#### Logging

##### log_roll_size_mb

- **Default:** 1024 MB
- **Description:** The size per log file. Unit: MB. The default value `1024` specifies the size per log file as 1 GB.

##### sys_log_dir

- **Default:** StarRocksFE.STARROCKS_HOME_DIR + "/log"
- **Description:** The directory that stores system log files.

##### sys_log_level

- **Default:** INFO
- **Description:** The severity levels into which system log entries are classified. Valid values: `INFO`, `WARN`, `ERROR`, and `FATAL`.

##### sys_log_verbose_modules

- **Default:** Empty string
- **Description:** The modules for which StarRocks generates system logs. If this parameter is set to `org.apache.starrocks.catalog`, StarRocks generates system logs only for the catalog module.

##### sys_log_roll_interval

- **Default:** DAY
- **Description:** The time interval at which StarRocks rotates system log entries. Valid values: `DAY` and `HOUR`.
  - If this parameter is set to `DAY`, a suffix in the `yyyyMMdd` format is added to the names of system log files.
  - If this parameter is set to `HOUR`, a suffix in the `yyyyMMddHH` format is added to the names of system log files.

##### sys_log_delete_age

- **Default:** 7d
- **Description:** The retention period of system log files. The default value `7d` specifies that each system log file can be retained for 7 days. StarRocks checks each system log file and deletes those that were generated 7 days ago.

##### sys_log_roll_num

- **Default:** 10
- **Description:** The maximum number of system log files that can be retained within each retention period specified by the `sys_log_roll_interval` parameter.

##### audit_log_dir

- **Default:** StarRocksFE.STARROCKS_HOME_DIR + "/log"
- **Description:** The directory that stores audit log files.

##### audit_log_roll_num

- **Default:** 90
- **Description:** The maximum number of audit log files that can be retained within each retention period specified by the `audit_log_roll_interval` parameter.

##### audit_log_modules

- **Default:** slow_query, query
- **Description:** The modules for which StarRocks generates audit log entries. By default, StarRocks generates audit logs for the slow_query module and the query module. Separate the module names with a comma (,) and a space.

##### audit_log_roll_interval

- **Default:** DAY
- **Description:** The time interval at which StarRocks rotates audit log entries. Valid values: `DAY` and `HOUR`.
- If this parameter is set to `DAY`, a suffix in the `yyyyMMdd` format is added to the names of audit log files.
- If this parameter is set to `HOUR`, a suffix in the `yyyyMMddHH` format is added to the names of audit log files.

##### audit_log_delete_age

- **Default:** 30d
- **Description:** The retention period of audit log files. The default value `30d` specifies that each audit log file can be retained for 30 days. StarRocks checks each audit log file and deletes those that were generated 30 days ago.

##### dump_log_dir

- **Default:** StarRocksFE.STARROCKS_HOME_DIR + "/log"
- **Description:** The directory that stores dump log files.

##### dump_log_modules

- **Default:** query
- **Description:** The modules for which StarRocks generates dump log entries. By default, StarRocks generates dump logs for the query module. Separate the module names with a comma (,) and a space.

##### dump_log_roll_interval

- **Default:** DAY
- **Description:** The time interval at which StarRocks rotates dump log entries. Valid values: `DAY` and `HOUR`.
  - If this parameter is set to `DAY`, a suffix in the `yyyyMMdd` format is added to the names of dump log files.
  - If this parameter is set to `HOUR`, a suffix in the `yyyyMMddHH` format is added to the names of dump log files.

##### dump_log_roll_num

- **Default:** 10
- **Description:** The maximum number of dump log files that can be retained within each retention period specified by the `dump_log_roll_interval` parameter.

##### dump_log_delete_age

- **Default:** 7d
- **Description:** The retention period of dump log files. The default value `7d` specifies that each dump log file can be retained for 7 days. StarRocks checks each dump log file and deletes those that were generated 7 days ago.

#### Server

##### frontend_address

- **Default:** 0.0.0.0
- **Description:** The IP address of the FE node.

##### priority_networks

- **Default:** Empty string
- **Description:** Declares a selection strategy for servers that have multiple IP addresses. Note that at most one IP address must match the list specified by this parameter. The value of this parameter is a list that consists of entries, which are separated with semicolons (;) in CIDR notation, such as 10.10.10.0/24. If no IP address matches the entries in this list, an IP address will be randomly selected.

##### http_port

- **Default:** 8030
- **Description:** The port on which the HTTP server in the FE node listens.

##### http_worker_threads_num

- **Default:** 0
- **Description:** Number of worker threads for http server to deal with http requests. For a negative or 0 value, the number of threads will be twice the number of cpu cores.
- Introduced in: 2.5.18, 3.0.10, 3.1.7, 3.2.2

##### http_backlog_num

- **Default:** 1024
- **Description:** The length of the backlog queue held by the HTTP server in the FE node.

##### cluster_name

- **Default:** StarRocks Cluster
- **Description:** The name of the StarRocks cluster to which the FE belongs. The cluster name is displayed for `Title` on the web page.

##### rpc_port

- **Default:** 9020
- **Description:** The port on which the Thrift server in the FE node listens.

##### thrift_backlog_num

- **Default:** 1024
- **Description:** The length of the backlog queue held by the Thrift server in the FE node.

##### thrift_server_max_worker_threads

- **Default:** 4096
- **Description:** The maximum number of worker threads that are supported by the Thrift server in the FE node.

##### thrift_client_timeout_ms

- **Default:** 5000
- **Description:** The length of time after which idle client connections time out. Unit: ms.

##### thrift_server_queue_size

- **Default:** 4096
- **Description:** The length of queue where requests are pending. If the number of threads that are being processed in the thrift server exceeds the value specified in `thrift_server_max_worker_threads`, new requests are added to the pending queue.

##### brpc_idle_wait_max_time

- **Default:** 10000
- **Description:** The maximum length of time for which bRPC clients wait as in the idle state. Unit: ms.

##### query_port

- **Default:** 9030
- **Description:** The port on which the MySQL server in the FE node listens.

##### mysql_service_nio_enabled

- **Default:** TRUE
- **Description:** Specifies whether asynchronous I/O is enabled for the FE node.

##### mysql_service_io_threads_num

- **Default:** 4
- **Description:** The maximum number of threads that can be run by the MySQL server in the FE node to process I/O events.

##### mysql_nio_backlog_num

- **Default:** 1024
- **Description:** The length of the backlog queue held by the MySQL server in the FE node.

##### max_mysql_service_task_threads_num

- **Default:** 4096
- **Description:** The maximum number of threads that can be run by the MySQL server in the FE node to process tasks.

##### mysql_server_version

- **Default:** 5.1.0
- **Description:** The MySQL server version returned to the client. Modifying this parameter will affect the version information in the following situations:
  1. `select version();`
  2. Handshake packet version
  3. Value of the global variable `version` (`show variables like 'version';`)

##### max_connection_scheduler_threads_num

- **Default:** 4096
- **Description:** The maximum number of threads that are supported by the connection scheduler.

##### qe_max_connection

- **Default:** 1024
- **Description:** The maximum number of connections that can be established by all users to the FE node.

##### check_java_version

- **Default:** TRUE
- **Description:** Specifies whether to check version compatibility between the executed and compiled Java programs. If the versions are incompatible, StarRocks reports errors and aborts the startup of Java programs.

#### Metadata and cluster management

##### meta_dir

- **Default:** StarRocksFE.STARROCKS_HOME_DIR + "/meta"
- **Description:** The directory that stores metadata.

##### heartbeat_mgr_threads_num

- **Default:** 8
- **Description:** The number of threads that can be run by the Heartbeat Manager to run heartbeat tasks.

##### heartbeat_mgr_blocking_queue_size

- **Default:** 1024
- **Description:** The size of the blocking queue that stores heartbeat tasks run by the Heartbeat Manager.

##### metadata_failure_recovery

- **Default:** FALSE
- **Description:** Specifies whether to forcibly reset the metadata of the FE. Exercise caution when you set this parameter.

##### edit_log_port

- **Default:** 9010
- **Description:** The port that is used for communication among the leader, follower, and observer FEs in the StarRocks cluster.

##### edit_log_type

- **Default:** BDB
- **Description:** The type of edit log that can be generated. Set the value to `BDB`.

##### bdbje_heartbeat_timeout_second

- **Default:** 30
- **Description:** The amount of time after which the heartbeats among the leader, follower, and observer FEs in the StarRocks cluster time out. Unit: second.

##### bdbje_lock_timeout_second

- **Default:** 1
- **Description:** The amount of time after which a lock in the BDB JE-based FE times out. Unit: second.

##### max_bdbje_clock_delta_ms

- **Default:** 5000
- **Description:** The maximum clock offset that is allowed between the leader FE and the follower or observer FEs in the StarRocks cluster. Unit: ms.

##### txn_rollback_limit

- **Default:** 100
- **Description:** The maximum number of transactions that can be rolled back.

##### bdbje_replica_ack_timeout_second

- **Default:** 10
- **Description:** The maximum amount of time for which the leader FE can wait for ACK messages from a specified number of follower FEs when metadata is written from the leader FE to the follower FEs. Unit: second. If a large amount of metadata is being written, the follower FEs require a long time before they can return ACK messages to the leader FE, causing ACK timeout. In this situation, metadata writes fail, and the FE process exits. We recommend that you increase the value of this parameter to prevent this situation.

##### master_sync_policy

- **Default:** SYNC
- **Description:** The policy based on which the leader FE flushes logs to disk. This parameter is valid only when the current FE is a leader FE. Valid values:
  - `SYNC`: When a transaction is committed, a log entry is generated and flushed to disk simultaneously.
  - `NO_SYNC`: The generation and flushing of a log entry do not occur at the same time when a transaction is committed.
  - `WRITE_NO_SYNC`: When a transaction is commited, a log entry is generated simultaneously but is not flushed to disk. If you have deployed only one follower FE, we recommend that you set this parameter to `SYNC`. If you have deployed three or more follower FEs, we recommend that you set this parameter and the `replica_sync_policy` both to `WRITE_NO_SYNC`.

##### replica_sync_policy

- **Default:** SYNC
- **Description:** The policy based on which the follower FE flushes logs to disk. This parameter is valid only when the current FE is a follower FE. Valid values:
  - `SYNC`: When a transaction is committed, a log entry is generated and flushed to disk simultaneously.
  - `NO_SYNC`: The generation and flushing of a log entry do not occur at the same time when a transaction is committed.
  - `WRITE_NO_SYNC`: When a transaction is committed, a log entry is generated simultaneously but is not flushed to disk.

##### replica_ack_policy

- **Default:** SIMPLE_MAJORITY
- **Description:** The policy based on which a log entry is considered valid. The default value `SIMPLE_MAJORITY` specifies that a log entry is considered valid if a majority of follower FEs return ACK messages.

##### cluster_id

- **Default:** -1
- **Description:** The ID of the StarRocks cluster to which the FE belongs. FEs or BEs that have the same cluster ID belong to the same StarRocks cluster. Valid values: any positive integer. The default value `-1` specifies that StarRocks will generate a random cluster ID for the StarRocks cluster at the time when the leader FE of the cluster is started for the first time.

#### Query engine

##### publish_version_interval_ms

- **Default:** 10
- **Description:** The time interval at which release validation tasks are issued. Unit: ms.

##### statistic_cache_columns

- **Default:** 100000
- **Description:** The number of rows that can be cached for the statistics table.

##### statistic_cache_thread_pool_size

- **Default:** 10
- **Description:** The size of the thread-pool which will be used to refresh statistic caches.

#### Loading and unloading

##### load_checker_interval_second

- **Default:** 5
- **Description:** The time interval at which load jobs are processed on a rolling basis. Unit: second.

##### transaction_clean_interval_second

- **Default:** 30
- **Description:** The time interval at which finished transactions are cleaned up. Unit: second. We recommend that you specify a short time interval to ensure that finished transactions can be cleaned up in a timely manner.

##### label_clean_interval_second

- **Default:** 14400
- **Description:** The time interval at which labels are cleaned up. Unit: second. We recommend that you specify a short time interval to ensure that historical labels can be cleaned up in a timely manner.

##### spark_dpp_version

- **Default:** 1.0.0
- **Description:** The version of Spark Dynamic Partition Pruning (DPP) used.

##### spark_resource_path

- **Default:** Empty string
- **Description:** The root directory of the Spark dependency package.

##### spark_launcher_log_dir

- **Default:** sys_log_dir + "/spark_launcher_log"
- **Description:** The directory that stores Spark log files.

##### yarn_client_path

- **Default:** StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-client/hadoop/bin/yarn"
- **Description:** The root directory of the Yarn client package.

##### yarn_config_dir

- **Default:** StarRocksFE.STARROCKS_HOME_DIR + "/lib/yarn-config"
- **Description:** The directory that stores the Yarn configuration file.

##### export_checker_interval_second

- **Default:** 5
- **Description:** The time interval at which load jobs are scheduled.

##### export_task_pool_size

- **Default:** 5
- **Description:** The size of the unload task thread pool.

#### Storage

##### default_storage_medium

- **Default:** HDD
- **Description:** The default storage media that is used for a table or partition at the time of table or partition creation if no storage media is specified. Valid values: `HDD` and `SSD`. When you create a table or partition, the default storage media specified by this parameter is used if you do not specify a storage media type for the table or partition.

##### tablet_sched_storage_cooldown_second

- **Default:** -1
- **Description:** The latency of automatic cooling starting from the time of table creation. The alias of this parameter is `storage_cooldown_second`. Unit: second. The default value `-1` specifies that automatic cooling is disabled. If you want to enable automatic cooling, set this parameter to a value greater than `-1`.

##### tablet_stat_update_interval_second

- **Default:** 300
- **Description:** The time interval at which the FE retrieves tablet statistics from each BE. Unit: second.

#### StarRocks shared-data cluster

##### run_mode

- **Default**: shared_nothing
- **Description**: The running mode of the StarRocks cluster. Valid values: shared_data and shared_nothing (Default).

  - shared_data indicates running StarRocks in shared-data mode.
  - shared_nothing indicates running StarRocks in shared-nothing mode.
CAUTION
You cannot adopt the shared_data and shared_nothing modes simultaneously for a StarRocks cluster. Mixed deployment is not supported.
DO NOT change run_mode after the cluster is deployed. Otherwise, the cluster fails to restart. The transformation from a shared-nothing cluster to a shared-data cluster or vice versa is not supported.

##### cloud_native_meta_port

- **Default**: 6090
- **Description**: The cloud-native meta service RPC port.

##### cloud_native_storage_type

- **Default**: S3
- **Description**: The type of object storage you use. In shared-data mode, StarRocks supports storing data in Azure Blob (supported from v3.1.1 onwards), and object storages that are compatible with the S3 protocol (such as AWS S3, Google GCP, and MinIO). Valid value: S3 (Default) and AZBLOB. If you specify this parameter as S3, you must add the parameters prefixed by aws_s3. If you specify this parameter as AZBLOB, you must add the parameters prefixed by azure_blob.

##### aws_s3_path

- **Default**: N/A
- **Description**: The S3 path used to store data. It consists of the name of your S3 bucket and the sub-path (if any) under it, for example, `testbucket/subpath`.

##### aws_s3_endpoint

- **Default**: N/A
- **Description**: The endpoint used to access your S3 bucket, for example, `https://s3.us-west-2.amazonaws.com`.

##### aws_s3_region

- **Default**: N/A
- **Description**: The region in which your S3 bucket resides, for example, `us-west-2`.

##### aws_s3_use_aws_sdk_default_behavior

- **Default**: false
- **Description**: Whether to use the default authentication credential of AWS SDK. Valid values: true and false (Default).

##### aws_s3_use_instance_profile

- **Default**: false
- **Description**: Whether to use Instance Profile and Assumed Role as credential methods for accessing S3. Valid values: true and false (Default).
  - If you use IAM user-based credential (Access Key and Secret Key) to access S3, you must specify this item as false, and specify aws_s3_access_key and aws_s3_secret_key.
  - If you use Instance Profile to access S3, you must specify this item as true.
  - If you use Assumed Role to access S3, you must specify this item as true, and specify aws_s3_iam_role_arn.
  - And if you use an external AWS account, you must also specify aws_s3_external_id.

##### aws_s3_access_key

- **Default**: N/A
- **Description**: The Access Key ID used to access your S3 bucket.

##### aws_s3_secret_key

- **Default**: N/A
- **Description**: The Secret Access Key used to access your S3 bucket.

##### aws_s3_iam_role_arn

- **Default**: N/A
- **Description**: The ARN of the IAM role that has privileges on your S3 bucket in which your data files are stored.

##### aws_s3_external_id

- **Default**: N/A
- **Description**: The external ID of the AWS account that is used for cross-account access to your S3 bucket.

##### azure_blob_path

- **Default**: N/A
- **Description**: The Azure Blob Storage path used to store data. It consists of the name of the container within your storage account and the sub-path (if any) under the container, for example, testcontainer/subpath.

##### azure_blob_endpoint

- **Default**: N/A
- **Description**: The endpoint of your Azure Blob Storage Account, for example, `https://test.blob.core.windows.net`.

##### azure_blob_shared_key

- **Default**: N/A
- **Description**: The Shared Key used to authorize requests for your Azure Blob Storage.

##### azure_blob_sas_token

- **Default**: N/A
- **Description**: The shared access signatures (SAS) used to authorize requests for your Azure Blob Storage.

#### Other FE static parameters

##### plugin_dir

- **Default:** STARROCKS_HOME_DIR/plugins
- **Description:** The directory that stores plugin installation packages.

##### small_file_dir

- **Default:** StarRocksFE.STARROCKS_HOME_DIR + "/small_files"
- **Description:** The root directory of small files.

##### max_agent_task_threads_num

- **Default:** 4096
- **Description:** The maximum number of threads that are allowed in the agent task thread pool.

##### auth_token

- **Default:** Empty string
- **Description:** The token that is used for identity authentication within the StarRocks cluster to which the FE belongs. If this parameter is left unspecified, StarRocks generates a random token for the cluster at the time when the leader FE of the cluster is started for the first time.

##### tmp_dir

- **Default:** StarRocksFE.STARROCKS_HOME_DIR + "/temp_dir"
- **Description:** The directory that stores temporary files such as files generated during backup and restore procedures. After these procedures finish, the generated temporary files are deleted.

##### locale

- **Default:** zh_CN.UTF-8
- **Description:** The character set that is used by the FE.

##### hive_meta_load_concurrency

- **Default:** 4
- **Description:** The maximum number of concurrent threads that are supported for Hive metadata.

##### hive_meta_cache_refresh_interval_s

- **Default:** 7200
- **Description:** The time interval at which the cached metadata of Hive external tables is updated. Unit: second.

##### hive_meta_cache_ttl_s

- **Default:** 86400
- **Description:** The amount of time after which the cached metadata of Hive external tables expires. Unit: second.

##### hive_meta_store_timeout_s

- **Default:** 10
- **Description:** The amount of time after which a connection to a Hive metastore times out. Unit: second.

##### es_state_sync_interval_second

- **Default:** 10
- **Description:** The time interval at which the FE obtains Elasticsearch indexes and synchronizes the metadata of StarRocks external tables. Unit: second.

##### enable_auth_check

- **Default:** TRUE
- **Description:** Specifies whether to enable the authentication check feature. Valid values: `TRUE` and `FALSE`. `TRUE` specifies to enable this feature, and `FALSE` specifies to disable this feature.

##### enable_metric_calculator

- **Default:** TRUE
- **Description:** Specifies whether to enable the feature that is used to periodically collect metrics. Valid values: `TRUE` and `FALSE`. `TRUE` specifies to enable this feature, and `FALSE` specifies to disable this feature.

## BE configuration items

Some BE configuration items are dynamic parameters which you can set them by commands when BE nodes are still online. The rest of them are static parameters. You can only set the static parameters of a BE node by changing them in the corresponding configuration file **be.conf**, and restart the BE node to allow the change to take effect.

### View BE configuration items

You can view the BE configuration items using the following command:

```shell
curl http://<BE_IP>:<BE_HTTP_PORT>/varz
```

### Configure BE dynamic parameters

You can configure a dynamic parameter of a BE node by using the `curl` command.

```Shell
curl -XPOST http://be_host:http_port/api/update_config?<configuration_item>=<value>
```

BE dynamic parameters are as follows.

#### enable_stream_load_verbose_log

- **Default:** false
- **Description:** Specifies whether to log the HTTP requests and responses for Stream Load jobs.
- **Introduced in:** 2.5.17, 3.0.9, 3.1.6, 3.2.1

#### report_task_interval_seconds

- **Default:** 10 seconds
- **Description:** The time interval at which to report the state of a task. A task can be creating a table, dropping a table, loading data, or changing a table schema.

#### report_disk_state_interval_seconds

- **Default:** 60 seconds
- **Description:** The time interval at which to report the storage volume state, which includes the size of data within the volume.

#### report_tablet_interval_seconds

- **Default:** 60 seconds
- **Description:** The time interval at which to report the most updated version of all tablets.

#### report_workgroup_interval_seconds

- **Default:** 5 seconds
- **Description:** The time interval at which to report the most updated version of all workgroups.

#### max_download_speed_kbps

- **Default:** 50,000 KB/s
- **Description:** The maximum download speed of each HTTP request. This value affects the performance of data replica synchronization across BE nodes.

#### download_low_speed_limit_kbps

- **Default:** 50 KB/s
- **Description:** The download speed lower limit of each HTTP request. An HTTP request aborts when it constantly runs with a lower speed than this value within the time span specified in the configuration item `download_low_speed_time`.

#### download_low_speed_time

- **Default:** 300 seconds
- **Description:** The maximum time that an HTTP request can run with a download speed lower than the limit. An HTTP request aborts when it constantly runs with a lower speed than the value of `download_low_speed_limit_kbps` within the time span specified in this configuration item.

#### status_report_interval

- **Default:** 5 seconds
- **Description:** The time interval at which a query reports its profile, which can be used for query statistics collection by FE.

#### scanner_thread_pool_thread_num

- **Default:** 48 (Number of Threads)
- **Description:** The number of threads which the storage engine used for concurrent storage volume scanning. All threads are managed in the thread pool.

#### thrift_client_retry_interval_ms

- **Default:** 100 ms
- **Description:** The time interval at which a thrift client retries.

#### scanner_thread_pool_queue_size

- **Default:** 102,400
- **Description:** The number of scan tasks supported by the storage engine.

#### scanner_row_num

- **Default:** 16,384
- **Description:** The maximum row count returned by each scan thread in a scan.

#### max_scan_key_num

- **Default:** 1,024 (Maximum Number of Scan Keys per Query)
- **Description:** The maximum number of scan keys segmented by each query.

#### max_pushdown_conditions_per_column

- **Default:** 1,024
- **Description:** The maximum number of conditions that allow pushdown in each column. If the number of conditions exceeds this limit, the predicates are not pushed down to the storage layer.

#### exchg_node_buffer_size_bytes

- **Default:** 10,485,760 Bytes
- **Description:** The maximum buffer size on the receiver end of an exchange node for each query. This configuration item is a soft limit. A backpressure is triggered when data is sent to the receiver end with an excessive speed.

#### memory_limitation_per_thread_for_schema_change

- **Default:** 2 GB
- **Description:** The maximum memory size allowed for each schema change task.

#### update_cache_expire_sec

- **Default:** 360 seconds
- **Description:** The expiration time of Update Cache.

#### file_descriptor_cache_clean_interval

- **Default:** 3,600 seconds
- **Description:** The time interval at which to clean file descriptors that have not been used for a certain period of time.

#### disk_stat_monitor_interval

- **Default:** 5 seconds
- **Description:** The time interval at which to monitor health status of disks.

#### unused_rowset_monitor_interval

- **Default:** 30 seconds
- **Description:** The time interval at which to clean the expired rowsets.

#### max_percentage_of_error_disk

- **Default:** 0%
- **Description:** The maximum percentage of error that is tolerable in a storage volume before the corresponding BE node quits.

#### default_num_rows_per_column_file_block

- **Default:** 1,024
- **Description:** The maximum number of rows that can be stored in each row block.

#### pending_data_expire_time_sec

- **Default:** 1,800 seconds
- **Description:** The expiration time of the pending data in the storage engine.

#### inc_rowset_expired_sec

- **Default:** 1,800 seconds
- **Description:** The expiration time of the incoming data. This configuration item is used in incremental clone.

#### tablet_rowset_stale_sweep_time_sec

- **Default:** 1,800 seconds
- **Description:** The time interval at which to sweep the stale rowsets in tablets. A shorter interval can reduce metadata usage during loading.

#### snapshot_expire_time_sec

- **Default:** 172,800 seconds
- **Description:** The expiration time of snapshot files. The system cleans snapshot files at a specified interval. If a snapshot is retained for a duration longer than the value of this parameter, it will be cleaned. The clean interval depends on disk space usage. If the disk space usage is less than 60%, the clean interval is `max_garbage_sweep_interval`. If the disk space usage is larger than 80%, the clean interval is `min_garbage_sweep_interval`.

#### trash_file_expire_time_sec

- **Default:** 86,400 seconds
- **Description:** The maximum duration files can be retained in trash. The system cleans trash files at a specified interval. If a trash file is retained for a duration longer than the value of this parameter, it will be cleaned. The clean interval depends on disk space usage. If the disk space usage is less than 60%, the clean interval is `max_garbage_sweep_interval`. If the disk space usage is larger than 80%, the clean interval is `min_garbage_sweep_interval`. Since v2.5.17, v3.0.9, and v3.1.6, the default value has been changed from 259200 to 86400.

#### base_compaction_check_interval_seconds

- **Default:** 60 seconds
- **Description:** The time interval of thread polling for a Base Compaction.

#### min_base_compaction_num_singleton_deltas

- **Default:** 5
- **Description:** The minimum number of segments that trigger a Base Compaction.

#### max_base_compaction_num_singleton_deltas

- **Default:** 100
- **Description:** The maximum number of segments that can be compacted in each Base Compaction.

#### base_compaction_interval_seconds_since_last_operation

- **Default:** 86,400 seconds
- **Description:** The time interval since the last Base Compaction. This configuration item is one of the conditions that trigger a Base Compaction.

#### cumulative_compaction_check_interval_seconds

- **Default:** 1 second
- **Description:** The time interval of thread polling for a Cumulative Compaction.

#### min_cumulative_compaction_num_singleton_deltas

- **Default:** 5
- **Description:** The minimum number of segments to trigger Cumulative Compaction.

#### max_cumulative_compaction_num_singleton_deltas

- **Default:** 1000
- **Description:** The maximum number of segments that can be merged in a single Cumulative Compaction. You can reduce this value if OOM occurs during compaction.

#### max_compaction_candidate_num

- **Default:** 40960
- **Description:** The maximum number of candidate tablets for compaction. If the value is too large, it will cause high memory usage and high CPU load.

#### update_compaction_check_interval_seconds

- **Default:** 60
- **Unit:** Second
- **Description:** The time interval at which to check compaction for Primary Key tables.

#### update_compaction_num_threads_per_disk

- **Default:** 1
- **Description:** The number of Compaction threads per disk for Primary Key tables.

#### update_compaction_per_tablet_min_interval_seconds

- **Default:** 120
- **Description:** The minimum time interval at which compaction is triggered for each tablet in a Primary Key table.

#### max_update_compaction_num_singleton_deltas

- **Default:** 1000
- **Description:** The maximum number of rowsets that can be merged in a single Compaction for Primary Key tables.

#### update_compaction_size_threshold

- **Default:** 268435456
- **Unit:** Byte
- **Description:** The Compaction Score of Primary Key tables is calculated based on the file size, which is different from other table types. This parameter can be used to make the Compaction Score of Primary Key tables similar to that of other table types, making it easier for users to understand.

#### update_compaction_result_bytes

- **Default:** 1073741824
- **Unit:** Byte
- **Description:** The maximum result size of a single compaction for Primary Key tables.

#### update_compaction_delvec_file_io_amp_ratio

- **Default:** 2
- **Description:** Used to control the priority of compaction for rowsets that contain Delvec files in Primary Key tables. The larger the value, the higher the priority.

#### repair_compaction_interval_seconds

- **Default:** 600
- **Unit:** Second
- **Description:** The time interval to poll Repair Compaction threads.

#### update_compaction_result_bytes

- **Default:** 1073741824
- **Unit:** Byte
- **Description:** The maximum result size of a single compaction for Primary Key tables.

#### update_compaction_delvec_file_io_amp_ratio

- **Default:** 2
- **Description:** Used to control the priority of compaction for rowsets that contain Delvec files in Primary Key tables. The larger the value, the higher the priority.

#### repair_compaction_interval_seconds

- **Default:** 600
- **Unit:** Second
- **Description:** The time interval to poll Repair Compaction threads.

#### min_compaction_failure_interval_sec

- **Default:** 120
- **Unit:** Second
- **Description:** The minimum time interval at which a tablet compaction can be scheduled since the previous compaction failure.

#### max_compaction_concurrency

- **Default:** -1
- **Description:** The maximum concurrency of compactions (including both Base Compaction and Cumulative Compaction). The value `-1` indicates that no limit is imposed on the concurrency.

#### periodic_counter_update_period_ms

- **Default:** 500 ms
- **Description:** The time interval at which to collect the Counter statistics.

#### pindex_major_compaction_limit_per_disk

- **Default:** 1
- **Description:** The maximum concurrency of compaction on a disk. This addresses the issue of uneven I/O across disks due to compaction. This issue can cause excessively high I/O for certain disks.
- **Introduced in:** 3.0.9

#### load_error_log_reserve_hours

- **Default:** 48
- **Unit:** Hour
- **Description:** The time for which data loading logs are reserved.

#### streaming_load_max_mb

- **Default:** 102400
- **Unit:** MB
- **Description:** The maximum size of a file that can be streamed into StarRocks.

#### streaming_load_max_batch_size_mb

- **Default:** 100
- **Unit:** MB
- **Description:** The maximum size of a JSON file that can be streamed into StarRocks.

#### memory_maintenance_sleep_time_s

- **Default:** 10
- **Unit:** Second
- **Description:** The time interval at which ColumnPool GC is triggered. StarRocks executes GC periodically and returns the released memory to the operating system.

#### write_buffer_size

- **Default:** 104857600
- **Unit:** Byte
- **Description:** The buffer size of MemTable in the memory. This configuration item is the threshold to trigger a flush.

#### tablet_stat_cache_update_interval_second

- **Default:** 300 seconds
- **Description:** The time interval at which to update Tablet Stat Cache.

#### result_buffer_cancelled_interval_time

- **Default:** 300 seconds
- **Description:** The wait time before BufferControlBlock releases data.

#### thrift_rpc_timeout_ms

- **Default:** 5,000 ms
- **Description:** The timeout for a thrift RPC.

#### txn_commit_rpc_timeout_ms

- **Default:** 60,000
- **Description:** The timeout for a transaction commit RPC. Since v3.1.0, this parameter is deprecated.
- **Unit:** ms

#### max_consumer_num_per_group

- **Default:** 3 (Maximum Number of Consumers in a Consumer Group of Routine Load)
- **Description:** The maximum number of consumers in a consumer group of Routine Load.

#### max_memory_sink_batch_count

- **Default:** 20 (Maximum Number of Scan Cache Batches)
- **Description:** The maximum number of Scan Cache batches.

#### scan_context_gc_interval_min

- **Default:** 5 minutes
- **Description:** The time interval at which to clean the Scan Context.

#### path_gc_check_step

- **Default:** 1,000 (Maximum Number of Files Scanned Continuously Each Time)
- **Description:** The maximum number of files that can be scanned continuously each time.

#### path_gc_check_step_interval_ms

- **Default:** 10 ms
- **Description:** The time interval between file scans.

#### path_scan_interval_second

- **Default:** 86,400 seconds
- **Description:** The time interval at which GC cleans expired data.

#### storage_flood_stage_usage_percent

- **Default:** 95
- **Description:** Hard limit of the storage usage percentage in all BE directories. If the storage usage (in percentage) of the BE storage directory exceeds this value and the remaining storage space is less than `storage_flood_stage_left_capacity_bytes`, Load and Restore jobs are rejected. You need to set this item together with the FE configuration item `storage_usage_hard_limit_percent` to allow the configurations to take effect.

#### storage_flood_stage_left_capacity_bytes

- **Default:** 107,374,182,400 Bytes (Remaining Storage Space Threshold for Load and Restore Job Rejection)
- **Description:** Hard limit of the remaining storage space in all BE directories. If the remaining storage space of the BE storage directory is less than this value and the storage usage (in percentage) exceeds `storage_flood_stage_usage_percent`, Load and Restore jobs are rejected. You need to set this item together with the FE configuration item `storage_usage_hard_limit_reserve_bytes` to allow the configurations to take effect.

#### tablet_meta_checkpoint_min_new_rowsets_num

- **Default:** 10 (Minimum Number of Rowsets to Trigger a TabletMeta Checkpoint)
- **Description:** The minimum number of rowsets to create since the last TabletMeta Checkpoint.

#### tablet_meta_checkpoint_min_interval_secs

- **Default:** 600 seconds
- **Description:** The time interval of thread polling for a TabletMeta Checkpoint.

#### max_runnings_transactions_per_txn_map

- **Default:** 100 (Maximum Number of Concurrent Transactions per Partition)
- **Description:** The maximum number of transactions that can run concurrently in each partition.

#### tablet_max_pending_versions

- **Default:** 1,000 (Maximum Number of Pending Versions on a Primary Key Tablet)
- **Description:** The maximum number of pending versions that are tolerable on a Primary Key tablet. Pending versions refer to versions that are committed but not applied yet.

#### tablet_max_versions

- **Default:** 1,000 (Maximum Number of Versions Allowed on a Tablet)
- **Description:** The maximum number of versions allowed on a tablet. If the number of versions exceeds this value, new write requests will fail.

#### max_hdfs_file_handle

- **Default:** 1,000 (Maximum Number of HDFS File Descriptors)
- **Description:** The maximum number of HDFS file descriptors that can be opened.

#### be_exit_after_disk_write_hang_second

- **Default:** 60 seconds
- **Description:** The length of time that the BE waits to exit after the disk hangs.

#### min_cumulative_compaction_failure_interval_sec

- **Default:** 30 seconds
- **Description:** The minimum time interval at which Cumulative Compaction retries upon failures.

#### size_tiered_level_num

- **Default:** 7 (Number of Levels for Size-tiered Compaction)
- **Description:** The number of levels for the Size-tiered Compaction strategy. At most one rowset is reserved for each level. Therefore, under a stable condition, there are, at most, as many rowsets as the level number specified in this configuration item.

#### size_tiered_level_multiple

- **Default:** 5 (Multiple of Data Size Between Contiguous Levels in Size-tiered Compaction)
- **Description:** The multiple of data size between two contiguous levels in the Size-tiered Compaction strategy.

#### size_tiered_level_multiple_dupkey

- **Default:** 10
- **Description:** In the Size-tiered Compaction policy, the multiple of the data amount difference between two adjacent levels for Duplicate Key tables.

#### size_tiered_min_level_size

- **Default:** 131,072 Bytes
- **Description:** The data size of the minimum level in the Size-tiered Compaction strategy. Rowsets smaller than this value immediately trigger the data compaction.

#### storage_page_cache_limit

- **Default:** 20%
- **Description:** The PageCache size. It can be specified as size, for example, `20G`, `20,480M`, `20,971,520K`, or `21,474,836,480B`. It can also be specified as the ratio (percentage) to the memory size, for example, `20%`. It takes effect only when `disable_storage_page_cache` is set to `false`.

#### compaction_memory_limit_per_worker

- **Default**: 2147483648 (2 GB)
- **Unit**: Byte
- **Description**: The maximum memory usage per compaction thread.

#### lake_enable_vertical_compaction_fill_data_cache

- **Default:** false
- **Description:** Whether to allow compaction tasks to cache data on local disks in a shared-data cluster.
- **Introduced in:** v3.1.7, v3.2.3

#### lake_pk_compaction_max_input_rowsets

- **Default:** 1000
- **Description:** The maximum number of input rowsets allowed in a Primary Key table compaction task in a shared-data cluster. Since v3.2.4 and v3.1.10, the default value of this parameter is changed from `5` to `1000`. After the Sized-tiered Compaction policy is enabled for Primary Key tables (by setting `enable_pk_size_tiered_compaction_strategy` to `true`), StarRocks does not need to limit the number of rowsets for each compaction to reduce write amplification. Therefore, the default value of this parameter is increased.
- **Introduced in:** v3.1.8, v3.2.3

#### compact_threads

- **Default:** 4
- **Description:** The maximum number of threads used for concurrent compaction tasks. This configuration is changed to dynamic from v3.1.7 and v3.2.2 onwards.
- **Introduced in:** v3.0.0

#### update_compaction_ratio_threshold

- **Default:** 0.5
- **Description:** The maximum proportion of data that a compaction can merge for a Primary Key table in a shared-data cluster. We recommend shrinking this value if a single tablet becomes excessively large. This parameter is supported from v3.1.5 onwards.

#### create_tablet_worker_count

- **Default**: 3
- **Unit**: N/A
- **Description**: The number of threads used to create a tablet. This configuration is changed to dynamic from v3.1.7 onwards.

#### number_tablet_writer_threads

- **Default**: 16
- **Unit**: N/A
- **Description**: The number of threads used for Stream Load. This configuration is changed to dynamic from v3.1.7 onwards.

#### pipeline_connector_scan_thread_num_per_cpu

- **Default**: 8
- **Unit**: N/A
- **Description**: The number of scan threads assigned to Pipeline Connector per CPU core in the BE node. This configuration is changed to dynamic from v3.1.7 onwards.

#### starlet_use_star_cache

- **Default**: true
- **Description**: Whether to enable block data cache in a shared-data cluster. `true` indicates enabling this feature and `false` indicates disabling it.
- **Introduced in:** v3.1

#### starlet_cache_evict_high_water

- **Default**: 0.2
- **Description**: In a shared-data cluster, if the percentage of the available disk capacity is below this value, file data cache eviction will be triggered. The default value indicates that file data cache will use at most 80% of the disk capacity.
- **Introduced in:** v3.0

### Configure BE static parameters

You can only set the static parameters of a BE by changing them in the corresponding configuration file **be.conf**, and restart the BE to allow the changes to take effect.

BE static parameters are as follows.

#### hdfs_client_enable_hedged_read

- **Default**: false
- **Unit**: N/A
- **Description**: Specifies whether to enable the hedged read feature. This parameter is supported from v3.0 onwards.

#### hdfs_client_hedged_read_threadpool_size

- **Default**: 128
- **Unit**: N/A
- **Description**: Specifies the size of the Hedged Read thread pool on your HDFS client. The thread pool size limits the number of threads to dedicate to the running of hedged reads in your HDFS client. This parameter is supported from v3.0 onwards. It is equivalent to the dfs.client.hedged.read.threadpool.size parameter in the hdfs-site.xml file of your HDFS cluster.

#### hdfs_client_hedged_read_threshold_millis

- **Default**: 2500
- **Unit**: Millisecond
- **Description**: Specifies the number of milliseconds to wait before starting up a hedged read. For example, you have set this parameter to 30. In this situation, if a read from a block has not returned within 30 milliseconds, your HDFS client immediately starts up a new read against a different block replica. This parameter is supported from v3.0 onwards. It is equivalent to the dfs.client.hedged.read.threshold.millis parameter in the hdfs-site.xml file of your HDFS cluster.

#### be_port

- **Default**: 9060
- **Unit**: N/A
- **Description**: The BE thrift server port, which is used to receive requests from FEs.

#### brpc_port

- **Default**: 8060
- **Unit**: N/A
- **Description**: The BE bRPC port, which is used to view the network statistics of bRPCs.

#### brpc_num_threads

- **Default**: -1
- **Unit**: N/A
- **Description**: The number of bthreads of a bRPC. The value -1 indicates the same number with the CPU threads.

#### priority_networks

- **Default**: Empty string
- **Unit**: N/A
- **Description**: The CIDR-formatted IP address that is used to specify the priority IP address of a BE node if the machine that hosts the BE node has multiple IP addresses.

#### heartbeat_service_port

- **Default**: 9050
- **Unit**: N/A
- **Description**: The BE heartbeat service port, which is used to receive heartbeats from FEs.

#### starlet_port

- **Default**: 9070
- **Unit**: N/A
- **Description**: An extra agent service port for CN (BE in v3.0) in a shared-data cluster.

#### heartbeat_service_thread_count

- **Default**: 1
- **Unit**: N/A
- **Description**: The thread count of the BE heartbeat service.

#### drop_tablet_worker_count

- **Default**: 3
- **Unit**: N/A
- **Description**: The number of threads used to drop a tablet.

#### push_worker_count_normal_priority

- **Default**: 3
- **Unit**: N/A
- **Description**: The number of threads used to handle a load task with NORMAL priority.

#### push_worker_count_high_priority

- **Default**: 3
- **Unit**: N/A
- **Description**: The number of threads used to handle a load task with HIGH priority.

#### transaction_publish_version_worker_count

- **Default**: 0
- **Unit**: N/A
- **Description**: The maximum number of threads used to publish a version. When this value is set to less than or equal to 0, the system uses half of the CPU core count as the value, so as to avoid insufficient thread resources when import concurrency is high but only a fixed number of threads are used. From v2.5, the default value has been changed from 8 to 0.

#### clear_transaction_task_worker_count

- **Default**: 1
- **Unit**: N/A
- **Description**: The number of threads used for clearing transaction.

#### alter_tablet_worker_count

- **Default**: 3
- **Unit**: N/A
- **Description**: The number of threads used for schema change.

#### clone_worker_count

- **Default**: 3
- **Unit**: N/A
- **Description**: The number of threads used for clone.

#### storage_medium_migrate_count

- **Default**: 1
- **Unit**: N/A
- **Description**: The number of threads used for storage medium migration (from SATA to SSD).

#### check_consistency_worker_count

- **Default**: 1
- **Unit**: N/A
- **Description**: The number of threads used for checking the consistency of tablets.

#### object_storage_connect_timeout_ms

- **Default**: `-1`, which means to use the default timeout duration of the SDK configurations.
- **Unit**: ms
- **Description**: Timeout duration to establish socket connections with object storage.
- **Introduced in:** 3.0.9

#### object_storage_request_timeout_ms

- **Default**: `-1`, which means to use the default timeout duration of the SDK configurations.
- **Unit**: ms
- **Description**: Timeout duration to establish HTTP connections with object storage.
- **Introduced in:** 3.0.9

#### sys_log_dir

- **Default**: `${STARROCKS_HOME}/log`
- **Unit**: N/A
- **Description**: The directory that stores system logs (including INFO, WARNING, ERROR, and FATAL).

#### user_function_dir

- **Default**: `${STARROCKS_HOME}/lib/udf`
- **Unit**: N/A
- **Description**: The directory used to store User-defined Functions (UDFs).

#### small_file_dir

- **Default**: `${STARROCKS_HOME}/lib/small_file`
- **Unit**: N/A
- **Description**: The directory used to store the files downloaded by the file manager.

#### sys_log_level

- **Default**: INFO
- **Unit**: N/A
- **Description**: The severity levels into which system log entries are classified. Valid values: INFO, WARN, ERROR, and FATAL.

#### sys_log_roll_mode

- **Default**: SIZE-MB-1024
- **Unit**: N/A
- **Description**: The mode in which system logs are segmented into log rolls. Valid values include `TIME-DAY`, `TIME-HOUR`, and `SIZE-MB-`size. The default value indicates that logs are segmented into rolls, each of which is 1 GB.

#### sys_log_roll_num

- **Default**: 10
- **Unit**: N/A
- **Description**: The number of log rolls to reserve.

#### sys_log_verbose_modules

- **Default**: Empty string
- **Unit**: N/A
- **Description**: The module of the logs to be printed. For example, if you set this configuration item to OLAP, StarRocks only prints the logs of the OLAP module. Valid values are namespaces in BE, including starrocks, starrocks::debug, starrocks::fs, starrocks::io, starrocks::lake, starrocks::pipeline, starrocks::query_cache, starrocks::stream, and starrocks::workgroup.

#### sys_log_verbose_level

- **Default**: 10
- **Unit**: N/A
- **Description**: The level of the logs to be printed. This configuration item is used to control the output of logs initiated with VLOG in codes.

#### log_buffer_level

- **Default**: Empty string
- **Unit**: N/A
- **Description**: The strategy for flushing logs. The default value indicates that logs are buffered in memory. Valid values are -1 and 0. -1 indicates that logs are not buffered in memory.

#### num_threads_per_core

- **Default**: 3
- **Unit**: N/A
- **Description**: The number of threads started on each CPU core.

#### compress_rowbatches

- **Default**: TRUE
- **Unit**: N/A
- **Description**: A boolean value to control whether to compress the row batches in RPCs between BEs. TRUE indicates compressing the row batches, and FALSE indicates not compressing them.

#### serialize_batch

- **Default**: FALSE
- **Unit**: N/A
- **Description**: A boolean value to control whether to serialize the row batches in RPCs between BEs. TRUE indicates serializing the row batches, and FALSE indicates not serializing them.

#### storage_root_path

- **Default**: `${STARROCKS_HOME}/storage`
- **Unit**: N/A
- **Description**: The directory and medium of the storage volume.
  - Multiple volumes are separated by semicolons (`;`).
  - If the storage medium is SSD, add `medium:ssd` at the end of the directory.
  - If the storage medium is HDD, add ,medium:hdd at the end of the directory.

#### max_length_for_bitmap_function

- **Default**: 1000000
- **Unit**: Byte
- **Description**: The maximum length of input values for bitmap functions.

#### max_length_for_to_base64

- **Default**: 200000
- **Unit**: Byte
- **Description**: The maximum length of input values for the to_base64() function.

#### max_tablet_num_per_shard

- **Default**: 1024
- **Unit**: N/A
- **Description**: The maximum number of tablets in each shard. This configuration item is used to restrict the number of tablet child directories under each storage directory.

#### max_garbage_sweep_interval

- **Default**: 3600
- **Unit**: Second
- **Description**: The maximum time interval for garbage collection on storage volumes.

#### min_garbage_sweep_interval

- **Default**: 180
- **Unit**: Second
- **Description**: The minimum time interval for garbage collection on storage volumes.

#### file_descriptor_cache_capacity

- **Default**: 16384
- **Unit**: N/A
- **Description**: The number of file descriptors that can be cached.

#### min_file_descriptor_number

- **Default**: 60000
- **Unit**: N/A
- **Description**: The minimum number of file descriptors in the BE process.

#### index_stream_cache_capacity

- **Default**: 10737418240
- **Unit**: Byte
- **Description**: The cache capacity for the statistical information of BloomFilter, Min, and Max.

#### disable_storage_page_cache

- **Default**: FALSE
- **Unit**: N/A
- **Description**: A boolean value to control whether to disable PageCache.
  - When PageCache is enabled, StarRocks caches the recently scanned data.
  - PageCache can significantly improve the query performance when similar queries are repeated frequently.
  - TRUE indicates disabling PageCache.
  - The default value of this item has been changed from TRUE to FALSE since StarRocks v2.4.

#### base_compaction_num_threads_per_disk

- **Default**: 1
- **Unit**: N/A
- **Description**: The number of threads used for Base Compaction on each storage volume.

#### base_cumulative_delta_ratio

- **Default**: 0.3
- **Unit**: N/A
- **Description**: The ratio of cumulative file size to base file size. The ratio reaching this value is one of the conditions that trigger the Base Compaction.

#### compaction_trace_threshold

- **Default**: 60
- **Unit**: Second
- **Description**: The time threshold for each compaction. If a compaction takes more time than the time threshold, StarRocks prints the corresponding trace.

#### cumulative_compaction_num_threads_per_disk

- **Default**: 1
- **Description**: The number of Cumulative Compaction threads per disk.

#### vertical_compaction_max_columns_per_group

- **Default**: 5
- **Description**: The maximum number of columns per group of Vertical Compactions.

#### enable_check_string_lengths

- **Default**: true
- **Description**: Whether to check the data length during loading to solve compaction failures caused by out-of-bound VARCHAR data.

#### max_row_source_mask_memory_bytes

- **Default**: 209715200
- **Unit**: Byte
- **Description**: The maximum memory size of the row source mask buffer. When the buffer is larger than this value, data will be persisted to a temporary file on the disk. This value should be less than the `compaction_mem_limit` parameter.

#### be_http_port

- **Default**: 8040
- **Unit**: N/A
- **Description**: The HTTP server port.

#### be_http_num_workers

- **Default**: 48
- **Unit**: N/A
- **Description**: The number of threads used by the HTTP server.

#### load_data_reserve_hours

- **Default**: 4
- **Unit**: Hour
- **Description**: The reservation time for the files produced by small-scale loadings.

#### streaming_load_rpc_max_alive_time_sec

- **Default**: 1200
- **Unit**: Second
- **Description**: The RPC timeout for Stream Load.

#### fragment_pool_thread_num_min

- **Default**: 64
- **Unit**: N/A
- **Description**: The minimum number of threads used for query.

#### fragment_pool_thread_num_max

- **Default**: 4096
- **Unit**: N/A
- **Description**: The maximum number of threads used for query.

#### fragment_pool_queue_size

- **Default**: 2048
- **Unit**: N/A
- **Description**: The upper limit of the query number that can be processed on each BE node.

#### enable_token_check

- **Default**: TRUE
- **Unit**: N/A
- **Description**: A boolean value to control whether to enable the token check. TRUE indicates enabling the token check, and FALSE indicates disabling it.

#### enable_prefetch

- **Default**: TRUE
- **Unit**: N/A
- **Description**: A boolean value to control whether to enable the pre-fetch of the query. TRUE indicates enabling pre-fetch, and FALSE indicates disabling it.

#### load_process_max_memory_limit_bytes

- **Default**: 107374182400
- **Unit**: Byte
- **Description**: The maximum size limit of memory resources that can be taken up by all load processes on a BE node.

#### load_process_max_memory_limit_percent

- **Default**: 30
- **Unit**: %
- **Description**: The maximum percentage limit of memory resources that can be taken up by all load processes on a BE node.

#### sync_tablet_meta

- **Default**: FALSE
- **Unit**: N/A
- **Description**: A boolean value to control whether to enable the synchronization of the tablet metadata. TRUE indicates enabling synchronization, and FALSE indicates disabling it.

#### routine_load_thread_pool_size

- **Default**: 10
- **Unit**: N/A
- **Description**: The thread pool size for Routine Load on each BE. Since v3.1.0, this parameter is deprecated. The thread pool size for Routine Load on each BE is now controlled by the FE dynamic parameter `max_routine_load_task_num_per_be`.

#### internal_service_async_thread_num

- **Default:** 10 (Number of Threads)
- **Description:** The thread pool size allowed on each BE for interacting with Kafka. Currently, the FE responsible for processing Routine Load requests depends on BEs to interact with Kafka, and each BE in StarRocks has its own thread pool for interactions with Kafka. If a large number of Routine Load tasks are distributed to a BE, the BE's thread pool for interactions with Kafka may be too busy to process all tasks in a timely manner. In this situation, you can adjust the value of this parameter to suit your needs. **Since v3.1.0, this parameter is deprecated. The thread pool size for Routine Load on each BE is now controlled by the FE dynamic parameter `max_routine_load_task_num_per_be`.**

#### brpc_max_body_size

- **Default**: 2147483648
- **Unit**: Byte
- **Description**: The maximum body size of a bRPC.

#### tablet_map_shard_size

- **Default**: 32
- **Unit**: N/A
- **Description**: The tablet map shard size. The value must be a power of two.

#### enable_bitmap_union_disk_format_with_set

- **Default**: FALSE
- **Unit**: N/A
- **Description**: A boolean value to control whether to enable the new storage format of the BITMAP type, which can improve the performance of bitmap_union. TRUE indicates enabling the new storage format, and FALSE indicates disabling it.

#### mem_limit

- **Default**: 90%
- **Unit**: N/A
- **Description**: BE process memory upper limit. You can set it as a percentage ("80%") or a physical limit ("100G"). The default hard limit is 90% of the server's memory size, and the soft limit is 80%. You need to configure this parameter if you want to deploy StarRocks with other memory-intensive services on a same server.

#### flush_thread_num_per_store

- **Default**: 2
- **Unit**: N/A
- **Description**: Number of threads that are used for flushing MemTable in each store.

#### block_cache_enable

- **Default**: false
- **Unit**: N/A
- **Description**: Whether to enable Data Cache. TRUE indicates Data Cache is enabled, and FALSE indicates Data Cache is disabled.

#### block_cache_disk_path

- **Default**: N/A
- **Unit**: N/A
- **Description**: The paths of disks. We recommend that the number of paths you configure for this parameter is the same as the number of disks on your BE machine. Multiple paths need to be separated with semicolons (;). After you add this parameter, StarRocks automatically creates a file named cachelib_data to cache blocks.

#### block_cache_meta_path

- **Default**: N/A
- **Unit**: N/A
- **Description**: The storage path of block metadata. You can customize the storage path. We recommend that you store the metadata under the $STARROCKS_HOME path.

#### block_cache_mem_size

- **Default**: 2147483648
- **Unit**: Byte
- **Description**: The maximum amount of data that can be cached in memory. Unit: bytes. The default value is 2147483648, which is 2 GB. We recommend that you set the value of this parameter to at least 20 GB. If StarRocks reads a large amount of data from disks after Data Cache is enabled, consider increasing the value.

#### block_cache_disk_size

- **Default**: 0
- **Unit**: Byte
- **Description**: The maximum amount of data that can be cached on a single disk. For example, if you configure two disk paths for the block_cache_disk_path parameter and set the value of the block_cache_disk_size parameter as 21474836480 (20 GB), a maximum of 40 GB data can be cached on these two disks. The default value is 0, which indicates that only memory is used to cache data. Unit: bytes.

#### jdbc_connection_pool_size

- **Default**: 8
- **Unit**: N/A
- **Description**: The JDBC connection pool size. On each BE node, queries that access the external table with the same jdbc_url share the same connection pool.

#### jdbc_minimum_idle_connections

- **Default**: 1
- **Unit**: N/A
- **Description**: The minimum number of idle connections in the JDBC connection pool.

#### jdbc_connection_idle_timeout_ms

- **Default**: 600000
- **Unit**: N/A
- **Description**: The length of time after which an idle connection in the JDBC connection pool expires. If the connection idle time in the JDBC connection pool exceeds this value, the connection pool closes idle connections beyond the number specified in the configuration item jdbc_minimum_idle_connections.

#### query_cache_capacity

- **Default**: 536870912
- **Unit**: N/A
- **Description**: The size of the query cache in the BE. Unit: bytes. The default size is 512 MB. The size cannot be less than 4 MB. If the memory capacity of the BE is insufficient to provision your expected query cache size, you can increase the memory capacity of the BE.

#### enable_event_based_compaction_framework

- **Default**: TRUE
- **Unit**: N/A
- **Description**: Whether to enable the Event-based Compaction Framework. TRUE indicates Event-based Compaction Framework is enabled, and FALSE indicates it is disabled. Enabling Event-based Compaction Framework can greatly reduce the overhead of compaction in scenarios where there are many tablets or a single tablet has a large amount of data.

#### enable_size_tiered_compaction_strategy

- **Default**: TRUE
- **Unit**: N/A
- **Description**: Whether to enable the Size-tiered Compaction strategy (excluding Primary Key tables). TRUE indicates the Size-tiered Compaction strategy is enabled, and FALSE indicates it is disabled.

#### enable_pk_size_tiered_compaction_strategy

- **Default:** true
- **Description:** Whether to enable the Size-tiered Compaction policy for Primary Key tables.
- **Introduced in**: v3.2.4, v3.1.10

#### lake_service_max_concurrency

- **Default**: 0
- **Unit**: N/A
- **Description**: The maximum concurrency of RPC requests in a shared-data cluster. Incoming requests will be rejected when this threshold is reached. When this item is set to 0, no limit is imposed on the concurrency.

#### starlet_star_cache_disk_size_percent

- **Default**: 80
- **Description**: The percentage of disk capacity that block data cache can use at most in a shared-data cluster.
- **Introduced in:** v3.1

<!--| aws_sdk_logging_trace_enabled | 0 | N/A | |
| be_exit_after_disk_write_hang_second | 60 | N/A | |
| be_service_threads | 64 | N/A | |
| bitmap_filter_enable_not_equal | 0 | N/A | |
| bitmap_max_filter_items | 30 | N/A | |
| bitmap_max_filter_ratio | 1000 | N/A | |
| bitmap_serialize_version | 1 | N/A | |
| block_cache_block_size | 1048576 | N/A | |
| block_cache_disk_path |  | N/A | |
| block_cache_disk_size | 21474836480 | N/A | |
| block_cache_enable | 0 | N/A | |
| block_cache_mem_size | 2147483648 | N/A | |
| broker_write_timeout_seconds | 30 | N/A | |
| brpc_socket_max_unwritten_bytes | 1073741824 | N/A | |
| cardinality_of_inject | 10 | N/A | |
| chunk_reserved_bytes_limit | 2147483648 | N/A | |
| cluster_id | -1 | N/A | |
| column_dictionary_key_size_threshold | 0 | N/A | |
| compact_thread_pool_queue_size | 100 | N/A | |
| compact_threads | 4 | N/A | |
| compaction_max_memory_limit | -1 | N/A | |
| compaction_max_memory_limit_percent | 100 | N/A | |
| compaction_memory_limit_per_worker | 2147483648 | N/A | |
| consistency_max_memory_limit | 10G | N/A | |
| consistency_max_memory_limit_percent | 20 | N/A | |
| cumulative_compaction_num_threads_per_disk | 1 | N/A | |
| default_query_options | | N/A | |
| delete_worker_count_high_priority | 1 | N/A | |
| delete_worker_count_normal_priority | 2 | N/A | |
| deliver_broadcast_rf_passthrough_bytes_limit | 131072 | N/A | |
| deliver_broadcast_rf_passthrough_inflight_num | 10 | N/A | |
| dependency_librdkafka_debug | all | N/A | |
| dependency_librdkafka_debug_enable | 0 | N/A | |
| dictionary_encoding_ratio | 0.7 | N/A | |
| dictionary_speculate_min_chunk_size | 10000 | N/A | |
| directory_of_inject |  | N/A | |
| disable_column_pool | 0 | N/A | |
| disable_mem_pools | 0 | N/A | |
| download_worker_count | 1 | N/A | |
| enable_check_string_lengths | 1 | N/A | |
| enable_event_based_compaction_framework | 0 | N/A | |
| enable_load_colocate_mv | 0 | N/A | |
| enable_metric_calculator | 0 | N/A | |
| enable_new_load_on_memory_limit_exceeded | 1 | N/A | |
| enable_orc_late_materialization | 1 | N/A | |
| enable_schema_change_v2 | 1 | N/A | |
| enable_segment_overflow_read_chunk | 1 | N/A | |
| enable_system_metrics | 1 | N/A | |
| es_http_timeout_ms | 5000 | N/A | |
| es_index_max_result_window | 10000 | N/A | |
| es_scroll_keepalive | 5m | N/A | |
| experimental_s3_max_single_part_size | 16777216 | N/A | |
| experimental_s3_min_upload_part_size | 16777216 | N/A | |
| ignore_broken_disk | 0 | N/A | |
| ignore_load_tablet_failure | 0 | N/A | |
| ignore_rowset_stale_unconsistent_delete | 0 | N/A | |
| internal_service_async_thread_num | 10 | N/A | |
| io_coalesce_read_max_buffer_size | 8388608 | N/A | |
| io_coalesce_read_max_distance_size | 1048576 | N/A | |
| jaeger_endpoint | | N/A | |
| jdbc_connection_pool_size | 8 | N/A | |
| l0_l1_merge_ratio | 10 | N/A | |
| lake_gc_metadata_check_interval | 1800 | N/A | |
| lake_gc_metadata_max_versions | 10 | N/A | |
| lake_gc_segment_check_interval | 3600 | N/A | |
| lake_gc_segment_expire_seconds | 259200 | N/A | |
| lake_metadata_cache_limit | 2147483648 | N/A | |
| late_materialization_ratio | 10 | N/A | |
| local_library_dir |  | N/A | |
| loop_count_wait_fragments_finish | 0 | N/A | |
| madvise_huge_pages | 0 | N/A | |
| make_snapshot_rpc_timeout_ms | 20000 | N/A | |
| make_snapshot_worker_count | 5 | N/A | |
| manual_compact_before_data_dir_load | 0 | N/A | |
| max_batch_publish_latency_ms | 100 | N/A | |
| max_client_cache_size_per_host | 10 | N/A | |
| max_cumulative_compaction_num_singleton_deltas | 1000 | N/A | |
| max_free_io_buffers | 128 | N/A | |
| max_hdfs_scanner_num | 50 | N/A | |
| max_length_for_bitmap_function | 1000000 | N/A | |
| max_length_for_to_base64 | 200000 | N/A | |
| max_load_dop | 16 | N/A | |
| max_pulsar_consumer_num_per_group | 10 | N/A | |
| max_row_source_mask_memory_bytes | 209715200 | N/A | |
| max_segment_file_size | 1073741824 | N/A | |
| max_transmit_batched_bytes | 262144 | N/A | |
| memory_max_alignment | 16 | N/A | |
| memory_ratio_for_sorting_schema_change | 0.8 | N/A | |
| meta_threshold_to_manual_compact | 10737418240 | N/A | |
| metric_late_materialization_ratio | 1000 | N/A | |
| min_base_compaction_size | 21474836480 | N/A | |
| min_cumulative_compaction_failure_interval_sec | 30 | N/A | |
| min_cumulative_compaction_num_singleton_deltas | 5 | N/A | |
| min_cumulative_compaction_size | 5368709120 | N/A | |
| mmap_buffers | 0 | N/A | |
| null_encoding | 0 | N/A | |
| num_cores | 0 | N/A | |
| object_storage_access_key_id |  | N/A | |
| object_storage_endpoint |  | N/A | |
| object_storage_endpoint_path_style_access | 0 | N/A | |
| object_storage_endpoint_use_https | 0 | N/A | |
| object_storage_max_connection | 102400 | N/A | |
| object_storage_region | | N/A | |
| object_storage_secret_access_key |  | N/A | |
| orc_coalesce_read_enable | 1 | N/A | |
| orc_file_cache_max_size | 8388608 | N/A | |
| orc_natural_read_size | 8388608 | N/A | |
| parallel_clone_task_per_path | 2 | N/A | |
| parquet_coalesce_read_enable | 1 | N/A | |
| parquet_header_max_size | 16384 | N/A | |
| parquet_late_materialization_enable | 1 | N/A | |
| path_gc_check | 1 | N/A | |
| path_gc_check_interval_second | 86400 | N/A | |
| pipeline_exec_thread_pool_thread_num | 0 | N/A | |
| pipeline_connector_scan_thread_num_per_cpu | 8 | N/A | The thread number per cpu of connector scanner  |
| pipeline_max_num_drivers_per_exec_thread | 10240 | N/A | |
| pipeline_prepare_thread_pool_queue_size | 102400 | N/A | |
| pipeline_prepare_thread_pool_thread_num | 0 | N/A | |
| pipeline_print_profile | 0 | N/A | |
| pipeline_scan_thread_pool_queue_size | 102400 | N/A | |
| pipeline_scan_thread_pool_thread_num | 0 | N/A | |
| pipeline_sink_brpc_dop | 64 | N/A | |
| pipeline_sink_buffer_size | 64 | N/A | |
| pipeline_sink_io_thread_pool_queue_size | 102400 | N/A | |
| pipeline_sink_io_thread_pool_thread_num | 0 | N/A | |
| plugin_path |  | N/A | |
| port | 20001 | N/A | |
| pprof_profile_dir |  | N/A | |
| pre_aggregate_factor | 80 | N/A | |
| priority_queue_remaining_tasks_increased_frequency | 512 | N/A | |
| profile_report_interval | 30 | N/A | |
| pull_load_task_dir |  | N/A | |
| query_cache_capacity | 536870912 | N/A | |
| query_debug_trace_dir |  | N/A | |
| query_scratch_dirs |  | N/A | |
| release_snapshot_worker_count | 5 | N/A | |
| repair_compaction_interval_seconds | 600 | N/A | |
| rewrite_partial_segment | 1 | N/A | |
| routine_load_kafka_timeout_second | 10 | N/A | |
| routine_load_pulsar_timeout_second | 10 | N/A | |
| rpc_compress_ratio_threshold | 1.1 | N/A | |
| scan_use_query_mem_ratio | 0.25 | N/A | |
| scratch_dirs | /tmp | N/A | |
| send_rpc_runtime_filter_timeout_ms | 1000 | N/A | |
| sleep_five_seconds | 5 | N/A | |
| sleep_one_second | 1 | N/A | |
| sorter_block_size | 8388608 | N/A | |
| storage_format_version | 2 | N/A | |
| sys_minidump_dir |  | N/A | |
| sys_minidump_enable | 0 | N/A | |
| sys_minidump_interval | 600 | N/A | |
| sys_minidump_limit | 20480 | N/A | |
| sys_minidump_max_files | 16 | N/A | |
| tablet_internal_parallel_max_splitted_scan_bytes | 536870912 | N/A | |
| tablet_internal_parallel_max_splitted_scan_rows | 1048576 | N/A | |
| tablet_internal_parallel_min_scan_dop | 4 | N/A | |
| tablet_internal_parallel_min_splitted_scan_rows | 16384 | N/A | |
| tablet_max_versions | 15000 | N/A | |
| tablet_writer_open_rpc_timeout_sec | 60 | N/A | |
| tc_max_total_thread_cache_bytes | 1073741824 | N/A | |
| thrift_connect_timeout_seconds | 3 | N/A | |
| txn_map_shard_size | 128 | N/A | |
| txn_shard_size | 1024 | N/A | |
| udf_thread_pool_size | 1 | N/A | |
| update_compaction_num_threads_per_disk | 1 | N/A | |
| update_compaction_per_tablet_min_interval_seconds | 120 | N/A | |
| update_memory_limit_percent | 60 | N/A | |
| upload_worker_count | 1 | N/A | |
| use_mmap_allocate_chunk | 0 | N/A | |
| vector_chunk_size | 4096 | N/A | |
| vertical_compaction_max_columns_per_group | 5 | N/A | |
| web_log_bytes | 1048576 | N/A | |-->