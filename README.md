# T-SQL Code Snippets
## My collection of useful SQL Queries that i use ofthen

----------------------

### Find all tables containing column with specified name

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

### Find all Databases and Size of each

```SQL
SELECT DB_NAME(database_id) AS name_of_base, CAST(SUM(size) * 8. / 1024 AS DECIMAL(8,2)) AS total_size_b
FROM sys.master_files
GROUP BY database_id
```

### Find all Tables in DB, Including the size and row count

```SQL
SELECT t.name AS table_name, p.rows AS table_rows, SUM(a.total_pages) * 8. / 1024 AS table_size
FROM sys.tables t JOIN sys.partitions p
ON t.object_id = p.object_id
JOIN sys.allocation_units a
ON a.container_id = p.partition_id
GROUP BY t.name, p.rows, a.total_pages
```


### Find all contraints

```SQL
SELECT * FROM sys.objects
WHERE type_desc LIKE '%CONSTRAINT'
```








