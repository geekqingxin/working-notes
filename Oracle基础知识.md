## 高水位线和低高水位线

[High water mark on a table](https://asktom.oracle.com/pls/asktom/f?p=100:11:0::::P11_QUESTION_ID:492636200346818072)

__高水位线__用来标识段中使用最大数据块的位置。高水位线上是未使用的数据块。在高水位线下的数据块上执行delete操作后，这些数据块将处于空闲状态，但高水位线不会随之下降，直到重建，截断或收缩表段。

全表扫描会扫描高水位线之下的所有块，包括空闲数据块(执行了delete操作)。这将会影响全表扫描的性能，尤其是在位于高水位线以下大部分块空闲的情况时。

在MSSM(Manual segment space management)表空间中，segment有明确的高水位线(High water mark)。

在ASSM(Auto Segment space management)表空间中，包含HWM和low HWM。

在MSSM中，当HWM上涨时（如，插入时），所有的块被格式化并且有效，Oracle可以安全的读取他们。

在ASSM中，当HWM上涨时，Oracle不立即格式化所有的块。直到他们在第一次被使用时，才会被格式化和安全读取。因此，当全扫描segment时，我们需要知道块是否可以被安全的读取，是不是没有被格式化(意味着他们没有包含任何有效数据，并且我们不需要处理它们)。为了达到这个目的，不是所有的块都要做安全/非安全检查，Oracle维护了一个low HWM和一个HWM。Oracle将全扫描表直到HWM，和low HWM下所有的数据块，只读取和处理这些数据。对于low HWM和HWM之间的块，他必须仔细并参考ASSM位图信息来管理这些数据块，来决定那些块应该读取，那些块应该忽略。

### 演示高水位线与全表扫描

_请在SQL plus中执行_

#### 创建测试表


	create table t_high_water_mark as 
	select rownum as id,
	       round(dbms_random.normal * 1000) as content,
	       dbms_random.string('p', 250) as pad
	  from dual
	connect by level <= 10000;

#### 将以下内容存入tab_stat.sql文件中

	set linesize 250
	select num_rows,
	       blocks blks,
	       empty_blocks em_blks,
	       avg_space,
	       chain_cnt,
	       avg_row_len,
	       round(num_rows / blocks) AS avg_rows_per_block,
	       last_analyzed lst_anly,
	       stale_stats
	  from dba_tab_statistics
	 where table_name = upper('&input_table_name')
	   and owner = upper('&owner');

#### 收集信息
	
	exec dbms_stats.gather_table_stats('LIUYF','T_HIGH_WATER_MARK',cascade=>true);

	analyze table T_HIGH_WATER_MARK compute statistics;

#### 执行@tab_stat.sql，输入表名和Owner


	  NUM_ROWS       BLKS    EM_BLKS  AVG_SPACE  CHAIN_CNT AVG_ROW_LEN AVG_ROWS_PER_BLOCK LST_ANLY        STALE_STA
	---------- ---------- ---------- ---------- ---------- ----------- ------------------ ------------------ ---------
		10000        387        125        919          0         262                 26 07-APR-17       NO


_表空闲块为：125_

#### 查看段上的块信息

	col segment_name format a50

查看段上的块信息

	select segment_name, segment_type, blocks, extents
		from dba_segments
	where segment_name = 'T_HIGH_WATER_MARK'
		and owner = 'LIUYF';

结果展示

	SEGMENT_NAME         SEGMENT_TYPE                                               BLOCKS    EXTENTS
	-------------------- ------------------------------------------------------ ---------- ----------
	T_HIGH_WATER_MARK    TABLE                                                         512         19

_数据块为：512_

#### 开户Autotrace only

	set autotrace traceonly;
	select count(*) from t_high_water_mark;
	
结果展示

	Execution Plan
	----------------------------------------------------------
	Plan hash value: 3543783079
	
	--------------------------------------------------------------------------------
	| Id  | Operation          | Name              | Rows  | Cost (%CPU)| Time     |
	--------------------------------------------------------------------------------
	|   0 | SELECT STATEMENT   |                   |     1 |   107   (0)| 00:00:02 |
	|   1 |  SORT AGGREGATE    |                   |     1 |            |          |
	|   2 |   TABLE ACCESS FULL| T_HIGH_WATER_MARK | 10000 |   107   (0)| 00:00:02 |
	--------------------------------------------------------------------------------
	
	
	Statistics
	----------------------------------------------------------
	          1  recursive calls
	          0  db block gets
	        374  consistent gets
	          0  physical reads
	          0  redo size
	        526  bytes sent via SQL*Net to client
	        523  bytes received via SQL*Net from client
	          2  SQL*Net roundtrips to/from client
	          0  sorts (memory)
	          0  sorts (disk)
	          1  rows processed

#### 删除部分数据

	set autotrace off;

	delete from t_high_water_mark where rownum <= 9900;

	commit;

#### 收集信息

	exec dbms_stats.gather_table_stats('LIUYF','T_HIGH_WATER_MARK',cascade=>true);
	
	analyze table T_HIGH_WATER_MARK compute statistics;

	@tab_stat.sql	-- 输入表名和owner
		
结果展示

	  NUM_ROWS       BLKS    EM_BLKS  AVG_SPACE  CHAIN_CNT AVG_ROW_LEN AVG_ROWS_PER_BLOCK LST_ANLY        STALE_STA
	---------- ---------- ---------- ---------- ---------- ----------- ------------------ ------------------ ---------
       100        387        125       7921          0         262                  0 07-APR-17       NO

#### 查看执行计划

	set autotrace traceonly;

	select count(*) from t_high_water_mark;

结果展示

	Execution Plan
	----------------------------------------------------------
	Plan hash value: 3543783079
	
	--------------------------------------------------------------------------------
	| Id  | Operation          | Name              | Rows  | Cost (%CPU)| Time     |
	--------------------------------------------------------------------------------
	|   0 | SELECT STATEMENT   |                   |     1 |   107   (0)| 00:00:02 |
	|   1 |  SORT AGGREGATE    |                   |     1 |            |          |
	|   2 |   TABLE ACCESS FULL| T_HIGH_WATER_MARK |   100 |   107   (0)| 00:00:02 |
	--------------------------------------------------------------------------------
	
	
	Statistics
	----------------------------------------------------------
	          0  recursive calls
	          0  db block gets
	        374  consistent gets
	          0  physical reads
	          0  redo size
	        526  bytes sent via SQL*Net to client
	        523  bytes received via SQL*Net from client
	          2  SQL*Net roundtrips to/from client
	          0  sorts (memory)
	          0  sorts (disk)
	          1  rows processed

#### 关闭测试计划

	set autotrace off;

#### 执行表收缩，再次查看测试计划

	alter table t_high_water_mark enable row movement;
	alter table t_high_water_mark shrink space cascade;
	alter table t_high_water_mark disable row movement;

	exec dbms_stats.gather_table_stats('LIUYF','T_HIGH_WATER_MARK');
	
	analyze table T_HIGH_WATER_MARK compute statistics;

	@tab_stat.sql	-- 输入表名和owner

结果展示

	  NUM_ROWS       BLKS    EM_BLKS  AVG_SPACE  CHAIN_CNT AVG_ROW_LEN AVG_ROWS_PER_BLOCK LST_ANLY        STALE_STA
	---------- ---------- ---------- ---------- ---------- ----------- ------------------ ------------------ ---------
       100          4          4       1435          0         262                 25 07-APR-17       NO

_已使用的块为：4，空闲的块为：4_

##### 执行测试计划，查看结果
	
	set autotrace traceonly;
	select count(*) from t_high_water_mark;

结果展示

	Execution Plan
	----------------------------------------------------------
	Plan hash value: 3543783079
	
	--------------------------------------------------------------------------------
	| Id  | Operation          | Name              | Rows  | Cost (%CPU)| Time     |
	--------------------------------------------------------------------------------
	|   0 | SELECT STATEMENT   |                   |     1 |     3   (0)| 00:00:01 |
	|   1 |  SORT AGGREGATE    |                   |     1 |            |          |
	|   2 |   TABLE ACCESS FULL| T_HIGH_WATER_MARK |   100 |     3   (0)| 00:00:01 |
	--------------------------------------------------------------------------------
	
	
	Statistics
	----------------------------------------------------------
	          1  recursive calls
	          0  db block gets
	          5  consistent gets
	          0  physical reads
	          0  redo size
	        526  bytes sent via SQL*Net to client
	        523  bytes received via SQL*Net from client
	          2  SQL*Net roundtrips to/from client
	          0  sorts (memory)
	          0  sorts (disk)
	          1  rows processed

_一致性读从374变为5_

#### 执行truncate，再次查看测试计划

	set autotrace off;

	truncate table t_high_water_mark;

	exec dbms_stats.gather_table_stats('LIUYF','T_HIGH_WATER_MARK');

	set autotrace traceonly;

	select count(*) from t_high_water_mark;

结果展示

	Execution Plan
	----------------------------------------------------------
	Plan hash value: 3543783079
	
	--------------------------------------------------------------------------------
	| Id  | Operation          | Name              | Rows  | Cost (%CPU)| Time     |
	--------------------------------------------------------------------------------
	|   0 | SELECT STATEMENT   |                   |     1 |     2   (0)| 00:00:01 |
	|   1 |  SORT AGGREGATE    |                   |     1 |            |          |
	|   2 |   TABLE ACCESS FULL| T_HIGH_WATER_MARK |     1 |     2   (0)| 00:00:01 |
	--------------------------------------------------------------------------------
	
	
	Statistics
	----------------------------------------------------------
	          5  recursive calls
	          0  db block gets
	          5  consistent gets
	          0  physical reads
	          0  redo size
	        525  bytes sent via SQL*Net to client
	        523  bytes received via SQL*Net from client
	          2  SQL*Net roundtrips to/from client
	          0  sorts (memory)
	          0  sorts (disk)
	          1  rows processed

_一致性读由5变成3_


