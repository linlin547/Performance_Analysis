# 性能测试经验之谈
#性能测试注意点，性能分析相关信息
前端性能框架：
-----
 ＊ Selenium+YSlow+ShowSlow实现页面性能评估自动化，还是不错的工具。
 
1.性能测试前评估
-----
  * 关键业务
  * 日PV量
  * 逻辑复杂度
  * 运营推广
  * 内存和CPU消耗高的业务
  * Then （那么）
  * Environment （环境变量）
  

2.通常问题
-----
  * ![feature](https://github.com/linlin547/Python_BDD_behave/blob/master/image/dir.png)
  * 一般性能测试构造数据时不要相同,因为可以避开缓存影响，需要测试缓存可以在处理。
  * LR中声明局部变量放在脚本开头
  * LR全局变量在globals.h中声明

3.behave 示例
-----
  * 新建Behave_pro目录
  * 新建/Behave_pro/features
  * 新建/Behave_pro/features/steps
  * 新建/Behave_pro/features/test.feature 场景描述
    * ![feature](https://github.com/linlin547/Python_BDD_behave/blob/master/image/feature.png)
  * 新建/Behave_pro/features/environment.py 前后依赖函数,类似setup初始化
    * ![feature](https://github.com/linlin547/Python_BDD_behave/blob/master/image/env.png)
  * 新建/Behave_pro/features/steps/test.py 测试步骤文件,对应test.feature中场景
    * ![step](https://github.com/linlin547/Python_BDD_behave/blob/master/image/step.png)

4.behave 执行
-----
  * 进入Behave_pro目录,输入 behave,运行结果
    * ![result](https://github.com/linlin547/Python_BDD_behave/blob/master/image/result.png)

  * 运行结果可以看出,执行了多个场景,当出现失败时,会展示红色字体,标记失败场景

5.参考站点:
-----
  * T先生的博客:
    * http://www.cnblogs.com/tman/p/4115795.html
  * 卡农Lucas的博客
    * http://www.cnblogs.com/devtesters/p/4368318.html
  * 官方Demo
    * http://jenisys.github.io/behave.example/
