# 我的性能测试经验分享
闲扯：
-----
 * 性能测试门槛较高，选择前还是要谨慎，那么看下一名合格的性能测试工程师要求：
   * 硬件大概信息,当硬件出现性能瓶颈后，提出可供替代硬件方案信息建议
   * 系统框架的优劣势，以及框架下各模块的合理配置优化建议
   * 上面这条好高大上，要完成这样任务的门槛也不低，我们来看下：
     * 编程语言：Java，C，C#，C++，PHP，你不知道以后的工作他们哪个会出现在你的视线中.....
        * Java，php占比较高，目前的主流互联网公司都在用Java，创业公司PHP会多一些
        * C，C#，C++,如果你不是去做银行的外包项目或者游戏公司，这部分一般用不到滴。
     * 服务端: Tomcat,Apache,IIS,Nginx,Mybatis,servlet，这些可不是要求你拼写单词哦～
     * 数据端：Oracle，Mysql，Sqlserver，Postgresql，Mongodb，Redis，Mercache
     * 好吧，这样的性能测试工程师至少我还没有见过，但我认为至少掌握以下硬性技能内容：
        * 性能测试工具的选择，以及掌握编写脚本的语言
        * 公司项目框架基本了解，数据的流转以及存储
        * 项目所用服务配置以及优化方法－－参数辣么多记不住，其实网络上可以找到很多有用的信息，优化还是需要自身积累
        * 服务端所用编程语言需要你本身具备该技能－－当然low点的办法是拉个后台开发人员陪你一起
        * 项目所用数据库的合理配置和优化方法 －－ 本文后面可供查询了解
 * 专职性能还好，如果说你想大包大揽，那么你不仅累，还要维护很多的知识，很痛苦，我还在挣扎中....
 * ####编写已用4小时,分享个bootstrap编写的github主页(记录我的职业生涯中积累的点点滴滴～)：
   * [天空的Github主页](http://linlin547.github.io)

前端性能框架：
-----
 * Selenium+YSlow+ShowSlow实现页面性能评估自动化，还是不错的工具。

1.性能测试划分
-----
  * 配置测试：硬件选型，软件参数配置（数据库，服务器配置）
  * 基准测试：以前版本作为基准
  * 负载测试：找系统瓶颈，评估性能指标，更多的是过程，阶梯式
  * 压力测试：高负载是压力，让系统崩溃，看恢复能力以及时间<br>

2.性能测试前评估
-----
  * 关键业务
  * 日PV量
  * 逻辑复杂度
  * 运营推广
  * 内存和CPU消耗高的业务<br>

3.通常问题
-----
  * 一般性能测试构造数据时不要相同,因为可以避开缓存影响，需要测试缓存可以在处理
  * 明确可用负载机，服务器软硬件信息
  * 明确业务数据流走向，才可以更好的在出现问题时排查问题
  * LR中声明局部变量放在脚本开头
  * LR全局变量在globals.h中声明
  * 网络瓶颈：内网100M，吞吐率不能超过100/8=12.5
  * Ip欺骗时，主机应该单网卡,设置为静态IP,多网卡应禁用
   * ![feature](https://github.com/linlin547/Loadrunner_performance_analysis/blob/master/image/ip.png)
  * Get请求：发送数据少，返回数据多；Post请求：发送数据多，返回数据少，大部分关联都是在post数据中
  * 添加负载机时，要勾选Use the Perc...选项，才可以一个脚本在多个负载机上运行，<br>否则只能同个脚本按组跑，可设置不同负载机ip<br>
  * 当性能指标出现拐点时，排查思路：
    * 负载机硬件资源
    * 网络瓶颈 －－ 观察网络吞吐量计算
    * 服务器硬件资源 －－ cpu，内存，磁盘
    * 数据库服务器硬件资源 －－ cpu，内存，磁盘
    * 服务器的各项参数配置，若分布式需检查配置是否生效，以及请求分配规则是否合理
    * 数据库各项性能指标 －－ 下面有细节分析过程
    * sql反推到代码块
    * 第三方监控工具，监控运行代码；例如：jprofiler(java) xhprof(php)

5.性能测试结果分析以及定位
-----
  * Linux相关信息采集(向运维学习)
    * 闲谈：
      * Linux调优需要知识很庞大，我们并不是运维人员，但可以通过关键指标数据发现直观的问题和提出合理的建议
      * 常用的几个小命令：
        * netstat: 查看端口
        * wc -l: 统计数量
        * grep: 过滤条件
    * top
      * Load Avg:三个数值(主要看系统cpu核数，例如：1核，那么负载值小于等于1都是无压力的，若大于1可能已经出现部分拥堵)
        * 一分钟时负载
        * 二分钟时负载
        * 三分钟时负载
    * iostat -x,系统IO使用监控
      * Cpu
        * %user: 在用户级别运行所使用的CPU的百分比
        * %sys: 在系统级别(kernel)运行所使用CPU的百分比
        * %iowait: CPU等待硬件I/O时,所占用CPU百分比
        * %idle: CPU空闲时间的百分比
      * Device
        * tps: 每秒钟发送到的I/O请求数
        * avgqu-sz: 是平均请求队列的长度,毫无疑问，队列长度越短越好
        * await：每一个IO请求的处理的平均时间（单位是微秒毫秒)
        * rkB/s: 每秒读取数据量
        * wkB/s: 每秒写入数据量
        * %util: 磁盘的繁忙程度，如接近100%那说明磁盘已经到瓶颈
    * vmstat,服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况
      * Cpu
        * r: 运行队列,当这个值超过了CPU数目，就出现CPU瓶颈了
        * us: 用户CPU时间
        * sy: 系统CPU时间，如果太高，表示系统调用时间长
        * id: 空闲 CPU时间，一般来说，id + us + sy = 100
      * Memory
        * swpd: 虚拟内存已使用的大小
          * 如果大于0，表示你的机器物理内存不足，不是程序内存泄露的原因,应该升级硬件了
        * free: 空闲的物理内存的大小
      * Swap
        * si: 每秒从磁盘读入虚拟内存的大小
          * 如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉
        * so: 每秒虚拟内存写入磁盘的大小，如果这个值大于0，同si
      * System
        * in: 每秒CPU的中断次数，包括时间中断
        * cs: 每秒上下文切换次数,线程／进城的切换
          * 该值过大表明CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用
    * Nmon,IBM公司产品，公司不保证准确性的系统监控工具，简单易用，一行命令
      * 这个工具可以满足你对性能监控的大部分数据获取，而且文件数据过大也会丢失数据－－可能我使用方式不对

  * 数据库相关优化信息(向DBA靠近)
    * 闲谈：
      * 数据库很多种，常用也就几种，但作为测试想去做好调优同样不现实，关键性指标是我们提出合理化建议的武器<br>正确的职责定位会让你有更好的进步空间
      * 面试常会被问到而且有意思的sql
        * mysql: select * from 表名 where 条件 order by 字段 limit 1,3(指针从1开始，取3条记录)
        * postgresql: select * from 表名 where 条件 order by 字段 limit 2 OFFSET 0（指针从0开始，取2条记录）
        * sqlserver: select top 10 * from ( select top 20 * from 表明 order by 字段q asc) a order by 字段q desc(取10到19条记录)
        * oracle: select * from (select rownum as rn,id from 表名 where 条件 order by 字段) where rn=行数  ( 或者 rn between n and m (取第n到第m条记录))
      * 数据库通常监控指标：
        * 数据库连接数
        * 慢查询sql
        * 线程池大小
        * 每秒事务数
        * QueryCache命中率
        * 锁定状态
    * Memcache(缓存数据库)
      * 查看运行状态信息: 连接到memcache，运行命令status，查看参数值
        * memcache命中率：get_hits/cmd_get*100% (1.cmd_get:总请求次数；2.get_hits:总命中次数)
    * Redis(缓存数据库)
      * 查看运行状态信息：连接redis，执行命令info，查看参数值
        * Redis命中率：keyspace_hits／(keyspace_hits＋keyspace_misses)*100% (1.keyspace_hits:命中次数；2.keyspace_misses未命中次数)
    * Mysql
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
    * 原来写文档也这么累，先歇歇，[3月31号]更新完后面内容～
    * Postgre
    * Mongo
    * Oracle



