Oracle Concept 笔记
===

1. 临时表也可以加索引、建立视图和触发器。  
2. 外部表只有结构，没有存储，描述如何展示外部表的数据。   
3. 外部表一般都是只读的。外部表不支持索引，虚拟列和列对象。   
4.  通常，普通表都是heap table，都是无序存储的。   
5. 正确的使用聚簇表具有如下好处：    
	1. 减少连接的聚簇表的I/O。  
	2. 提高仿佛连接聚簇表的时间。  
	3. 减少了存储空间，因为聚簇表之间的key只存储一次，而不需要重复存储。  

6. 不可用的索引和索引分区将不再占用空间。当一个索引由正常状态变为不可用时，数据库将释放其所占用的空间。   在进行大批量导入是，可以先将索引置为不可用，然后导入数据，最后将索引重新启用，并重建索引（rebuilding）。  

7. 主键和唯一键会自动添加索引，不过请记得要为外键添加索引。

8. 唯一索引和非唯一索引： 
	1. 唯一索引中，每一个rowid存在于唯一的数据值。叶块中的数据只是按照key值来排序。 
	in a unique index, one rowid exists for each data value. The data in the leaf blocks is sorted only by key. 
	2. 非唯一索引中，是按照key和rowid（升序）来排序的。  
	For a nonunique index, the rowid is included in the key in sorted order, so nonunique indexes are sorted by th index key and rowid(ascending).  

9. Index类型：  
	1. B-tree indexes  
		1. Index-organized tables 
		2. Reverse key indexes  
		3. Descendign indexes  
		4. B-tree cluester indexes  
	2. Bitmap and bitmap join indexes  
		1. Bitmap indexes on a single table  
		2. Bitmap join index  
		select count(*) 
		from employees,jobs  
		where employees.job_id = jobs.job_id  
		and jobs.job_title='Accountant';  

		The preceding query would typically use an index on jobs.job_title to retrieve the rows for Accountant and then the job ID, and an index on employees.job_id to find the matching rows. To retrieve the data from the index itself rather than from a scan of tables, you could create a bitmap join index as follows:  

		create bitmap index employees_bm_idx  
		on employees (jobs.job_title)
		from employees, jobs
		where employees.job_id=jobs.job_id;
	3. Function-based indexes  
		A function-based index is also useful for indexing only specific rows in a table. For example, the cust_valid column in the sh.customer table has either I or A as a value. To index only the A rows, you could write a function that returns a null value for any rows other than A rows. You could create the index as follows:  
		create index cust_valid_idx  
		on customers ( case cust_valid when 'A' then 'A' end);  
	4. Application domain indexes  

10. 索引扫描  
	In a index scan, the database retrieves a row by traversing the index, using the indexed column values specified by the statement. If the database scans the index for a value, the it will find this value in nI/Os where n is the height of the B-tree index. This is the basic principle behind Oracle Database indexes.  
	If a SQL statement access only indexed columns, then the database reads values directly form the index rather than from the table. If the statement accesses columns in addition to the indexed columns, then the database uses rowids to find the rows in the table. Typically, the database retriees table data by alternately reading an index block and then a table block.  

11. 索引扫描类型:  
	1. Full Index Scan
	2. Fast Full Index Scan--is a full index scan in which the database access the data in the index itself without accessing the table, and the database read th eindex blocks in no particular order.    
	3. Index Range Scan  
	4. Index Unique Scan-- In contrast to an index range scan, an index unique scan must have either 0 or 1 rowid assoicated with an index key. The database performs a unique scan when a predicate references all of the columns in a UNIQUE index key using an equality operator. An index unique scan stops processing as soon as it finds the first record because no second record is poosible.   
	5. Index Skip Scan--An index skip scan uses logical subindexes of a composite index. The database "skips" through a single index as if it were searching separate indexes. Skip scanning is beneficial if there are few distinct values in the leading column of a composite index and many distinct values in nonleading key of the index.  

12. 分区表的存储  
	A partitioned table is made up of one or more table partition segments. If you create a partitioned table named hash_products, then no table segment is allocated for table. Instead, the database stores data for each table partition in its own partition segment. Each table partition segment contains a portion of the table data.  

	Some or all partitions of a heap-organized table can be stored in a compressed format. Compression saves space and can speed query execution. Thus, compression can be useful in environments such as data warehouses, where the amount of insert and update operations is small, and in OLTP environments.  

	The attributes for table compression can be declared for a tablespace, table, or table partition. If declared at the tablespace level, then tables created in tablespace are compressed by default. You can alter compression attribute for a table, in which case the change only applies to new data going into the table. Consequently, a single table or partition may contain compressed and uncompressed blocks, which guarantees that data size will not increase because of compression. If compression could increase the size of a block, then the database does not apply it to the block.  

13. 分区索引和非分区索引分类:  
	1. Nonpartitioned Index 
	2. Partitioned Index 
		1. Local Partitioned Index
			1. Local Prefixed Index
			2. Local Nonprefixed Index
		2. Global Partitioned Index 

14. 分区中的bitmap索引  
	1. Like other indexes, you can create a bitmap index on partitioned tables. The only restriction is that bitmpa indexes must be local to the partitioned table--they cannot be global indexes. Global bitmap indexes are supproted only on nonpartitioned tables. 

15. 本地分区索引的存储  
	Like a table partition, a local index partition is stored in tis own segment. Each segment contains a portion of the total index data. Thus, a local index made up of four partitions is not stored in a single index segment, but in four separate segments.  

16. 全局分区索引  
	A global partitioned index is a B-tree index that is partitioned independently of the underlying table on which it is created. A single index partition can point to any or all table partitions, whereas in a locally partitioned index, a one-to-one parity exists between index partitions and table partitions.  
	In general, global indexes are useful for OLTP applications, where rapid access, data integrity, and availability are important. In an OLTP system, a table may be partitioned by one key, for example, the employees.department_id column, but an application may need to access the data with many different keys, for example, by employee_id or job_id. Global indexes can be useful in this scenario.  

17. 分区索引表特性:  
	1. Partition columns must be a subset of primary key columns.  
	2. Secondary indexes can be partitioned locally and globally.  
	3. OVERFLOW data segments are always equipartitioned with the table partitions.  

	Oracle Database supprots bitmap indexes on partitioned and nonpartitioned index-organized tables. A mapping table is required for creating bitmap indexes on an index-organized table.

18. Object views 
	Like relational views, object views can present only the data that you want users to see. For example, an object view could present data about IT programmers but omit sensitive data about salaries. The following example creates an employee_type object and then the view it_prog_view base on this object:  

	create type employee_type as object
	(
		employee_id number(6),  
		last_name 	varchar2(25),   
		job_id		varchar2(10)
	)  

	create view it_prog_view of employee_type  
		with object identifier(employee_id) as  
	select e.employeeid, e.last_name, e.job_id  
	from employees e  
	where job_id ='IT_PROG';  

	Object views are useful in prototyping or transitioning to object-oriented applications because the data in the view can be taken from relational tables and accessed as if the table were defined as an object table. You can run object-oriented applications without converting existing tables to a different physical structure.  

19. 物化视图  
	A materialized view can be partitioned. You can define a materialized view on a partitioned table and one or more indexes on the materialized view.  
	物化视图如果想要快速刷新（fast refresh），则必须有 materialized view log 才能实现。  

	查询重写对应用来说是透明的。如果一个查询可以通过直接访问物化视图来获得结果集而不用实时关联很多复杂的基表，那么在生成计划的时候，会使用查询重写。将对多个基表的查询转换为查询物化视图。  
	

20. 




