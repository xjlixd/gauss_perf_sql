# gauss_perf_sql
--查看当前数据库的配置项情况
SELECT * FROM DV_PARAMETERS
--查看版本
SELECT * FROM DV_VERSION

--查看慢SQL
select * from sys.DV_LONG_SQL;

--记录系统中所有表（包含所有系统表和用户表）信息
SELECT * FROM SYS.SYS_TABLES;

--查看系统统计
select* from  V$SYSSTAT

--记录系统表中的存储过程、自定义函数、自定义高级包和触发器的信息
select * from  sys.SYS_PROCS

--查看连接池状态
select * from  DV_CONNPOOL_STATS

--记录分布式两阶段事务的信息， 查看CN的2PC状态
SELECT * FROM SYS.SYS_PENDING_DIST_TRANS;

--记录系统中未决的两阶段事务的信息
SELECT * FROM SYS.SYS_PENDING_TRANS;

--查看全局事务状态
SELECT * FROM SYS.DV_GLOBAL_TRANSACTIONS;

--------------------------------------------
--性能视图
--查看数据库实例信息
SELECT * FROM DV_INSTANCE;

--查看当前数据库的数据文件分配情况
SELECT * FROM DV_DATA_FILES

--查看当前申请的内存信息
SELECT * FROM DV_GMA;

--查看当前SGA内存的统计项, 记录内存指针
SELECT * FROM DV_GMA_STATS; 

--查看所有事务信息。
SELECT * FROM DV_ALL_TRANS; 

--查看当前归档日志情况
SELECT * FROM DV_ARCHIVED_LOGS;

--查看BUFFER_POOL中索引各层页面占用情况。缓存序号,索引层号,索引页数量
SELECT * FROM DV_BUFFER_INDEX_STATS;

--查看BUFFER_POOL中各类页面的占用情况
SELECT * FROM DV_BUFFER_PAGE_STATS;

--查看当前BUFFER_POOL的分配情况
SELECT * FROM DV_BUFFER_POOLS;

--查看SQL的DML语句执行情况。
SELECT * FROM DV_SQLS 

--查看当前系统统计项。
SELECT * FROM DV_SYS_STATS 

--查看当前操作系统的CPU/MEM使用情况。
SELECT * FROM DV_SYSTEM

--连接池和对应工作线程池信息。
SELECT * FROM DV_REACTOR_POOLS

--查看当前系统事件。
SELECT * FROM DV_SYS_EVENTS

--用于在主机上查询和备机的连接关系、日志发送情况等信息，
SELECT * FROM DV_HA_SYNC_INFO;

--查看当前的表空间情况。
SELECT * FROM  DV_TABLESPACES

--查看所有temp undo segment队列的实时状态信息。
SELECT * FROM  DV_TEMP_UNDO_SEGMENT

--查看事务信息。
SELECT * FROM DV_TRANSACTIONS

--系统存在大量的“buffer busy waits”时，查看所有等待事件的统计信息。
SELECT * FROM DV_WAIT_STATS

--查看当前锁资源情况。
select* from DV_LOCKS 
--显示锁对象的信息。
select* from DV_LOCKED_OBJECTS
 
 --查看当前申请的内存信息
SELECT * FROM DV_GMA;

--查看当前SGA内存的统计项, 记录内存指针
SELECT * FROM DV_GMA_STATS; 


 
--查看表空间的信息
select * from ADM_TABLESPACES 

--查看表的大小
select * from adm_segments where TABLESPACE_NAME='HWPOC_GSDB' and owner='HWPOC_GSDB_FI' order by pages desc 

--查看索引
select * from DB_INDEXES

--合计表的大小
select  t1.owner,t1.segment_type,sum(bytes)/1024/1024/1024 as `Size(G)`
from adm_segments t1 
where t1.TABLESPACE_NAME='HWPOC_GSDB' and t1.owner='HWPOC_GSDB_FI'  
  and (t1.segment_name='T_GL_VOUCHERENTRY' 
    or t1.segment_name in (select INDEX_NAME from DB_INDEXES where table_name ='T_GL_VOUCHERENTRY' and owner='HWPOC_GSDB_FI')
)
group by t1.owner,t1.segment_type

