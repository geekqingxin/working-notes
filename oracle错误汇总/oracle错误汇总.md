### 1. 错误信息如下

    EXP-00056: ORACLE error 604 encountered
    ORA-00604: error occurred at recursive SQL level 1
    ORA-01653: unable to extend table SYS.AUD$ by 128 in tablespace SYSTEM
    ORA-02002: error while writing to audit trail
    ORA-00604: error occurred at recursive SQL level 1
    ORA-01653: unable to extend table SYS.AUD$ by 128 in tablespace SYSTEM

__原因：__Oracle system auditing is turned on (This is the default in Oracle 11-up).

__解决办法：__

##### 1. 方案1

###### 1.1 以sysdba登录

    export ORACLE_SID=JDBDEVDB

    sqlplus / as sysdba
###### 1.2 核对连接上正确的数据库实例

    select name,log_mode from v$database;

    select instance_name from v$instance;

###### 1.3 验证系统空间耗尽

    set pagesize 500
    
    select a.tablespace_name,
        (select sum(b.bytes) / 1024 / 1024 / 1024
            from dba_data_files b
            where a.tablespace_name = b.tablespace_name) as GB_TOT,
        (select sum(c.bytes) / 1024 / 1024 / 1024
            from dba_free_space c
            where a.tablespace_name = c.tablespace_name) as GB_AVAIL,
        100 * (select sum(c.bytes)
            from dba_free_space c
            where a.tablespace_name = c.tablespace_name) /
        (select sum(b.bytes)
            from dba_data_files b
            where a.tablespace_name = b.tablespace_name) as PERC_FREE
    from dba_tablespaces a
    where tablespace_name = 'SYSTEM';

###### 1.4 上面的查询显示system表空间是被用完了。在这篇文章中，我们将清除sys.aud$表来解决这个问题。另外，我们也可以增加system表空间，来解决这个问题。

###### 1.5 验证aud$确实消耗了大量的空间。

    select sum(bytes) / 1024 / 1024
    from dba_segments
    where SEGMENT_NAME = 'AUD$';

###### 1.6 清除aud$

    truncate table sys.aud$;
    commit;

###### 1.7 验证SYSTEM表空间增加了

    set pagesize 500
    
    select a.tablespace_name,
       (select sum(b.bytes) / 1024 / 1024 / 1024
          from dba_data_files b
         where a.tablespace_name = b.tablespace_name) as GB_TOT,
       (select sum(c.bytes) / 1024 / 1024 / 1024
          from dba_free_space c
         where a.tablespace_name = c.tablespace_name) as GB_AVAIL,
       100 * (select sum(c.bytes)
                from dba_free_space c
               where a.tablespace_name = c.tablespace_name) /
       (select sum(b.bytes)
          from dba_data_files b
         where a.tablespace_name = b.tablespace_name) as PERC_FREE
    from dba_tablespaces a
    where tablespace_name = 'SYSTEM';

##### 2. 方案2

###### 2.1 前4步，与方案1一样

###### 2.2 查询SYSTEM表空间，数据文件

    select file_name, bytes/1024/1024
    from dba_data_files
    where tablespace_name='SYSTEM';

###### 2.3 查看剩余表空间

    select tablespace_name,sum(bytes)/1024/1024
    from dba_free_space
    group by tablespace_name;

###### 2.4 查看数据文件是否自动扩展

    SELECT FILE_NAME, TABLESPACE_NAME, AUTOEXTENSIBLE
    FROM dba_data_files
    where tablespace_name='SYSAUX';

    SELECT FILE_NAME, TABLESPACE_NAME, AUTOEXTENSIBLE
    FROM dba_data_files
    where tablespace_name='SYSAUX';

###### 2.5 开启表空间自动扩展

    # SYSTEM
    alter database datafile '/oradata/jdbdevdb_new/system01.dbf' autoextend on;

    # SYSAUX
    alter database datafile '/oradata/jdbdevdb_new/sysaux01.dbf' autoextend on;

__如表空间已开启自动扩展，且文件已满。需要添加新文件。__

    alter tablespace SYSTEM add datafile '/oradata/jdbdevdb_new/system02.dbf' size 1000m;

###### 2.6 执行2.3和2.4，验证空间剩余情况
    
