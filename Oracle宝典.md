Oracle 宝典
===

疑问题目： 3、4、5、7、8、9、12、17、18、20、24、27、31、32、33  

1. 如何备份控制文件  
sql> alter database backup controlfile to trace; 

2. 如何在不影响子表的前提下，重建一个母表。  
将子表的外键强制失效，重建母表，激活外键。  

3. pfile和spfile的转换  
create pfile from spfile;  
create pfile='d:\pfile.txt' from spfile;  

create spfile from pfile;  
create spfile='c:\oracle\spfile\spfileorcl.ora' from pfile='c:\oracle\pfile\initpfile.ora';  


4. 解释block,extent,segment  
一个table至少是一个segment，如果是分区表，则每个分区是一个segment。table可以看成是逻辑概念，而segment可以看成这个逻辑概念的物理实现。  
segment由一个或者多个extents组成，segment不可以跨表空间但可以跨数据文件。
extent有多个连续的block组成，不可以跨越数据文件。
block是由1个或者多个os块组成，是Oracle io 的最小存储单位。默认Oracle block 是8k。  

Oracle Data Blocks

At the finest level of granularity, Oracle database data is stored in data blocks. One data block corresponds to a specific number of bytes of physical database space on disk. The standard block size is specified by the DB_BLOCK_SIZE initialization parameter. In addition, you can specify up to five other block sizes. A database uses and allocates free database space in Oracle data blocks.

Extents

The next level of logical database space is an extent. An extent is a specific number of contiguous data blocks, obtained in a single allocation, used to store a specific type of information.

Segments

Above extents, the level of logical database storage is a segment. A segment is a set of extents allocated for a certain logical structure. The following table describes the different types of segments.

Segment	Description
Data segment	Each nonclustered table has a data segment. All table data is stored in the extents of the data segment.
For a partitioned table, each partition has a data segment.
Each cluster has a data segment. The data of every table in the cluster is stored in the cluster's data segment.

Index segment	Each index has an index segment that stores all of its data.
For a partitioned index, each partition has an index segment.
Temporary segment	Temporary segments are created by Oracle when a SQL statement needs a temporary database area to complete execution. When the statement finishes execution, the extents in the temporary segment are returned to the system for future use.
Rollback segment	If you are operating in automatic undo management mode, then the database server manages undo space using tablespaces. Oracle recommends that you use automatic undo management.
Earlier releases of Oracle used rollback segments to store undo information. The information in a rollback segment was used during database recovery for generating read-consistent database information and for rolling back uncommitted transactions for users.
Space management for these rollback segments was complex, and Oracle has deprecated that method. This book discusses the undo tablespace method of managing undo; this eliminates the complexities of managing rollback segment space, and lets you exert control over how long undo is retained before being overwritten.

Oracle does use a SYSTEM rollback segment for performing system transactions. There is only one SYSTEM rollback segment and it is created automatically at CREATE DATABASE time and is always brought online at instance startup. You are not required to perform any operations to manage the SYSTEM rollback segment.


Oracle dynamically allocates space when the existing extents of a segment become full. In other words, when the extents of a segment are full, Oracle allocates another extent for that segment. Because extents are allocated as needed, the extents of a segment may or may not be contiguous on disk.  

5. 如何获得表结构  
	1. describe table_name
	2. dbms_metadata.get_ddl('TABLE','EMP','SCOTT');  
	3. dbms_metadata.get_ddl('INDEX','PK_DEPT','SCOTT');  
	4. select  dbms_metadata.get_ddl('TABLE','EMP','SCOTT') from dual ;  
 
6. 为什么使用索引，什么情况下应该使用索引。  
为了快速获取少量数据。当获取的数据是全表数据的极少部分时，适合用索引。  

7. Oracle中的数据库和实例分别指的是什么？  
	1. 数据库--物理操作系统文件或磁盘的集合。
	2. 实例--一组Oracle后台进程/线程以及一个共享内存区，这些内存有同一个计算机上运行的线程/进程所共享。  

	实例和数据库之间的关系是：数据库可以由多个实例装载和打开，而实例可以在任何时间点装载和打开一个数据库。实例在其整个生存期中最多能装载和打开一个数据库。  

7. 数据库启动时经过几个状态。
	1. startup nomount  -- 用于重建控制文件或数据库
	2. startup mount	-- 用于改变数据库的归档或执行恢复状态
	3. startup open 	-- 正常启动

8. start restrict--约束启动，启动数据库后只有具有一定访问权限的用户才能访问数据库。如果需要改变这种情况，需要使用如下命令：  
alter system disable restricted session;


8. 如何区别v$和gv$视图？  
inst_id 指明集群环境中具体的某个instance。

9. 如何生成 explain plan？  
执行utlxplan.sql,建立plan表针对特定的sql语句，使用  
explain plan for select * from emp where empno=1005;  
select * from table(dbms_xplan.display);   

10. 解释$ORACLE_HOME和$ORACLE_BASE的区别？
$ORACLE_BASE是Oracle的根目录，$ORACLE_HOME是Oracle的产品目录。

11. 如何判断数据库时区和会话时区。  
select dbtimezone from dual;  
select sessiontimezone from dual;
`两者有什么区别呢？`  

12. GLOBAL_NAMES设置为Ture的作用。  ？？
global_names指明连接数据库的方式。如果设置为true,则会带有全局域名。比如 tjdb.tjdyf.com。 
global_name为什么很重要呢？  
因为在创建db link的时候，db Link的最终格式与此相关，实际上global_name由两部分组成，DB_NAME和DB_DOMAIN。在创建db link的时候，Oracle会自动将db_domain作为后缀添加上去。而且一旦加入就很难变更。  
查看当前的global_names  
select * from global_name;   


13. 如何加密PL/SQL程序？  
Wrap程序