--查看慢SQL
SELECT ELAPSED_TIME/1000000 as `Total Time(s)`
   ,EXECUTIONS as `EXECS`
   ,case when EXECUTIONS>0 then ELAPSED_TIME/EXECUTIONS/1000000 else NULL end  as `Average Run Time(s)`
   ,IO_WAIT_TIME/1000000 as `IO WAIT TIME (s)`
   ,CON_WAIT_TIME/1000000 as `Lock Time(s)`
   ,CPU_TIME/1000000 as `CPU Time(s)`
   ,LAST_ACTIVE_TIME as `Last Run Time`
   ,* 
FROM DV_SQLS 
where MODULE='JDBC' 
order by `Average Run Time(s)` desc,ELAPSED_TIME desc


--查看锁
select count(*) from dv_locks where SID=69

--查看索引
select * from user_indexes where table_name='T_GL_VOUCHER'
select * from user_ind_columns where index_name=('IDX_GL_VCH');

--查看所有的会话
select * from v$session
select * from dv_sessions where status='ACTIVE'
--终止会话的线程
select 'alter system kill session '||''''||sid||','||serial#||''''||' immediate;' from v$session where status='ACTIVE'
alter system kill session '53,26' ;

--查看buffer busy waits的等待状态
select * from DV_WAIT_STATS; CLASS--等待时间的名称 COUNT--发生等待事件的次数 TIME--等待时间总和（单位：毫秒）

--列出当前系统的等待时间，高斯中为DV_SESSION_WAITS

	SELECT event
			, sum(decode(wait_time,0,1,0)) "Curr"
			, sum(decode(wait_time,0,0,1)) "Prev", count(*)"Total" 
	FROM v$session_wait GROUP BY event ORDER BY count(*);
--查看当前实例内存使用信息。
select * from DV_MEM_STATS;

--查看正在运行的SQL
SELECT SID,SERIAL#, EVENT, PROGRAM, CLIENT_IP
    , (SYSDATE - SQL_EXEC_START)*86400, WAIT_SID,CURRENT_SQL,SQL_ID, MODULE 
FROM DV_SESSIONS WHERE STATUS = 'ACTIVE';

其中：
l WAIT_SID表示阻塞会话ID，为空表示不阻塞。
2 (SYSDATE - SQL_EXEC_START)*86400表示该会话当前SQL已经执行的时间，单位：秒。


--对表执行统计分析，使用sql语句执行
analyze table my_table compute statistics;
等价于
analyze table my_table compute statistics for table for all indexes for all columns;

--查看表的统计信息
select table_name,num_rows,blocks,empty_blocks from user_table where table_names in ('T1','T2','T3','T4');

--查看字段的统计信息
select table_name,column_name,num_distinct,low_value,high_value,density from user_tab_columns where table_name in ('T1','T2','T3','T4');


--查看索引的统计信息
select table_name,index_name,blevel,leaf_blocks,distinct_keys,avg_leaf_blocks_per_key avg_leaf_blocks
      ,avg_data_blocks_per_key avg_data_blocks,clustering_factor,num_rows
from user_indexes where table_name in ('T1','T2','T3','T4');

--在线重建索引
alter index idx_xxxx rebuild online;
rebuild online 执行表扫描获取数据，有排序的操作;

alter index indexname rebuild; 
Rebuild以index fast full scan（or table full scan）方式读取原索引中的数据来构建一个新的索引，有排序的操作;

--判断重建索引的依据
select height,DEL_LF_ROWS/LF_ROWS from index_stats;
说明：当 查询出来的 height>=4 或者 DEL_LF_ROWS/LF_ROWS>0.2的索引，该索引考虑重建 。

--判断索引是否倾斜的严重,是否浪费了空间,对索引进行结构分析：
Analyze index index_name validate structure;
