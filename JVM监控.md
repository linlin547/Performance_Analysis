# JVM监控-jprofiler
  * 线程死锁定位-网络知识，已应用
    * Thread Views-Thread History(显示一个与线程活动和线程状态在一起的活动时间表)查看全部线程中blocked的线程
        * 界面如下:
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/线程死锁图.png)
    * Thread Views-Thread Monitor(包括所有的活动线程以及它们目前的活动状况) 查看实时线程状态，根据blocked状态编号线程,点击菜单Thread Dump
        * 界面如下:
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/实时线程状态图.png)
    * 在Thread Dump(显示所有线程的堆栈跟踪)中点击blocked状态线程对象查看
        * 界面如下:
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/线程对象情况.png)
  * 内存泄漏定位-网络知识，已应用
    * 服务器运行过程中，发现内存只升不降；执行gc后，内存也不能被完全释放
    * 初始化检验环境：
        * 切换到“Live Memory-->All Objects”标签，可以看到当前tomcat中的对象情况
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/1.png)
        * 操作前，先F4,运行“Run GC”,使jvm进行内存回收清理无效的对象,为了便于比较内存的增长情况，</br>
        可以点击右键--->"Mark Current",来将当前内存使用情况作为参照；点击后会显示“Difference”列，该列会列出对象数量的变化和变化比率
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/2.png)
    * 打开内存记录： 
        * 点击“Start Recordings”按钮，开始记录。执行这步的主要目的是为下面“Heap Walker”设置一个监控区间；
          如果不记录的话“Heap Walker”将分析jvm虚拟机的所有内存，即耗时又不能准确的发现内存泄漏的原因。
    * 执行压测操作，再次执行gc:
        * 压力工具访问被测应用，执行完之后再次F4进行GC----这样是为了消除可以回收的对象;
          执行内存回收后，仍然存在于内存中的对象有可能是泄漏的对象，instance count中红色的部门为不能回收的对象，
          difference列列出了增加的对象数量,随后会在HeapWalker中观察这些对象，分析哪些对象是泄漏的;
          一般引起泄漏的对象包括：String、char[]、HashMap、Concurrenthashmap等，这类对象需要重点关注.
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/3.png)
    * 关闭内存记录：
        * 点击“Stop Recordings”关闭内存记录，告诉jProfiler把这段记录作为分析对象
    * 找到增加迅速的对象类型，打开HeapWalker：
        * 在视图中找到增长快速的对象类型，本例Concurrenthashmap的增长速度很快。
          在memory视图中找到Concurrenthashmap---点右键----选择“Show Selectiion In Heap Walker”，
          切换到HeapWarker 视图；切换前会弹出选项页面，注意一定要选择“Select recorded  objects”，
          这样Heap Walker会在刚刚的那段记录中进行分析；否则，会分析tomcat的所有内存对象，这样既耗时又不准确；
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/4.png)
    * 在HeapWalker中，找到泄漏的对象:
        * 1.HeapWarker 会分析内存中的所有对象，包括对象的引用、创建、大小和数量 
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/5.jpg)
        * HeapWarker视图下方可以进行页面切换，通过切换到References页签，可以看到这个类的具体对象实例
        * 为了在这些内存对象中，找到泄漏的对象（应该被回收），可以在步骤1对象上点击右键，选择“Use Selected Instances”缩小对象范围</br>
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/6.jpg)
    * 通过引用分析该对象：
        * 在References引用页签中，可以看到该对象的的引用关系，可以切换incoming/outcoming，显示引用的类型：
            * incoming  表示显示这个对象被谁引用；
            * outcoming 表示显示这个对象引用的其他对象；
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/7.jpg)
        * 选择“Show In Graph”将引用关系使用图形方式展现
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/8.jpg)
        * 选中该对象，点击“Show Paths To GC Root”，会找到引用的根节点
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/9.jpg)
        * 上图中这个HashMap Segment对象最终的引用是在ConcurrentHashMap和ReentranLock对象中
    * 通过创建分析该对象：
        * 如果以上步骤还不能定位内存泄露的地方，我们可以尝试使用Allocations页签，
        该页签显示对象是如何创建出来的，我们可以从创建方法开始检查，检查所有用到该对象的地方，直到找到泄漏位置
        ![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/10.jpg)
        
    