* Mysql监控
    * mysql两种引擎：
      * MyISAM：表锁－－一般不会在使用表锁
      * InnoDB：行锁－－常用
    * 表索引查看命令：desc 表名
    * 查看设置：show variables like...
    * 查看状态：show global status like...
    * 慢查询设置：show variables like '%slow%';
      * 需要开启慢查询日志，因慢查询日志耗费性能，建议独立一台机器写慢查询log
      * 设置慢查询时间，一般在5s内即可
      * 设置慢查询log位置
    * 慢查询语句：show global status like '%slow%';
    * 展示大于查询时间的sql数量
    * 根据慢查询可以判断是索引问题还是sql语句编写问题
    * mysql最大连接数配置信息：show variables like 'max_connections';
    * mysql当前连接数： show global status like 'Max_used_connections';
      * 例如：最大连接数10个，每秒请求数据库连接数确是100个，可想不是很合理
    * key_buffer_size索引缓存命中率(内存中)
      * key_buffer_size是对MyISAM引擎性能影响最大的一个参数
        * 查询配置：show variables like 'key_buffer_size';
        * 查询当前读状态：show global status like 'key_read%';
        * 查询当前写状态：show global status like 'key_write%';
          * 返回参数Key_read_requests：读索引请求总数
          * 返回参数Key_reads：未从内存中找到，从磁盘读取的信息
          * 返回参数Key_write_requests：写索引请求总数
          * 返回参数Key_writes：写如磁盘的请求
          * 命中率为：
            * 读命中率：key_buffer_read_hits=(1-Key_reads/Key_read_requests)*100%
            * 写命中率：key_buffer_write_hits=(1-Key_writes/Key_write_requests)*100%
    * InnodbBuffer缓存命中率(内存中)
      * 缓存Innodb类型表的数据和索引的内存空间
      * InnodbBuffer是对InnoDB引擎性能影响最大的一个参数
      * 查询当前读状态：show status like 'Innodb_buffer_pool_read%';
      * 查询当前写状态：show status like 'Innodb_buffer_pool_write%';
        *命中率：
          * 读命中率：innodb_buffer_read_hits=(1-Innodb_buffer_pool_reads/Innodb_buffer_pool_read_requests)*100%
          * 写命中率：......
    * 查询线程池设置：show variables like 'thread%';
      * 线程池在短链接中功效更明显，频繁的生成和销毁连接
      * thread_cache_size:线程池中应该存放的连接线程数
        * 启动时，mysql不会全部创建，随着请求线程使用完，被释放存到线程池中，当线程池达到最大值，不在放入线程池
      * thread_stack：初始化每个线程分配内存大小
      * 查询配置：show variables like 'thread%';
      * 系统被连接的次数：show status like 'Connections';
        * 连接总数：Connections
      * 查询状态：show status like '%thread%';
        * Threads_created:创建过的线程，若数值较大，说明配置的thread_cache_size太小不合理，需要增加
        * Threads_cached: 线程池中缓存的线程
        * Thread Cache命中率(90%以上最优): Threads_Cache_Hit=(Connections-Threads_created)/Connections*100%
    * 行锁
      * 查询状态：show status like '%lock%';
        * Innodb_row_lock_current_waits：当前等待锁的数量,阻塞数量
        * Innodb_row_lock_time_avg：每次平均锁定的时间
        * Innodb_row_lock_time_max：最长一次锁定时间
        * Innodb_row_lock_waits：系统启动到现在、总共锁定次数
      * 查询最近一次死锁：show engine innodb status;
