# Postgresql数据库
* 查询当前数据库的连接数：
  * select * from pg_stat_activity;
  * 结果集会显示出当前连接的数据库名，用户，IP地址，连接开始时间，查询的语句

* pg_stat_statements统计了SQL的很多信息，但postgre并没有为我门创建，
    方便我们分析SQL的性能，需要修改postgresql.conf<br>
  * 操作步骤如下
    * 修改配置文件，并且重启PostgreSQL方能生效：
      * PG_STAT_STATEMENTS OPTIONS
        * shared_preload_libraries = 'pg_stat_statements'
        * custom_variable_classes = 'pg_stat_statements'
        * pg_stat_statements.max = 1000
        * pg_stat_statements.track = all<br>
  * 创建pg_stat_statements扩展：
      * CREATE EXTENSION pg_stat_statements;从此之后，PostgreSQL就能记录SQL的统计信息。
      * 配置：![feature](https://github.com/linlin547/Loadrunner_performance_analysis/blob/master/image/statements.png)
* 慢查询：
  * SELECT  query, calls, total_time, (total_time/calls) as average ,rows,
   100.0 * shared_blks_hit /nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent 
   FROM  pg_stat_statements 
   ORDER BY average DESC LIMIT 10;
     * query:慢查询sql
     * calls：访问次数
     * total_time：运行时间，版本大于9.2的单位毫秒
     * hit_percent:命中率
     * 这个结果会一直增加，如果不想这样，那么可以在每次监控时先清理一下，执行SQL：select pg_stat_statements_reset()
* 查询表中存在的锁
  * select a.locktype,a.database,a.pid,a.mode,a.relation,b.relname
   from pg_locks a
   join pg_class b on a.relation = b.oid
   where upper(b.relname) = 'TABLE_NAME';
   * 以上为查询某表上是否存在锁的SQL语句，查到后发现确实存在锁，如下：<br>
     locktype | database |  pid  |      mode      | relation | relname<br>
    ----------+----------+-------+-----------------+----------+---------<br>
     relation |  439791 | 26752 | AccessShareLock |  2851428 |table_name<br>
   * 根据上面查出来的pid去表pg_stat_activity查询一下该锁对应的SQL语句：
     * select usename,current_query ,query_start,procpid,client_addr from pg_stat_activity where procpid=17509;
   * 如下：<br>
      usename  |  current_query   |   query_start   | procpid |  client_addr<br>
       -----------+--------------------------------------------------------------------------------------------<br>
      gpcluster | DELETE FROM TABLE_NAME WHERE A = 1  | 2011-05-14 09:35:47.721173+08 |  17509 | 192.168.165.18<br>

   * 通过以上可以发现，就是上面的锁导致该语句一直挂在那里,只需要和技术人员确认下这个SQL就好啦
   
   * 索引问题：<br>
   * 一个比较厉害的SQL，可以查询出表名，使用索引的百分比，读取该表的行数：
   * SELECT relname,
       100 * idx_scan / (seq_scan + idx_scan) percent_of_times_index_used, 
       n_live_tup rows_in_table
       FROM 
         pg_stat_user_tables
       WHERE 
           seq_scan + idx_scan > 0 
       ORDER BY 
         n_live_tup DESC;<br>
   * 索引缓存命中率：
     * SELECT
       sum(idx_blks_read) as idx_read,
       sum(idx_blks_hit)  as idx_hit,
       (sum(idx_blks_hit) - sum(idx_blks_read)) / sum(idx_blks_hit) as ratio
     FROM 
       pg_statio_user_indexes;<br>
     * 一般来说，你应该要求这个达到99%，和你一般缓存命中率一样


