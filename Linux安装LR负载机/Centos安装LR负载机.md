* 以下安装步骤兼容centos6和centos7
	* 1.上传LR负载机到centos
	* 2.解压负载机
	```
		unzip Linux.zip
	```
	* 3.将解压后Linux目录下所有文件赋权限(否则无法执行安装文件)
	```
		chmod -R 777 Linux/*
	```
	* 4.进入Linux目录，执行安装(root用户)
	```
		1.cd Linux
		2.sh installer.sh
	```		
	```
		4.1.输入n执行下一步		
		4.2.输入a同意		
		4.3.输入i安装		
		4.4.输入f安装完成
	```
	* 5.创建LR运行用户(root用户)
	```
		创建用户：useradd -g 0 -s /bin/bash lr_test
		添加用户密码：passwd lr_test
		注意：这里用户名可以自行定义，使用bash或csh也都可以，只不过配置略有不同，以下以bash配置方式为例
	```
	* 6.创建环境变量(root用户)
	```
		vim /opt/HP/HP_LoadGenerator/env.sh
	```
	```
		添加内容如下：
		#/bin/bash
		export PRODUCT_DIR=/opt/HP/HP_LoadGenerator
		export M_LROOT=$PRODUCT_DIR
		export LD_LIBRARY_PATH=$M_LROOT/bin:$M_LROOT/lib:/usr/lib:/usr/lib64
		export DISPLAY='0.0'
		export PATH=$PATH:$M_LROOT/bin
	```
	![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/envsh.png)
	* 7.配置文件加载到/etc/profile中，以便开机、切换用户时都能自动加载(root用户)
	```
		vim /etc/profile
	```
	```
		添加内容如下：
		source /opt/HP/HP_LoadGenerator/env.sh
	```
	```
		保存后root用户执行：source /etc/profile
	```
	![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/etc-profile.png)
	* 8.切换为lr_test用户查看环境变量
	```
		1.命令：su lr_test
		2.命令：env
	```
	![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/env_done.png)
	* 9.启动负载机(lr_test用户)
	```
		1.命令：cd /opt/HP/HP_LoadGenerator/bin 
		2.命令：./m_daemon_setup start
	```
	```
		运行问题1:
			./m_daemon_setup: ./m_agent_daemon: /lib/ld-linux.so.2: bad ELF interpreter: 没有那个文件或目录
		解决方案：
			1.切换为root用户
			2.执行：yum install glibc.i686 
	```
	```
		运行问题2:
			m_agent_daemon: error while loading shared libraries: libstdc++.so.5: cannot open shared object file: No such file or directory
		解决方案：
			1.切换为root用户
			2.执行yum whatprovides libstdc++.so.5 (查看有哪些可用安装包)
			3.根据提示结果(我的)系统可用为compat-libstdc++-33-3.2.3-69.el6.i686
			4.执行：yum install compat-libstdc++-33-3.2.3-69.el6.i686
	```
	```
		启动成功：
			[lr_test@localhost bin]$ ./m_daemon_setup start
			 m_agent_daemon ( 3497 ) ---会提示有进程号3497
		查看已启动的进程：
			ps -ef | grep m_agent_daemon
	```
	```
		启动失败：
			[lr_test@localhost bin]$ ./m_daemon_setup start
			 m_agent_daemon ( is down ) ---会提示 is down
		解决方案：
			1.lr_test用户执行：env | grep HOSTNAME
				结果：HOSTNAME=localhost.localdomain
			2.root用户执行：vim /etc/hosts
				127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
				注意：必须确保第一步查到的hostname在127.0.0.1后面才可以，这里localhost.localdomain存在
	```
	* 10.关闭防火墙
	```
		centos6(root用户):
			1.执行：service iptables stop (关闭防火墙)
			2.执行：setenforce 0 (关闭selinux)
	```
	```
		centos7(root用户):
			1.执行：systemctl stop firewalld (关闭防火墙)
			2.执行：setenforce 0 (关闭selinux)
	```
	* 11.Loadrunner controller添加linux负载机
		* 11.1.添加Linux负载机
		![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/负载机1.png)
		* 11.2.修改添加linux负载机配置
		![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/负载机2.png)
		* 11.3.连接linux负载机
		![feature](https://github.com/linlin547/Performance_Analysis/blob/master/image/负载机3.png)