14. audit trace 存放在哪个oracle目录结构中？  
unix $ORACLE_HOME/rdbms/audit  
windows-- event viewer 事件查看器  

15. 当用户进程出错，哪个后台进程负责清理它？  
PMON

16. 哪个后台进程负责刷新 materialized view?  
The Job Queue Processes.

17. 如何判断哪个session正在连接以及他们等待的资源？  
v$session/v$session_wait 

18. 描述什么是Redo log。  
Redo log是用于存放数据库数据改动情况的物理和逻辑结构。  
日志组常见操作：  	
	1. 添加日志组--alter database add logfile group 4 ('c:\oracle\redo04.log','d:\oracle\redo04.log') size 100m;  
	2. 添加新的日志成员--alter database add logfile member 'c:\oracle\redo011.log' to group 1;  
	3. 删除日志组--alter database drop logfile group 1; 然后从操作系统上删除对应的物理文件。  
	4. 删除日志组成员--alter database drop logfile member 'c:\oracle\redo011.log';  
	5. 清除联机重做日志--alter database clear logfile group 1 ;   
	一般不建议执行该操作。除非联机重做日志自身遭到了破坏。执行该操作后必须马上备份。  

19. 如何强制切换日志、强制检测点  
alter system switch logfile;  
alter system checkpoint;  

20. 列举出两个判断DDL改动的方法？

21. Coalescing做了什么？  
Coalescing针对于字典管理的tablespace进行碎片整理，将临近的小的extents合并成单个的大extent。

22. 临时表空间和永久表空间的区别是什么？  
临时表空间主要存储临时性数据，主要用于排序、索引重建以及临时表的存储等。而永久表空间主要用于永久性数据的存储。比如普通的表、索引等。

23. 在Oracle中，新建立的用户需要什么权限才能连接数据库？  
connect

24. 如何为一个tablespace添加数据文件？ （如何确定是固定大小，又或者如何控制增长速度） 
alter tablespace <tablespace_name> add datafile <datafile_name> size <size>;

25. 修改数据文件的大小。  
alter database datafile <datafile_name> resize <new_size>;

26. 如何查看数据文件、临时数据文件的大小？  
select file_name,bytes from dba_data_files;  
select file_name,bytes from dba_temp_files;  

27. 如何查看tablespace的剩余空间？  
select tablespace_name,sum(bytes)/1024/1024
from dba_free_space
group by tablespace_name;  

28. 如何判断谁往表里面增加了一条记录？  
auditing

29. 如何重建表索引？  
alter index <index_name> rebuild; 

30. 解释什么是partitioning（分区）以及它的优点。  
分区将大表和大索引分而治之，将大化小，更方便与管理与操作。  

31. 你刚刚编译了一个 package，但是有错误报道，如何显示错误信息？  
show errors

32. 如何收集统计信息？  
	1. 收集表的统计信息：  
	analyze table emp compute statistics;  
	2. 收集20%的行的统计信息：  
	analyze table emp estimate statistics sample 20 percent;  
	3. 用表中的10000行数据估计统计量。  
	analyze table emp estimate statistics sample 10000 rows;

