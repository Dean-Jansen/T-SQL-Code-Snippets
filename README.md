# T-SQL Code Snippets
## My collection of useful SQL Queries that i use ofthen

----------------------

### Find All tables Containing Column With Specified Name

```SQL
select * from INFORMATION_SCHEMA.COLUMNS 
where COLUMN_NAME like '%color%' 
order by TABLE_NAME
```

### Returns cached queries, Including CPU Usage, Execution Time, Query Plan Etc

```SQL
SELECT total_worker_time/execution_count AS AvgCPU  
, total_worker_time AS TotalCPU
, total_elapsed_time/execution_count AS AvgDuration  
, total_elapsed_time AS TotalDuration  
, (total_logical_reads+total_physical_reads)/execution_count AS AvgReads 
, (total_logical_reads+total_physical_reads) AS TotalReads
, execution_count   
, SUBSTRING(st.TEXT, (qs.statement_start_offset/2)+1  
, ((CASE qs.statement_end_offset  WHEN -1 THEN datalength(st.TEXT)  
ELSE qs.statement_end_offset  
END - qs.statement_start_offset)/2) + 1) AS txt  
, query_plan
FROM sys.dm_exec_query_stats AS qs  
cross apply sys.dm_exec_sql_text(qs.sql_handle) AS st  
cross apply sys.dm_exec_query_plan (qs.plan_handle) AS qp 
ORDER BY 1 DESC
```

### Find all Databases and Size of Each

```SQL
SELECT DB_NAME(database_id) AS name_of_base, CAST(SUM(size) * 8. / 1024 AS DECIMAL(8,2)) AS total_size_b
FROM sys.master_files
GROUP BY database_id
```

### Find All Tables in DB, Including The Size and Row Count

```SQL
SELECT t.name AS table_name, p.rows AS table_rows, SUM(a.total_pages) * 8. / 1024 AS table_size
FROM sys.tables t JOIN sys.partitions p
ON t.object_id = p.object_id
JOIN sys.allocation_units a
ON a.container_id = p.partition_id
GROUP BY t.name, p.rows, a.total_pages
```


### Find All Contraints

```SQL
SELECT * FROM sys.objects
WHERE type_desc LIKE '%CONSTRAINT'
```

### Find SQL Job Step History

```SQL
select 
 j.name as 'JobName',
 s.step_id as 'Step',
 s.step_name as 'StepName',
 msdb.dbo.agent_datetime(run_date, run_time) as 'RunDateTime',
 ((run_duration/10000*3600 + (run_duration/100)%100*60 + run_duration%100 + 31 ) / 60) 
         as 'RunDurationMinutes',
[message]
From msdb.dbo.sysjobs j 
INNER JOIN msdb.dbo.sysjobsteps s 
 ON j.job_id = s.job_id
INNER JOIN msdb.dbo.sysjobhistory h 
 ON s.job_id = h.job_id 
 AND s.step_id = h.step_id 
 AND h.step_id <> 0
where j.name like 'LPD-SQLstaging%' 
and s.step_Name = 'STG_Port_Product_Target'

order by JobName, RunDateTime desc
```

### Find All Foreign Keys

```SQL
select 
 j.name as 'JobName',
 s.step_id as 'Step',
 s.step_name as 'StepName',
 msdb.dbo.agent_datetime(run_date, run_time) as 'RunDateTime',
 ((run_duration/10000*3600 + (run_duration/100)%100*60 + run_duration%100 + 31 ) / 60) 
         as 'RunDurationMinutes',
[message]
From msdb.dbo.sysjobs j 
INNER JOIN msdb.dbo.sysjobsteps s 
 ON j.job_id = s.job_id
INNER JOIN msdb.dbo.sysjobhistory h 
 ON s.job_id = h.job_id 
 AND s.step_id = h.step_id 
 AND h.step_id <> 0
where j.name like 'LPD-SQLstaging%' 
and s.step_Name = 'STG_Port_Product_Target'

order by JobName, RunDateTime desc
```


### Get fragmentation level of SQL Index

```SQL
SELECT dbschemas.[name] as 'Schema', 
dbtables.[name] as 'Table', 
dbindexes.[name] as 'Index',
indexstats.alloc_unit_type_desc,
indexstats.avg_fragmentation_in_percent,
indexstats.page_count
FROM sys.dm_db_index_physical_stats (DB_ID(), NULL, NULL, NULL, NULL) AS indexstats
INNER JOIN sys.tables dbtables on dbtables.[object_id] = indexstats.[object_id]
INNER JOIN sys.schemas dbschemas on dbtables.[schema_id] = dbschemas.[schema_id]
INNER JOIN sys.indexes AS dbindexes ON dbindexes.[object_id] = indexstats.[object_id]
AND indexstats.index_id = dbindexes.index_id
WHERE indexstats.database_id = DB_ID() --and dbtables.name = 'tblDIP_AttributeValues'
ORDER BY indexstats.avg_fragmentation_in_percent desc
```




