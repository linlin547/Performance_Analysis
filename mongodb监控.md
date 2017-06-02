# Mongodb数据库
  * mongodb连接数：db.serverStatus().connections
    * {"current" : 2581, //当前连接数<br>
      "available" : 48619, //可用连接数<br>
        "totalCreated" :NumberLong(187993238) //截止目前为止总共创建的连接数}<br>
        可看到当前mongod的最大连接数即为51200=2581+48619<br>
  * 索引统计信息：db.serverStatus().indexCounters
    * {"accesses" : 35369670951, //索引访问次数，值越大表示你的索引总体而言建得越好，如果值增长很慢，表示系统建的索引有问题<br>
        "hits" : 35369213426, //索引命中次数，值越大表示mogond越好地利用了索引<br>
        "misses" : 0, //表示mongod试图使用索引时发现其不在内存的次数，越小越好<br>
        "resets" : 0, //计数器重置的次数<br>
        "missRatio" : 0 //丢失率，即misses除以hits的值}<br>
  * 慢查询开启和查询：
    * 开启慢查询：
      * Profiling级别：0：关闭，不收集任何数据 1：收集慢查询数据，默认是100毫秒 2：收集所有数据
      * 查看状态：db.getProfilingStatus()
        * 返回结果：{ "was" : 1, "slowms" : 200 }
      * 设置级别和时间： db.setProfilingLevel(1,200)
    * 查询慢查询语句
      * 返回最近的10条记录：db.system.profile.find().limit(10).sort({ ts : -1 }).pretty()
    * 慢查询语句分析：db.库名.find(查询出的慢查询语句).explain()
      * 返回key：
        * queryPlanner：查询计划的选择器，首先进行查询分析，最终选择一个winningPlan
        * executionStats：为执行统计层面返回winningPlan的统计结果
        * 查询优化的时候，只需要关注queryPlanner， executionStats即可
        * 3.2以后版本关注stage，是否需要为语句增加索引：
          * COLLSCAN ：全表扫描
          * IXSCAN：索引扫描
          * FETCH:：根据索引去检索指定document
  * Mongodb有4种锁：r：某个数据库读锁,R：全局读锁,w：某个数据库写锁,W：全局写锁
      * 全局锁信息：db.serverStatus().globalLock  
        * {"totalTime" :NumberLong("172059990000"), //mongod启动后到现在的总时间，单位微秒<br>
            "lockTime" :NumberLong(2031058), //mongod启动后全局锁锁住的总时间，单位微秒<br>
            "currentQueue" : {<br>
                "total" : 0, //当前的全局锁等待锁等待的个数<br>
                "readers" : 0, //当前的全局读锁等待个数<br>
                   "writers" : 0 //当前全局写锁等待个数}<br>
      * 某个库的锁信息：db.serverStatus().locks
  * mongo自带的监控工具：进入mongoDB的bin目录执行：mongostat 
    * 查询结果：![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/mongod_stat.png)
    * 详情解释：![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/mongodbstate.png)
      
    