33. 如何启动会话级别的trace？  
	1. dbms_session.set_sql_trace_in_session(sid,serial#,true);  
	2. alter session set sql_trace = true;  

	如果需要停止，则：  
	1. dbms_session.set_sql_trace_in_session(sid,serial#,false);  
	2. alter session set sql_trace = false;  

	设定标识符，方便跟踪：  
	alter session set tracefile_identifier='scotts_trace';    

	使用tkprof实用程序，将trace文件输出为较为容易理解的格式：  
	tkprof ora76942.ora trace_output.txt

34. import和SQL*LOADER有什么区别？  
import只能导入Oracle自身导出的二进制dmp文件。而SQL*LOADER则可以导入其他外部格式的数据文件。

35. 用于网络连接的2个文件。  
	1. tnsnames.ora
	2. sqlnet.ora 

36. ORA-01555的应对办法？  
具体的错误信息是 snapshot too old within rollback seg, 通常可以通过增大 rollback seg 来解决问题。当然也需要查看一下造成错误具体的sql语句。

37. standby的特点  
备用数据库(standby database)：oracle推出的一种高可用性（high available）数据库方案，在主节点与备用节点间通过日志同步来保证数据的同步，备用节点作为主节点的备份可以实现快速切换与灾难恢复。

38. SQL turnning advisor 和 SQL access advisor的主要区别是什么：  
SQL trunning advisor往往建议使用特定的SQL资源文件、收集优化器统计信息或重新编写查询以提高性能。  
SQL access advisor则往往建议使用新的索引或物化视图以提高SQL语句的运行性能。   

39. 什么时候发生快速索引全扫描？  
仅当索引包含需要返回的所有数据时，而不需要再查询表时。才可能发生快速全索引扫描（fast full index scan）。  
例如 select goodsid,goodsname from pub_goods  如果pub_goods(goodsid,goodsname)列都有索引的话。  

数据调优类
---

1. 列举常见的几种表连接方式。  
	1. merge join
	2. hash join
	3. nested loop

2. 不借助第三方工具，如何查看sql的执行计划？  
	1. sqlplus 中使用 set autotrace on;
	2. utlxplan.sql 创建 plan_table表  
	explain plan 
	set statement_id='query1'
	for 
	select * 
	from a
	where aa=1;

	select operation,options,object_name,object_type,ID,parent_id
	from plan_table
	where statement_id='query1'
	order by id;

3. 如何似乎用CBO。CBO和RULE的区别？  
在初始化参数中或者会话中设置optimizer_mode=choose/all_rows/first_row 就可以使用CBO。RBO在有索引的情况下一定会选择索引。而CBO则会根据统计信息选择代价最小的访问路径。  
CBO需要有最新的表统计信息，以反映实际情况。
RBO则需要良好的SQL语言书写习惯，选择最有效的表名顺序。  


4. 如何定位重要（消耗资源多）的sql。  
根据v$sqlarea中的逻辑读/diskread。以及寻找cpu使用过量的session，查出当前session的SQL语句。或者在10g中直接擦洗消耗最多的统计。  
使用CPU多的用户的session  
select a.sid,spid,status,substr(a.program,1,40) prog,
		a.terminal,a.sql_text,osuser,value/60/100 VALUE
from v$session a, v$process b, v$sessstat c
where c.statistic# = 12 
and c.sid = a.sid
and a.paddr = b.addr
order by value desc;

5. 说说你对索引的认识。（索引的结构、对dml影响、对查询影响、为什么能提高查询性能）
	1. 索引有B-Tree、Bitmap、Cluster等类型。  
	1. 默认的索引是B-Tree。
	2. 对insert的影响（索引分裂，要保证tree的平衡）
	3. 对delete的影响（删除行的时候要标志该节点为删除。但是当前节点标记为删除并不代表该块可用，要直到该块所有节点都标记为删除才可重用。）
	4. 对update的影响（如果更新表的索引字段，则要更新相应索引中的键值。导致跳跃。首先将原索引键值删除，然后在合适的位置插入新的键值。）

6. 绑定变量是什么？有什么优缺点？  
绑定变量就是一个占位符，使用绑定变量可以减少硬解析。在OLTP系统中绑定变量很有好处。因为OLTP中查询基本都是小查询，且重复频度较高。但是在OLAP系统中不宜使用绑定变量，因为OLTP系统中主要都是大查询，且由于绑定变量可能导致直接执行旧执行计划而导致效率很差（走错路径，选择了成本较高的路径）。

7. 如何稳定执行计划  
使用 stored outline 或者 直接在sql中使用hints  

8. 和排序相关的内存使用怎么调整？  
sort_area_size 8i
pga_aggregate_target  

9. 存在表T(a,b,c)，要去根据字段c排序后取出第21-30条记录显示。写出sql。  
select a,b,c from (select a,b,c from T order by c)  where rownum<=30
minus
select a,b,c from (select a,b,c from T order by c)  where rownum<=20  
或者
select * from 
( select rownum rn, a.*  from 
	( select a,b,c from T order by c
	)a 
)
where rn between 21 and 30;

10. pctused 和 pctfree 表示什么含义和作用。  
pctused：如果数据块的使用率小于pctused的值，则该块重新加入到freelist里面。
pctfree：如果数据块的使用率高于pctfree的值，则该块从freelist中移出。 

11. 简单描述table/segment/extent/block之间的关系。   
一个table至少是一个segment，如果是分区表，则每个分区是一个segment。table可以看成是逻辑概念，而segment可以看成这个逻辑概念的物理实现。  
segment由一个或者多个extents组成，segment不可以跨表空间但可以跨数据文件。
extent有多个连续的block组成，不可以跨越数据文件。
block是由1个或者多个os块组成，是Oracle io 的最小存储单位。

12. 本地管理表空间和字典管理表空间的特点，ASSM有什么特点？  
字典管理表空间使用freelist管理，本地管理表空间使用位图管理。 
字典管理表空间的空闲块列表存储在字典表中，在进行空间分配和回收的时候，可能造成字典表的争用。  
本地管理表空间的空闲块列表存储在数据头文件，能有效减少数据字典表的竞争，当分配和收缩空间是会产生回滚，不需要合并。 

13. 回滚段的作用是什么？  
保存数据的映像，保证数据读取的时间点的一致性。Oracle的数据多版本特定就是用回滚段来实现的。数据库在恢复和rollback时需要使用回滚段，一个事务一次只能使用一个回滚段。  

14. 日志的作用是什么？  
记录对数据库的操作的变化。便于以后的恢复。有以下特点:
	1. 每个数据库至少包含两个日志组，每个日志组至少包含两个日志成员。
	2. 日志文件组以循环方式进行写操作。 
	3. 每个日志文件成员对应一个物理文件。   

15. SGA主要有哪些部分，各部分主要用途是什么？  p59待续  
SGA是oracle为实例分配的一组共享缓冲存储区，用于存放数据库数据和控制信息，以实现对数据库数据的管理和操作。  
	1. db_cache--缓存最近读写过的数据。
	2. shared_pool--缓存sql执行计划、数据字典信息等。
	3. large_pool--parallel、rman等会用到
	4. java_pool--java程序和sqlj存储过程运行时用到。

16. Oracle的主要进程有哪些，都分别有什么作用？  
	1. smon--合并空间，实例恢复
	2. pmon--清理失败的进程
	3. arch--负责在日志切换时把已满的日志组进行备份或归档。
	4. lgwr--写日志进程，将重做日志从日志缓冲区写入在线重做日志文件 
	5. ckpt--检查点进程，触发检查点事件。每次当缓冲区高速缓存中的更改永久地记录在数据库中时，更新控制文件和数据文件中的数据库状态信息。
	6. dbwr--数据写入，把修改后的数据从数据缓冲区写入datafile
	7. reco--保证分布式事务的一致性，在分布式事务中，要么同时commit，要么同时rollback；
	8. cjq--负责将调度与执行系统中已定义好的job，完成一些预定义的工作。

17. 归档当前的redo log或者全部redo log   
alter system archive log current;
alter system archive log all;

疑问：7、8、12


备份恢复类
---

1. 备份如何分类？  
逻辑备份、物理备份  
或者  
冷备份、热备份  

2. 如果一个表在2012-07-11 11:00:00被drop掉，在有完善归档和备份的情况下，如何恢复。  
拷贝备份，然后  
recover database until time 2012--7-11 11:00:00  
alter database open resetlogs;

3. RMAN是什么？有什么特点？  
RMAN叫回复管理器，可以执行  
	1. 热备份
	2. 可以存储rman脚本
	3. 可以增量备份
	4. 自动管理备份集
	5. 可以备份和恢复整个数据库、表空间、数据文件、控制文件等。
	6. 可以压缩备份集

3. RMAN的优点：  
	1. 支持增量备份
	2. 自动管理备份文件
	3. 自动化备份与恢复
	4. 不产生重做日志
	5. 恢复目录
	6. 支持映像拷贝
	7. 新块的比较特性
	8. 备份的数据文件压缩处理
	9. 备份文件有效性检查功能

3. 进入RMAN：	
	1. rman target /
	2. rman  
	   connect target system/oracle@orcl

3. RMAN常用操作：
	1. RMAN实现脱机备份： 	
		1. 先关闭数据库  shutdown immediate;    
		2. 启动到mount状态  startup mount;  
		3. 执行备份    backup database;  
		4. 打开数据库  alter database open;  
	2. 备份控制文件：
		1. 备份到快速闪回区：    
			backup current controlfile;
		2. 备份到指定位置：  
			backup current controlfile format '/home/oracle/ctlfile.bk';  
	3. 将数据库设置为归档模式  
		1. shutdown immediate;  
		2. startup monut;  
		3. alter database archivelog;  
		4. alter database open;  
	4. 联机备份一个表空间：  
		backup tablespace sysaux;  
	5. 联机备份数据文件：  
		bakcup datafile 3;  
	6. 增量备份：　
		先做0级备份，然后做1级备份  
		1. backup incremental level 0 database;   
		2. bakcup incremental levle 1 database;  
	7. 启用和禁用块跟踪特性：  
		1. 启用块跟踪：   
			alter database enable block change tracking using file 'd:/oraclechange.log';
		2. 更改块跟踪文件的位置（必须在mount状态下进行）：  
			shutdown immediate;  
			startup mount;  
			alter database rename 'd:/oraclechange.log' to 'e:/oraclechange.log'; 
			alter database open;  
		3. 禁用块跟踪特性：  
			alter database disable block change tracking;  
		4. 查看禁用块跟踪特性的结果：　
			select filename,status,bytes from v$block_change_tracking;  

	8. 创建和维护恢复目录
		1. 创建恢复目录用户并授权  
			create user rman_backup identified by rman  
			temporary tablespace temp  
			default tablespace users  
			quota unlimited on users;  

			grant connect,resource to rman_backup;  
			grant recovery_catalog_owner to rman_backup;  

		2. 先连接到恢复目录，然后连接到目标数据库  
			$ rman catalog rman_backup/rman@orcl  
			rman>connect target system/oracle@leejia  
			或者  
			rman>connect catalog rman_backup/rman@orcl target system/oracle@leejiea  
		3. 创建恢复目录  
			$ rman catalog rman_backup/rman@orcl  
			rman> create catalog  
		4. 在恢复目录中注册目标数据库（必须先连接到数据库，然后注册）  
			$ rman catalog rman_backup/rman@orcl target system/oracle@leejiea  
			rman> register database;
		5. 同步恢复目录  
			rman> resync catalog;  
		6. 在恢复目录中保存脚本  
			1. 创建rman脚本  
				rman> create script rman_backup{  
				sql 'alter system checkpoint';  
				backup database format 'f:\offile_backup\back_%u.dbf';    
				backup current controlfile format 'f:\offline_backup\back_ctl_%u.dbf';  
				}  
			2. 执行rman脚本  
				rman> run { execute script rman_backup; }  
		7. 调用操作系统文件执行rman命令  
			$ rman catalog rman_backup/rman@orcl target system/oracle@orcl cmdfile 'rman_backup.rcv'  
		8. 将脚本文件转换为操作系统文件  
			print script rman_backup to file 'rman_backup.txt';  
	9. 使用RMAN 实现脱机备份的恢复（noarchivelog模式）不完全恢复     
		1. startup nomount;  
		2. restore controlfile from autobackup;--从备份的文件中恢复控制文件    
		3. alter database monut;    
		4. restore database;  
		5. alter database open;  
	10. 使用RMAN实现脱机备份的恢复 （archivelog 模式）  
		1. startup mount;  
		2. restore database;  
		3. recover database;   
		4. alter database open;  
	11. 从联机热备份中恢复  
		1. 恢复表空间  
			$ rman target system/oracle@orcl  
			rman> sql 'alter tablespace sysaux offline';
			rman> restore tablespace sysaux;  
			rman> recover tablespace sysaux;  
			rman> sql 'alter tablespace sysaux online';  
		2. 恢复数据文件  
			$ rman target system/oracle@orcl  
			rman> sql 'alter datafile /oracle/data/sysaux01.dbf offline';  
			rman> restore datafile '/oracle/data/sysaux01.dbf';  
			rman> recover datafile '/oracle/data/sysaux01.dbf';  
			rman> sql 'alter datafiel /oracle/data/sysaux01.dbf online';  
		3. 恢复数据块（假设数据文件6的部分块损坏）  
			1. 先验证备份集中的数据文件6的完整性。  
				rman> backup validate datafiel 6;  
			2. 查看当前数据文件6中损坏的数据块  
				select * from v$database_block_corruption;  --得知是具体哪个文件块损坏。  
			3. 使用rman完成数据块恢复。  
				rman> blockrecover datafile 6 block 118 from backupset;  
			4. 恢复数据文件6  
				rman> recover datafile 6;  
	12. RMAN的备份恢复验证指令  
		1. 验证备份集是否可用
			rman> validate backupset 30;  
			30 是 list backup summary 中的备份集的编号。  
		2. 验证某个需要恢复的对象是否在备份集中，比如某个表空间，某个数据文件等等。  
			rman> restore tablespace sysaux validate;   
			rman> restore datafile '/oracle/data/orcl/sysaux01.dbf' validate;  
		3. 验证恢复某个对象所需的备份文件是否存在  
			rman> restore database preview;  
			rman> restore tablespace sysaux preview;  
			rman> restore datafile 5 preview;  


3. 一致备份和不一致备份的区别  
	主要区别就是是否需要恢复。要实现一致备份可以关闭数据库使用脱机备份方式，也可以使数据库处于mount状态，使用rman工具实现。 

4.  imp的使用实例：
	1. 导出整个数据库  
		exp system/oracle@orcl full=y file=fullbackup.dmp; 
	2. 导出不包含数据的全库备份  
		exp system/oracle@orcl full=y rows=n file=fullbackup.dmp;  
	3. 导出指定的表  
		exp system/oracle@orcl tables=scott.emp,scott.dept file=scott.dmp;  
	4. 导出指定用户
		exp system/oracle@orcl owner=scott file=soctt.dmp;  
	5. 导出指定表空间  
		exp system/oracle@orcl tablespaces= users file=users.dmp;  
5. 传输表空间相关问题：  
	1. 传输表空间的限制：
		1. 源数据库和目标数据库必须使用相同的字符集和国家字符集  
		2. 目标数据库不能存在于传输表空间同名的表空间存在。如果出现这种情况，必须在源端或者目标端修改表空间名称，然后再传输。  
		3. 如果一些对象具有更底层的对象如物化视图，或者带有包含对象如分区表，这样的对象不能被传输，除非所有的底层对象或者包含对象在传输表空间集中。  
		4. 从Oracle 10g开始，可以传输XMLTyptes的传输表空间，但是必须使用imp/exp工具集，而不能使用数据泵。当使用exp工具时，确保constraints和trigger参数设置为y（默认值就是y）。  
		查询包含XMLTyptes的表空间：  
			select distinct p.tablespaces_name
			from dba_tablespaces p, dba_xml_tables x, dba_users u, all_all_tables t  
			where t.table_name = x.table_name   
			and t.tablespace_name = p.tablespace_name  
			and x.owner = u.username;  
	2. 传输表空间必须是自包含的，以下情况违反了自包含。
		1. 传输表空间中存储了另一个表空间中某表的索引。
		2. 分区表部分在传输表空间中。  
		3. 传输表空间中包含LOB列，该列指向传输表空间外的LOB对象。
	3. 实现传输表空间的步骤(相同平台)： 
		1. 查询当前的平台的endianess。  
			select * from v$transportable_platform;  --结果集中的各种平台下表空间可以相互传输。  
		2. 查询传输表空间的自包含。  
			sql> execute dbms_tts.transprot_set_check('test2',true);  
			sql> select * from transport_set_violations;  --如果没有任何内容则表示自包含没问题。  
		3. 将表空间设置为read only。 
			sql> alter tablespace test2 read only;  
		4. 导出表空间test2的元数据。  
			d:\> exp system/oracle file=expdp.dmp tablespaces=test2 transprot_tablespace=y  
		5. 将表空间的元数据和数据文件导入到目标数据库ORCL1;  
			d:\> imp system/oracle file=dxpdp.dmp datafiles=d:\oracle\oradata\test2.dbf transport_tablespace=y  
		6. 验证表空间test2  
			sql> select tablespace_name, status, contents  
				 from dba_tablespaces   
				 where tablespace_name='TEST2';   
		7. 修改目标数据库的表空间TEST2为读写模式。   
			sql> alter tablespace test2 read write;  
		8. 修改源数据库表空间TEST2为读写模式。  
			sql> alter tablespace test2 read write;  
	4. 使用EXP/IMP实现跨平台表空间的迁移（不同字节序列）  
		1. 检查源数据库和目标数据库的字节序列格式：  
			select d.platform_name, endian_format  
			from  v$trnasportable_platform tp, v$database d
			where tp.platform_name = d.platform_name;  
		2. 检查表空间的自包含：  
			sql> execute dbms_tts.transport_set_check('TEST2',true);
			sql> execute dbms_tts.transport_set_check('TEST2','USERS',ture) -- 如果要验证多个表空间。    
			sql> select * from transprot_set_violations;  
		3. 将表空间切换为read模式： 
			sql> alter tablespace test2 read only;  
		4. 导出表空间test2的元数据：  
			d:\> exp system/oracle file=expdp.dmp tablespaces=test2 transprot_tablespace=y  
		5. 在源数据端转换传输表空间元数据集的endianess：  
			d:\> rman target system/oracle
			rman> convert tablespace test2 to platform 'Solaris[tm] OE (32-bit) '
				  format 'd:\expdp_trans.dmp';
			经过上述转换步骤，表空间TEST2的所有数据文件完成了格式转换，使得该数据文件支持Solaris[tm] OE (32-bit)平台的字节序列存储格式，并将转换后的数据文件保存为 d:\expdp_trans.dmp文件。 
		6. 使用imp在目标端导入TEST2表空间。  
			$ imp system/oracle file=/u01/bk/expdp.dmp datafiles=/u01/bk/expdp_trans.dmp transport_tablespace=y  

		7. 如果是在目标端进行数据格式转换：  
			rman> convert datafile '/u01/finance/orabackup/test2.dbf' to platform 'Solaris[tm] OE (32-bit)'
				  from platform='Microsoft Windows IA (32-bit) ''
				  db_file_name_covert='/u01/finance/orabackup/', '/u01/finance/oradata/'
				  parallelism=5; 

6. impdp/expdp中的参数文件的使用：  
	parfile:   exp.par  
	direcotry = pump_dir   
	dumpfile = para_data_%u.dmp  
	content = data_only  
	exclude = table:"in('emp','dept')"  
	logfile = para_data.log  
	filesize = 100m  
	parallel = 2  
	job_name = pram_data_only  

	d:\> expdp scott/tiger@orcl parfile=exp.par

7. impdp/expdp实例： 
	1. impdp导入产生SQL语句
		impdp system/oracle@orcl directory = pump_dir dumpfile = scott_tables sqlfile = scott_table.sql 
	2. 重新映射模式  
		impdp system/oracle@orcl dumpfile = pump_dir:schema_scott.dmp remap_schema = scott:linzi  
	3. 重新映射数据文件  
		impdp system/oracle@orcl dumpfile = pump_dir:backup_full.dmp remap_datafile = 'c:\mydb.dbf':'d:\mydb\newdb.dbf'  
	4. 重映射表空间  
		impdp system/oracle@orcl dumpfile = pump_dir:backup_full.dmp remap_tablespace = 'users':'mynewusers'  

8. 使用数据泵迁移表空间  
	1. 验证表空间  
		sql> execute sys.dbms_tts.transport_set_check('users',true);
	2. 将表空间置于只读状态  
		sql> alter tablespace user read only;  
	3. 导出迁移表空间的元数据  
		d:\> expdp system/oracle@orcl dumpfile = pump_dir:transport_users_tbs.dmp transprot_tablespaces = users  
	4. 拷贝数据文件  
	5. 在目标数据库导入迁移表空间  
		d:\> impdp system/oracle@orcl dumpfile = pump_dir:transport_users_tbs.dmp  transport_datafiles='users01.dbf'  

9. 用户管理的备份和恢复  
	1. 用户管理的数据文件联机热备份  
		1. 将数据库设置为归档模式(前置条件)  
			shutdown immediate  
			startup mount  
			alter database archivelog  
			alter database open  
		2. 查找数据文件以及对应的表空间  
			select file_name, tablespace_name 
			from dba_data_files  
		3. 将表空间置为备份模式  
			sql> alter tablespace users begin backup;  
			或者也可以直接将数据库置为备份模式  
			sql> alter database begin backup;  
		4. 将数据文件备份到其他目录  
			sql> host copy f:\oracle\data\users01.dbf d:\backup\ 
		5. 将表空间退出备份模式  
			sql> alter tablespace users end backup;  
	2. 备份控制文件  
		sql> alter database backup controlfile to 'f:\backup\ctlbk.ctl';  
	3.使用END BACKUP指令恢复表空间备份异常  
		如果数据库已经异常关闭，而此时的数据文件依然处于备份模式，该数据文件不会被打开，此时需要关闭该数据文件的备份模式，或者对该数据文件实现实例恢复。以下是实验模拟：  
		1. 将表空间置为备份模式。 
			sql> alter tablespace index_tbs begin backup;  
		2. 异常关闭数据库  
			sql> shutdown abort  
		3. 启动数据库，报错。 提示文件需要介质恢复。  
		4. 关闭数据库，并启动到mount状态。  
			sql> shutdown immeidate  
		5. 结束备份模式  
			sql> alter database end backup; or alter tablespace index_tbs end backup;
		6. 打开数据库  
			sql> alter database open;  

10. 回收站的使用：  
	1. 查询回收站内容 
		sql> select owner,original_name,object_name,ts_name,droptime  
		from dba_recyclebin; 
	2. 恢复删除的表  
		sql> flashback table emp to before drop;  
	3. 恢复删除的表并重新命名  
		sql> flashback table "BIN$YSJKDFSKFKWERSDKFJ8J34" to before drop  
		rename to new_test;  
	4. 表相关的索引在被删除放入回收站，如果表后来被恢复，那么索引也被恢复，但是索引名将不是原名，而是一个长字符串，需要修改。  
	5. purge永久删除表  
		sql> drop table test1 purge;  
	6. 永久删除表空间  
		sql> drop tablespace test including contents;  
	7. 在回收站中删除表  
		sql> pruge table  test2;   
	8. 删除回收站中某个表空间的所有表    
		sql> purge tablespace users;  
	9. 删除回收站中与某个表空间相关的特定用户的表  
		sql> purge tablespace users user scott; 

11. 闪回数据库  
	将闪回缓冲区中的数据写入闪回日志的操作由RVWR进程负责，一旦启动了闪回数据库，该进程会自动启动。  
	如果数据库中绝大多数是查询操作，此时闪回数据库开销很小，因为他不需要做什么。所以，闪回数据库的系统开销取决于是否有密集的写操作。(对于闪回数据库而言，闪回日志不会被归档)  

	Oracle默认不启动闪回数据库，如果需要启动闪回数据库特性，必须将数据库设置为归档模式，并启用闪回恢复区，因为闪回日志文件存放在闪回恢复区中。如果是RAC环境下，必须将恢复闪回区存储在集群文件或ASM中。  

	1. 启用闪回特性 
		sql> show parameter db_recovery_file_dest;  
		sql> show parameter db_recovery_file_dest_size;  
		sql> show parameter db_flashback_retention;  
		sql> shutdown immediate;  
		sql> startup mount;  
		sql> alter database flashback on;  
		sql> alter database open;  
	2. 查看闪回数据库启动状态  
		sql> select dbid,name,flashback_on  
			 from v$database;  
	3. 关闭表空间闪回数据库特性  
		sql> alter tablespace users flashback off;  
	4. 关闭数据库闪回特性  
		sql> alter database flashback off;  
	5. 使用闪回特性  
		既可以在rman中使用，也可以在sql中使用。  
		在执行闪回数据库时，需要将数据库切换到mount状态，在闪回数据库结束后必须使用  alter database open resetlogs 打开数据库，即需要重新设置重做日志，使得重做日志序列号重新计数。  
		1. rman中使用  
			rman> flashback database to time = to_date('2012/05/25 12:34:00','yyyy/mm/dd hh24:mi:ss');
			rman> flashback database to scn = 638832;  
			rman> flashback database to sequence = 345 thread =1 ;  --将数据库闪回到特定日志序列之前的状态，不包括序列号345.  
		2. sql中使用  
			sql> flashback database to timestamp(sydate-1/24);  
			sql> flashback database to scn 678854;  
		3. 闪回特性使用完整实例  
			1. create table test as select * from emp;  
			2. select count(*) from test;   
			3. select current_scn from v$database;  -- 获得当前scn，比如是  609845。  
			4. insert into test select *  from emp ; 
			5. commit;  
			6. 关闭数据库，并启动到mount状态  
				shutdown immediate; 
				startup mount;
			7. flashback database to scn 609845;
			8. alter database open resetlogs;  
			9. select count(*) from test;   --恢复完成。  
		4. 查询可以闪回到最小的SCN以及闪回到的时间  
			sql> select oldest_flashback_scn, oldest_flashback_time 
				 from v$flashback_database_log;  
		5. 查看当前闪回日志记录的各种开销  
			sql> select * from v$flashback_database_stat;  
		6. 查询快闪恢复区的空间使用情况  
			sql> select name,space_limit,space_used,space_recliamable,number_of_files  
				 from v$recovery_file_dest;  

12. 复原点技术  
	复原点技就是将数据恢复到改点时的状态。普通的复原点无非就是SCN的别名,还是无法确定闪回数据一定成功。因为这依赖于需要的闪回日志是否存在，而闪回日志又由快闪恢复区管理。  
	还有一种有保证的复原点技术，这种技术可以保证在快闪恢复区空间可以保证的情况下，总可以闪回到该复原点。如果快闪恢复区空间不足，则数据库关闭。  
	1. 创建普通复原点  
		sql> create restore point rp1;  
	2. 查询复原点信息  
		sql> select name,scn,storage_size,guarantee_flashback_database  
			 from v$restore_point;  
	3. 创建有保证的复原点  
		sql> create restore point grp1 guarantee flashback database;  
	3. 查询复原点信息  
		sql> select name,scn,storage_size,guarantee_flashback_database  
			 from v$restore_point; 
	4. 删除有保证的复原点(和删除普通复原点是一样的)  
		sql> drop restore point grp1;  
		sql> drop restore point rp1;  
	


3. 快速闪回区的优点：  
实现了文件备份的自动管理，使得备份与恢复数据库更简单（指令更简洁），并且可以集中管理磁盘空间。   
两个关于闪回的重要参数：　
db_recovery_file_dest  
db_recovery_file_dest_size  

4. 将闪回缓冲区中的数据写入闪回日志的操作由RVWR进程负责，一旦启动了闪回数据库，该进程会自动启动。 



4. 对于一个要求恢复时间比较短的系统（数据库50G，每天归档5G),你如何设计备份策略。  
参考：每天一个全备份  

5. 对一个存在性能问题的系统，说出你的诊断处理思路。  
做一个stackpack or AWR，了解大概情况，确定相关参数设置是否正确。
根据报告中的 top5 event ，top sql 等方面来做相应的调整。 
查看v$system_event/v$session_event/v$session_wait
查看v$sql/v$sqltext/v$sqlarea确定disk_reads、buffer_gets/execution等信息。 
iostat/top/vmstat

6. 对stackpack有什么认识？  
一个性能诊断工具，其本质就是在两个时间点采集系统数据，然后根据两个snapshot，产生一个对比报告。  

7. 如果系统现在需要创建一个很大表上创建一个索引，你会考虑哪些因素，如何做以尽量减少对应用的影响。  
	1. 考虑增大sort_area_size(8i)/pga_aggregate_target(9i)
	2. 用并行方式来建立
	3. 在系统空闲时候建立  
	4. 将索引存储在与表不同的表空间

8. 可以获取数据库启动时间的视图是？  
v$instance  

8. In which dictionary table or view would you look to determine at which time a snapshot or mview last successfully refreshed?  
user_mviews or user_mview_refresh_times。  

8. raid10和raid5有什么区别。  
raid10是先镜像后条带化，适合对写入速度要求较高的系统，特别是online redolog文件。raid5适合大部分数据库系统和数据仓库系统。读性能优于写性能。但是在redolog不适合用raid5。因为redolog是线性写入，而raid5会将数据打散。

9. 什么是聚簇索引，什么是非聚簇索引，什么又是主键？  
聚簇索引的顺序就是数据的物理存储顺序，叶节点就是数据节点。  
非聚簇索引的顺序与数据物理排列顺序无关，叶节点仍然是索引节点，只不过有一个指针指向对应的数据块。   
主键是能够唯一表示数据表中每条记录的字段或者字段的组合。通过它可以强制表的实体完整性。  

10. 什么是事务？  
事务是用户定义的一个操作序列，这些操作要么全部都做，要么全部不做。事务具有ACID性--原子性、一致性、隔离性、持久性。  

11. truncate 和 delete 的区别？  
	1. delete将产生回滚信息，而truncate不产生。
	2. truncate是DDL操作，执行隐含的commit。不能回滚。而delete是DML操作，可以回滚。  
	3. truncate会重置表的高水位线，但是delete不会。
	4. truncate 并不触发delete触发器。 
	5. 没有对象权限允许一个用户truncate另一个用户的表，除非拥有drop any table的权限。  
	6. 当一个表被truncate，表奇迹索引的存储大小将被充值回初始大小。而delete则不会。

12. 如何查看主机CPU、内存、IP和磁盘空间情况？  
	1. 磁盘空间--df
	2. 

13. 常用的进程管理、主机性能查看命令有哪些？  

14. 建立组dba和该组下的用户oracle，默认shell为bash。  


15. 写crontab，让脚本/opt/test.sh在每周日晚上8：000执行。  
0 20 7 0 0 /opt/test.sh   

16. 如何查找当前目录及其所有子目录下含有"ORA-"或"warning"字符内容的所有带log后缀的文件。  
ls - R  *.log | grep "ORA-|warning"   

17. 写一个shell，完成自动登录数据库（用户名密码是test/testpwd，网络连接符是db_wending）并获取数据库当前SCN的功能。  
sqlplus test/tespwd@db_wending  
select current_scn from v$database  

18. 如何了解一个运行已经很久的数据库系统？  

19. 如何获得SQL的执行计划和统计信息？  

20. 列出你常用的数据字典视图和动态统计信息。   

21. 描述你使用过的数据库备份和恢复技术，及其优缺点。  

22. 前台系统反映非常慢，需要你去分析诊断。请详细陈述诊断流程，包括写下可能用到的操作系统命令、数据库视图等。  

23. 公司准备把Oracle 9i 升级为 10.2.0.4 但批准的停库时间仅仅为1-分钟，你打算如何应对？  

24. 详细阐述RAC、Dataguard、Stream Replication、Advanced Replication等HA技术的原理和优缺点。  

25. 写存储过程，把当前用户下的数据库对象个数信息按对象类型分组输出。  
create or replace procedure show_objects_count
is 
being
	dbms_output.putline('object_type'||chr(13)||'counts'); 
	for c in(
				select object_type,count(object_name) counts
				from user_objects
				group by object_type;

			) loop
		dbms_output.putline(c.object_type||chr(13)||c.counts);
	end loop;
end;

26. 写下你常常关注的oracle 初始化参数。

27. 怎样得到数据库的SID  
select name from v$database;  

28. 计算一个表占用的空间大小。  
select owner,table_name,num_rows,blocks*<block_size>/1024/1024 "Size M",empty_blocks,last_analyzed  
from dba_tables  
where table_name = 'XXX';  

dba_tables中包含了：table_name,tablespace_name,ini_trans,max_trans,logging,num_rows,blocks,chain_cnt_avg_row_len,cache,sample_size,last_analyzed,partitioned,row_movement。  

29. extent的分配方式:  
默认第一次分配的extent为8个block，当分配到第16个extent还是不够的时候，从第17个extent开始就是128个block，直到第78个。从第79个开始就分配1024个blcok，一直分化下去。

create  table zx_lxj_test1  
(credate date,  
goodsid number,  
qty number)  
nologging  
storage  
( initial 65M  
 -- next 14000M  
);  

如果在创建表的时候，指定了initial大小： 	
	1. <=1024k ,  则初始分配的每个extent为8个block。
	2. >1024k  ,  则初始分配的每个extent为128个block。
	3. >64M    ,  则初始分配的每个extent为1024个block。

30. 索引在表空间的存储情况  
select segment_name,count(*) "extent counts" ,sum(bytes)/1024/1024 "Size M"  
from dba_extents   
where segment_type='INDEX'  
and owner='XXX'  
group by segment_name;  

31. 查看表空间可用空间：  
select a.tablespace_name,sum(a.bytes)/1024/1024 "Size M",sum(a.blocks) "Block counts"  
from dba_free_space a  
group by a.tablespace_name  
order by a.tablespace_name；   

32. 查看数据库版本信息：  
select * from v$version;  
包含版本信息，核心版本信息，位数信息等。  
如果只查看程序的位数，在linux，可以通过   
file $ORACLE_HOME/bin/oracle  
来查看。

33. v$license 中记录了数据库打开的最大会话数，还记录了cpu的核数。  

34. 移动表和索引到另一个空间：  
alter table zx_lxj_test1 move tablespace TEST;  
alter index zx_lxj_test1_idx rebulid tablespace test;  

alter table zx_lxj_test1 add constraint zx_lxj_test1_pk primary key(goodsid);

alter table zx_lxj_test1 disable primary key;
alter talbe zx_lxj_test1 enable primary key;

alter trigger zx_lxj_test1_tri disable;  
alter trigger zx_lxj_test1_tri enable;  


34. 将小表放入keep池，或者移出keep池。  
alter table <table_name> storage(buffer_pool keep);  
alter table <table_name> storage(buffer_pool default);  


35. 查询做比较大的排序进程：   
select b.TABLESPACE,b.SEGFILE#,b.SEGBLK#,b.BLOCKS,a.SID,a.SERIAL#,a.USERNAME,a.OSUSER,a.STATUS  
from  v$session a,v$sort_usage b  
where a.SADDR=b.SESSION_ADDR  
order by b.TABLESPACE,b.SEGFILE#,b.SEGBLK#,b.BLOCKS   

36. 查询被锁对象  
select /*+ rule */ lpad('--',decode(b.block,1,0,4))||s.username user_name,  
b.type,o.owner||'.'||o.object_name object_name,  
s.sid,s.serial#,decode(b.REQUEST,0,'BLOCKED','WAITING') status  
,s.SQL_ID  
FROM dba_objects o,v$session s,v$lock v,v$lock b  
where v.ID1=o.object_id and v.SID=s.sid  
and v.SID=b.SID and (b.BLOCK=1 or b.REQUEST>0)  
--and v.TYPE='TM'  
order by b.ID2,v.ID1,user_name desc;  



其他
---

1. linux的运行级别  
	0--系统停机  
	1--单用户模式  
	2--多用户模式  
	3--网络多用户模式  
	4--保留  
	5--X11模式（图形界面）  
	6--重启  
系统将其设置在/etc/inittab配置文件中：
	id:3:initdefault:  





ASM
---
ASM--ASM自动存储管理是Oracle推荐的一种智能化数据库文件存储管理工具，它采用OMF文件格式来创建目录文件，它通过自动管理磁盘组来平衡I/O，通过镜像技术来实现数据文件的高可用性。

ASM磁盘管理的几个优势：  
	1. 自动磁盘管理  
	2. 文件级镜像  
	3. 避免磁盘热点  
	4. 方便的管理数据库文件  
