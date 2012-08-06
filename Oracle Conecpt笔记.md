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

