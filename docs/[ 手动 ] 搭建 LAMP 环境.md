## LAMP 环境
### 环境准备
```bash

	操作系统:       CentOS 7.9.2009 64位 
	Linux内核版本:  3.10.0-1160.59.1
	Apache版本:    2.4.53
	MySQL版本:     5.7.37
	PHP版本:       7.4.28
	
```

#### 系统环境
```bash

	关闭 SELinux 和 firewalld 防火墙 # 测试机
	
	# firewalld 停止服务 并 禁止开机启动 ----------------------------------------------|
	[root@localhost ~]$ systemctl stop firewalld      
	[root@localhost ~]$ systemctl disable firewalld   
	
	# SELinux 停止服务 并 禁止开机启动 ------------------------------------------------|
	[root@localhost ~]$ setenforce 0 
	[root@localhost ~]$ sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config 
	[root@localhost ~]$ reboot
	[root@localhost ~]$ getenforce

```

##### 关闭相关服务
为了防止 rpm 安装的软件和接下来安装的源码软件包冲突,停止相同服务
```bash
	[root@localhost ~]$ systemctl stop httpd  && systemctl disable httpd
	[root@localhost ~]$ systemctl stop mysqld && systemctl disable mysqld
```

##### LAMP关系图谱
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201103224247356.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3c5MTg1ODk4NTk=,size_16,color_FFFFFF,t_70#pic_center)

##### 安装编译工具
```bash

	# 编译工具 -----------------------------------------------------------------------|
		# 基础编译工具
			yum -y install gcc gcc-c++ make cmake zip bzip2 openssl 		
		# 通用库支持脚本 # jpeg6 软件所依赖
			yum -y install libtool openssl-devel		
		# Apache依赖库
			yum -y install expat-devel libxml2-devel pcre-devel					
		# MySQL依赖库
			yum -y install libaio-devel libtirpc-devel bison-devel glibc-devel ncurses-devel ncurses-libs 
		# PHP依赖库 
			yum -y install libxml2-devel libxml2 libiconv # PHP 安装依赖库需要根据编译设置进行调整,参考下文
	
	# 一键安装 -----------------------------------------------------------------------|
	yum -y install gcc gcc-c++ make cmake zip bzip2 libtool* openssl openssl-devel expat-devel libxml2-devel pcre-devel	libaio-devel libtirpc-devel bison-devel glibc-devel ncurses-devel ncurses-libs libxml2-devel libxml2 libiconv 
	
```

##### 相关软件包下载
```bash

	'Apache 及其依赖' 
	# -----------------------------------------------------------------------|
		# Apache
			wget https://dlcdn.apache.org/httpd/httpd-2.4.53.tar.gz
		# apr
			wget https://dlcdn.apache.org//apr/apr-1.7.0.tar.gz		
		# apr-util
			wget https://dlcdn.apache.org//apr/apr-util-1.6.1.tar.gz		

	'MySQL 及其依赖'
	# -----------------------------------------------------------------------|
		# MySQL 
			wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.37.tar.gz 		
		# ncurses
			wget https://invisible-mirror.net/archives/ncurses/ncurses-6.3.tar.gz
		# boost
			wget http://www.sourceforge.net/projects/boost/files/boost/1.59.0/boost_1_59_0.tar.gz
		# rpcsvc-proto
			wget https://github.com/thkukuk/rpcsvc-proto/releases/download/v1.4/rpcsvc-proto-1.4.tar.gz

	'PHP 及其依赖'
	# -----------------------------------------------------------------------|
		# php
			wget https://www.php.net/distributions/php-7.4.28.tar.gz
		# libxml2
			wget http://xmlsoft.org/sources/libxml2-2.9.11.tar.gz
		# libmcrypt
			wget https://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz
		# mhash
			wget https://sourceforge.net/projects/mhash/files/mhash/0.9.9.9/mhash-0.9.9.9.tar.gz
		# mcrypt
			wget https://sourceforge.net/projects/mcrypt/files/MCrypt/2.6.8/mcrypt-2.6.8.tar.gz
		# zlib # apache & php 共同依赖
			wget https://www.zlib.net/fossils/zlib-1.2.8.tar.gz
			# wget https://zlib.net/zlib-1.2.12.tar.gz 
		# libpng # 1.6.37 与 zlib 有版本冲突问题
			wget https://jaist.dl.sourceforge.net/project/libpng/libpng16/1.6.37/libpng-1.6.21.tar.gz
			wget https://jaist.dl.sourceforge.net/project/libpng/libpng16/1.6.37/libpng-1.6.37.tar.gz
		# jpeg6
			wget https://sourceforge.net/projects/libjpeg/files/libjpeg/6b/jpegsrc.v6b.tar.gz
		# freetype
			wget https://sourceforge.net/projects/freetype/files/freetype2/2.3.5/freetype-2.3.5.tar.gz

	'一键解压缩脚本'
	# -----------------------------------------------------------------------|
        [root@localhost ~]$ vim tar.sh 
        #!/bin/bash
        cd /opt
        /bin/ls *.tar.gz > ls.list
        	for TAR in $(cat ls.list)
        		do
             	/bin/tar -xf $TAR
             	done
        /bin/rm ls.list
        unzip pecl-memcache-php7.zip > /dev/null 
        echo "ok"

```

源码软件包安装报错确认与解决方案
>```bash
 echo $?     # 安装软件过程中由于频繁刷屏，建议在每个步骤结束后执行此命令
 ./configure # 此步骤报错多是依赖关系没解决或是编译工具未安装（注意关键词提示）
 make        # 此步骤多是编译时选项参数书写错误、不存在、漏写等问题
>		     # 一般需要检查上一个步骤：./configure --help
>```
---
#### 安装依赖

> ​	主要目的 : 为三大件编译安装提供支持
>
> ​	<!-- 此次搭建 LAMP 环境大部分依赖内容,阿里centos7-yum源以更新并支持,以下依赖包安装,作为掌握理解手动搭配环境所要理解掌握. -->

##### 安装 libxml2
>​	**Libxml2 是一个 xml c 语言版的解析器**，本来是为 Gnome 项目开发的工具，是一个基于 MIT License 的免费开源软件。它除了支持 c 语言版以外，还支持 c++、PHP、Pascal、Ruby、Tcl 等语言的绑定，能在 Windows、Linux、Solaris、MacOsX 等平台上运行。功能还是相当强大的，相信满足一般用户需求没有任何问题。

```bash

	# [root@localhost ~]$ yum install -y libxml2-devel python-devel # 以一键安装
	[root@localhost ~]$ cd /opt/libxml2-2.9.11
	[root@localhost ~]$ ./configure --prefix=/usr/local/rely/libxml2
	[root@localhost ~]$ make && make install 

```

##### 安装 libmcrypt
>​	**libmcrypt 是加密算法扩展库**。支持 DES, 3DES, RIJNDAEL, Twofish, IDEA, GOST, CAST-256, ARCFOUR, SERPENT, SAFER+等算法。

```bash
	
	[root@localhost ~]$ cd /opt/libmcrypt-2.5.8
	[root@localhost ~]$ ./configure --prefix=/usr/local/rely/libmcrypt
	[root@localhost ~]$ make && make install
	# 安装 libltdl，也在 libmcrypt 源码目录中，非新软件
	[root@localhost ~]$ cd /opt/libmcrypt-2.5.8/libltdl	
	[root@localhost ~]$ ./configure --enable-ltdl-install && make && make install	
	
```

##### 安装 mhash
>​	**mhash 是基于离散数学原理的不可逆向的 php 加密方式扩展库**，其在默认情况下不开启。mhash 的可以用于创建校验数值，消息摘要，消息认证码，以及无需原文的关键信息保存（如密码）等。

```bash
	[root@localhost ~]$ cd /opt/mhash-0.9.9.9
	[root@localhost ~]$ ./configure && make && make install
```

##### 安装 mcrypt
>​	**mcrypt 是 php 里面重要的加密支持扩展库**。mcrypt 库支持 20 多种加密算法和 8 种加密模式

```bash
	
	[root@localhost ~]$ cd /opt/mcrypt-2.6.8
	# 此扩展库需要以export形式配置
	[root@localhost ~]$ export LD_LIBRARY_PATH=/usr/local/rely/libmcrypt/lib:/usr/local/lib 
	[root@localhost ~]$ ./configure --with-libmcrypt-prefix=/usr/local/rely/libmcrypt 
	[root@localhost ~]$ make && make install
	
```

##### 安装 zlib
>​	**zlib 是提供数据压缩用的函式库**，由 Jean-loup Gailly 与 Mark Adler 所开发，初版 0.9 版在 1995年 5 月 1 日发表。zlib 使用 DEFLATE 算法，最初是为 libpng 函式库所写的，后来普遍为许多软件所使用。此函式库为自由软件，使用 zlib 授权

```bash
	
	[root@localhost ~]$ cd /opt/zlib-1.2.8
	[root@localhost ~]$ ./configure
	[root@localhost ~]$ vim Makefile
	# 找到 CFLAGS=-O3 -DUSE_MMAP , 在后面加入 -fPIC .注意是小f 大写PIC	
		CFLAGS=-O3 -DUSE_MMAP -fPIC
	[root@localhost ~]$ make && make install	
	
```

##### 安装 libpng
>​	libpng 软件包包含 libpng 库.这些库**被其他程式用于解码 png 图片**

```bash
	
	[root@localhost ~]$ cd /opt/libpng-1.6.37
	[root@localhost ~]$ ./configure --prefix=/usr/local/rely/libpng 
	[root@localhost ~]$ make && make install

	'注意':libpng与zlib
	# -----------------------------------------------------------------------|
	#	[root@localhost ~]$ cd /usr/local/php/libpng-1.6.30 
	#	...
	#	make && make install 
	#	...
	#	make all-am 
	#	make[1]: Entering directory '/usr/local/php/libpng-1.6.30'
	#	/bin/bash ./libtool --tag=CC --mode=link gcc -g -O2 -o pngfix \
	#		contrib/tools/pngfix.o libpng16.la -lm -lz -lmlibtool: 
	#	link: gcc -g -O2 -o .libs/pngfix contrib/tools/pngfix.o ./.libs/libpng16.so \
	#		 -lz -lm./.libs/libpng16.so: 
	#	undefined reference to'inflateValidate' collect2: 
	#	error: ld returned 1 exit status 
	#	make[1]: *** [pngfix] Error 1 
	#	make[1]: Leaving directory `/usr/local/php/libpng-1.6.30’ 
	#	make: *** [all] Error 2
	原因 : 'zlib.1.2.11' 版本跟 'libpng-1.6.31' 版本'不能正常匹配'.
        根据他人测试，libpng-1.6.30 与 zlib.1.2.11 也不行, 目前可行的是：zlib-1.2.8 与 libpng-1.6.21 		
        遂将上文下载软件版本更改为 zlib-1.2.8 与 libpng-1.6.21 
            wget https://sourceforge.net/projects/libpng/files/zlib/1.2.8/zlib-1.2.8.tar.gz
            wget https://sourceforge.net/projects/libpng/files/libpng16/1.6.37/libpng-1.6.21.tar.gz
            
```

##### 安装 jpeg6
>​	**jpeg6 提供用于解码.jpg 和.jpeg 图片的库文件**

```bash

	'注意：此软件默认不会自动创建所需目录，所以目录必须手工建立'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ mkdir /usr/local/rely/jpeg6
		[root@localhost ~]$ mkdir /usr/local/rely/jpeg6/bin
		[root@localhost ~]$ mkdir /usr/local/rely/jpeg6/lib
		[root@localhost ~]$ mkdir /usr/local/rely/jpeg6/include
		[root@localhost ~]$ mkdir -p /usr/local/rely/jpeg6/man/man1
	
	# -----------------------------------------------------------------------|
		# [root@localhost ~]$ yum -y install libtool* # 以一键安装
		[root@localhost ~]$ cd /opt/jpeg-6b
		[root@localhost ~]$ cp -a /usr/share/libtool/config/config.sub ./
		[root@localhost ~]$ cp -a /usr/share/libtool/config/config.guess ./
	
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ ./configure --prefix=/usr/local/rely/jpeg6 --enable-shared --enable-static
		[root@localhost ~]$ make && make install

	# –enable-shared 与–enable-static 参数分别为建立共享库和静态库使用的 libtool
	
```

##### 安装 freetype
>​	**FreeType 库是一个完全免费(开源)的、高质量的且可移植的字体引擎**，它提供统一的接口来访问多种字体格式文件，支持单色位图、反走样位图的渲染。

```bash
	[root@localhost ~]$ cd /opt/freetype-2.3.5
	[root@localhost ~]$ ./configure --prefix=/usr/local/rely/freetype/ && make && make install	
```

#### Tips
```bash
	
	--nodeps     是强制解除依赖关系
	--prefix=    指定编译路径
	
	export 命令
	# -----------------------------------------------------------------------|
	# 在 shell 中执行程序时，shell 会提供一组环境变量。
	# export 可新增，修改或删除环境变量，供后续执行的程序使用。export 的效力仅限于该次登陆操作。
		export [-fnp][变量名称]=[变量设置值]
			    -f 　代表[变量名称]中为函数名称。
			    -n 　删除指定的变量。变量实际上并未删除，只是不会输出到后续指令的执行环境中。
			    -p 　列出所有的shell赋予程序的环境变量。
			    
```

---

### 安装 Apache

#### 安装准备
>​	源码包 2.4.\* 版本之后中默认没有集成 apr 的依赖包
>​	安装httpd时需要安装 apr 和 apr-util 。这两个是一个通用函数库，可以使httpd不关心底层操作系统平台，方便移植

```bash
    [root@localhost ~]$ cd /opt/httpd-2.4.53
    [root@localhost ~]$ cp -a /opt/apr-1.7.0 /opt/httpd-2.4.53/srclib/apr # 上文中以提前下载完毕
    [root@localhost ~]$ cp -a /opt/apr-util-1.6.1 /opt/httpd-2.4.53/srclib/apr-util     
```

#### 安装 pcre
>​	Apache 默认需要依赖 pcre 软件，但由于 Apache 软件版本更新较快，系统预安装的 pcre 通常无法不匹配，所以需要人为手动安装适合版本

```bash
	[root@localhost ~]$ cd /opt/pcre-8.45
	[root@localhost ~]$ ./configure && make && make install
	# [root@localhost ~]$ yum -y install pcre-devel	# 以一键安装	
```

#### 安装 mod_ssl
>​	Apache 的加密传输模块 mod_ssl，需要安装此软件产生

```bash
	# [root@localhost ~]$ yum -y install openssl-devel expat-devel libxml2-devel # 以一键安装	
```

#### 安装 Apache 
>​	Apache 服务名称 httpd

```bash

	'编译安装'
	# -----------------------------------------------------------------------|
        [root@localhost ~]$ cd /opt/httpd-2.4.53
        [root@localhost ~]$ ./configure --prefix=/usr/local/apache2 --sysconfdir=/usr/local/apache2/etc --with-included-apr --enable-so --enable-deflate=shared --enable-expires=shared --enable-rewrite=shared --enable-ssl
        [root@localhost ~]$ make && make install
        # 若前面配置 zlib 时没有指定安装目录，Apache 配置时不要添加 –with-z=/usr/local/zlib/参数， 
        # --enable-ssl 选项是为了后期实现 https 提前设置的参数

	'其他选项' ./configure --hlep 
	# -----------------------------------------------------------------------|  
        --sysconfdir=DIR				//指定配置文件路径
        --enable-so						//支持动态共享模块，必加项，用于以模块的方式让php与apache结合工作
        --enable-proxy-fcgi				//用于以fcgi（fastcgi）的方式让php与apache结合工作
        --enable-cgi 					//用于以cgi的方式让php与apache结合工作
            # apache与php结合工作的几种方式：
            #    模块：php以模块的形式集成在httpd中，httpd和php一个进程
            #    CGI：httpd用CGI和php交互，php独立创建进程（好像是tcp9000），由httpd创建、管理php的进程
            #    FCGI：php完全独立了，自己单独的服务器，自己预先创建进程，自己管理进程
        --enable-ssl					//支持ssl，用于需要启用https的站点
        --enable-deflate				//支持压缩，即压缩后发送给客户端浏览器，浏览器再解压缩，节省流量
        --enable-mpms-shared=MPM-LIST	//定义要启用的httpd支持的MPM模式（多道处理模块）event|worker|prefork|winnt|all
        --with-mpm=MPM 					//定义MPM后设定默认的MPM模式（MPM={event|worker|prefork|winnt}），2.4默认event
            # MPM的几种模式：
            #    event：一个进程（内部生成多个线程）来响应多个请求，进程通过事件驱动（轮询和通知）的机制来确定内部线程的处理状态
            #    worker：一个进程生成多个子进程（线程）来响应多个请求
            #    prefork：一个进程响应一个请求
        --enable-cgid				//CGI scripts. Enabled by default with threaded MPMs
        	# 线程方式event|worker的MPM模式时必须启用
        --enable-rewrite			//支持URL重写，就是重定向啦

```

#### 启动 Apache 
```bash

	'配置文件 Apache' 
	# -----------------------------------------------------------------------| 
	[root@localhost ~]$ vim /usr/local/apache2/etc/httpd.conf
		 搜索 ServerName （约在 200 行左右）  
		 改为 ServerName localhost:80（并且去掉前面的#注释）
		 
	'启动 Apache' 
	# -----------------------------------------------------------------------| 	 
        [root@localhost ~]$ /usr/local/apache2/bin/apachectl start
        [root@localhost ~]$ ps aux | grep httpd 		# 查看进程
        [root@localhost ~]$ netstat -antp | grep :80	# 查看端口

```

#### 配置环境/服务
```bash

	'可选配置'：
    # -----------------------------------------------------------------------| 
        让service命令识别启动脚本：
			cp /usr/local/apache2/bin/apachectl /etc/init.d/httpd
			chmod +x /etc/init.d/httpd
			systemctl daemon-reload 
            # 让系统能够直接识别/usr/local/apache/bin/下的命令：
            # 方法一：ln -s /usr/local/apache/bin/* /usr/bin # 软链接到某一个PATH变量目录中
            # 方法二：编辑一个开机启动脚本，追加PATH变量
        
        配置 Apache 环境变量
				echo "export PATH=$PATH:/usr/local/apache2/bin">>/etc/profile	
				
        开机自启动：
                vim /etc/rc.d/rc.local
                /usr/local/apache/bin/apachectl start # 开机启动
            
 	'Apache 编译安装默认的目录' 
	# -----------------------------------------------------------------------| 
         bin：		# 二进制文件（启动脚本是此目录下的apachectl）    
         cgi-bin：	# CGI程序的存放目录
         htdocs：	# 网页的存放目录
         man：		# 帮助文档
         modules：	# 模块目录
         error：		# 错误信息
         icons：		# 图标  
         logs：		# 日志文件
         manual：	# 官方手册

    # 如果采用 yum 安装方式, apache 的路径文件如下
    # -----------------------------------------------------------------------|
		# /etc/httpd					：主目录  
		# /etc/httpd/conf/httpd.conf 	：主配置文件  
		# /etc/httpd/conf.d				：附加模块配置文件  
		# /etc/httpd/modules			：模块文件路径链接  
		# /etc/httpd/bin 				：二进制命令  
		# /etc/httpd/logs				：默认日志文件位置

```

> **报错提示：** 若启动时提示`/usr/local/apache2/modules/mod_deflate.so 无权限`，可关闭 SELinux 解 决，类似此类.so 文件不能载入或没有权限的问题，都是 SELinux 问题，MySQL 和 Apache 都可能有类似问题。

> **验证：** 通过浏览器输入地址访问：`http://服务器 ip`，若显示“It works”即表明 Apache 正常工作

---

### 安装 MySQL
#### MySQL 分类
```bash

	'MySQL 安装方式分类'
	# -----------------------------------------------------------------------|
	# 三种包虽目的功能一致,但安装、配置方式却不尽相同.需要根据下载的版本进行选择性调整安装方法.
		1. 所有操作系统(通用)源码包
		2. linux系统 glibc 源码包
		3. rpm包

		glibc是GNU发布的libc库，即c运行库。glibc是linux系统中最底层的api，几乎其它任何运行库都会依赖于glibc
		如果rpm包名里面有 linux 并且指定了 linux 版本，glibc则是针对 linux 的c运行库。

	'特点'
	# -----------------------------------------------------------------------|
        安装方式     优点				              缺点
        RPM包       安装卸载简单                    可定制性差                        
        glibc      可定制性相比与rpm包灵活           安装相比rpm包复杂,需要手动初始化数据库 
        源码包      可定制性最强,格局需求和功能定制     安装繁杂,需要手动初始化数据库         

	'下载链接'
	# -----------------------------------------------------------------------|
		通用源码包: # 包含 boost 
		https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.37.tar.gz 
		https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-boost-8.0.28.tar.gz
		 
		glibc: # linux 版本
		https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.37-linux-glibc2.12-x86_64.tar 
		https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-8.0.28-linux-glibc2.12-x86_64.tar 

	'检查冲突项'
	# -----------------------------------------------------------------------|
		rpm  -qa | grep mysql | xargs rpm -e --nodeps    > /dev/null  2>&1
		rpm  -qa | grep mariadb | xargs rpm -e --nodeps  > /dev/null  2>&1

```

#### [源码包]MySQL安装 

##### 安装 ncurses
>​	**Ncurses 提供字符终端处理库**，包括面板和菜单。它提供了一套控制光标，建立窗口，改变前景背景颜色以及处理鼠标操作的函数。使用户在字符终端下编写应用程序时绕过了那些恼人的底层机制。简而言之，他是一个可以使应用程序直接控制终端屏幕显示的函数库。

```bash

	[root@localhost ~]$ yum -y install ncurses-devel ncurses-libs # 以一键安装
	[root@localhost ~]$ cd /opt/ncurses-6.3
	[root@localhost ~]$ ./configure --with-shared --without-debug --without-ada --enable-overwrite
	[root@localhost ~]$ make && make install

```

##### 安装 bison
>​	mysql 在 5.5 以后，不再使用./configure 工具，进行编译安装。而使用 cmake 工具替代了./configure工具。bison 是一个自由软件，用于自动生成语法分析器程序，可用于所有常见的操作系统

```bash
	yum -y install bison bison-devel # 以一键安装
```

##### 编译 MySQL

```bash
	
	'编译准备'
	# -----------------------------------------------------------------------|	
        [root@localhost ~]$ rm -rf /etc/my.cnf        # 清除系统 /etc/my.cnf 配置文件 
        [root@localhost ~]$ rm -rf /etc/my.cnf.d

        [root@localhost ~]$ mkdir -p /opt/mysql-5.7.37/bld
        [root@localhost ~]$ cd /opt/mysql-5.7.37/bld
        [root@localhost ~]$ cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/usr/local/mysql/data -DSYSCONFDIR=/usr/local/mysql/etc -DWITH_INNOBASE_STORAGE_ENGINE=ON -DWITH_MYISAM_STORAGE_ENGINE=ON -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DENABLED_LOCAL_INFILE=ON -DMYSQL_TCP_PORT=3306 -DMYSQL_UNIX_ADDR=/usr/local/mysql/tmp/mysql.sock -DWITH_INNODB_MEMCACHED=ON -DWITH_BOOST=../boost

        [root@localhost ~]$ make
        [root@localhost ~]$ make install
	
	'清理编译文件'
	# -----------------------------------------------------------------------|
	    make clean 
	    rm CMakeCache.txt 			    

    'cmake参数解释' # 仅解释常用配置，详细可官方文档查询
    # -----------------------------------------------------------------------|
        cmake
        ..													# 源码目录为上级目录
        -DCMAKE_INSTALL_PREFIX=/usr/local/mysql			    # 指定MySQL安装目录
        -DMYSQL_DATADIR=/usr/local/mysql/data             	# 指定MySQL数据目录
        -DSYSCONFDIR=/usr/local/mysql/etc                 	# 指定my.cnf选项文件目录
        -DWITH_INNOBASE_STORAGE_ENGINE=ON                 	# Innodb引擎
        -DWITH_MYISAM_STORAGE_ENGINE=ON					    # MyISAM引擎
        -DDEFAULT_CHARSET=utf8								# 服务器字符集，默认latin1
        -DDEFAULT_COLLATION=utf8_general_ci				    # 服务器排序规则，默认latin1_swedish_ci
        -DENABLED_LOCAL_INFILE=ON							# 是否为加载数据启用本地，默认为OFF
        -DMYSQL_TCP_PORT=3306								# 服务器监听端口，默认为3306
        -DMYSQL_UNIX_ADDR=/usr/local/mysql/tmp/mysql.sock Unix # 套接字文件路径，默认/tmp/mysql.sock
        -DWITH_INNODB_MEMCACHED=ON							# 是否生成memcached共享库，默认OFF
        -DWITH_BOOST=../boost                               # Boost库源代码的位置，指向下载的源码包里，相对/绝对路径皆可

    '其他配置'
    # -----------------------------------------------------------------------|
        # -DCMAKE_INSTALL_PREFIX=/usr/local/mysql            安装位置
        # -DMYSQL_UNIX_ADDR=/usr/local/mysql/run/mysql.sock  指定 socket（套接字）文件位置
        # -DEXTRA_CHARSETS=all                               扩展字符支持
        # -DDEFAULT_CHARSET=utf8                             默认字符集
        # -DDEFAULT_COLLATION=utf8_general_ci       		  默认字符校对
        # -DWITH_MYISAM_STORAGE_ENGINE=1            		  安装 myisam 存储引擎
        # -DWITH_INNOBASE_STORAGE_ENGINE=1          		  安装 innodb 存储引擎
        # -DWITH_MEMORY_STORAGE_ENGINE=1           		  安装 memory 存储引擎
        # -DWITH_READLINE=1                         		  支持 readline 库
        # -DENABLED_LOCAL_INFILE=1                  		  启用加载本地数据
        # -DMYSQL_USER=mysql                        		  指定 mysql 运行用户
        # -DMYSQL_TCP_PORT=3306                     		  指定 mysql 端口
        # -DSYSCONFDIR=/etc/my.cnf                  		  配置文件的默认路径  

        # 由于MySQL支持很多的存储引擎而默认编译的存储引擎包括：csv、myisam、myisammrg和heap。
        # 若要安装其它存储引擎，可以使用类似如下编译选项：
        # -DWITH_ARCHIVE_STORAGE_ENGINE=1           		  安装ARCHIVE存储引擎  
        # -DWITH_BLACKHOLE_STORAGE_ENGINE=1         		  安装BLACKHOLE存储引擎  
        # -DWITH_FEDERATED_STORAGE_ENGINE=1         		  安装FEDERATED存储引擎

        # 若要明确指定不编译某存储引擎，可以使用类似如下的选项：
        # -DWITHOUT__STORAGE_ENGINE=1
        # -DWITHOUT_EXAMPLE_STORAGE_ENGINE=1        		  不启用或不编译EXAMPLE存储引擎  
        # -DWITHOUT_FEDERATED_STORAGE_ENGINE=1  
        # -DWITHOUT_PARTITION_STORAGE_ENGINE=1

        # 如若要编译进其它功能，如SSL等，则可使用类似如下选项来实现编译时使用某库或不使用某库：
        # -DWITH_READLINE=1  
        # -DWITH_SSL=system                         		  表示使用系统上的自带的SSL库  
        # -DWITH_ZLIB=system  
        # -DWITH_LIBWRAP=0 

        # 其它常用的选项：
        # -DMYSQL_TCP_PORT=3306                    		       设置默认端口的  
        # -DMYSQL_UNIX_ADDR=/tmp/mysql.sock         		   MySQL进程间通信的套接字的位置  
        # -DENABLED_LOCAL_INFILE=1                  		   是否启动本地的LOCAL_INFILE  
        # -DDEFAULT_COLLATION=utf8_general_ci       		   默认的字符集排序规则  
        # -DWITH_DEBUG=0                            		   是否启动DEBUG功能  
        # -DENABLE_PROFILING=1                      		   是否启用性能分析功能

	[官方手册: MySQL 5.7 参考手册 ： 2.9.7 MySQL 源代码配置选项]	
	
```

##### 初始化

>​	**MySQL 安装后需要调整相应配置文件和参数才能正常运行**

###### 初始化准备
```bash

	'创建用户'
	# -----------------------------------------------------------------------|
		# 创建mysql用户和组，该用户没有设置登录权限，如需要请自行修改
		[root@localhost ~]$ groupadd mysql
		[root@localhost ~]$ useradd -r -s /sbin/nologin mysql -g mysql
            # -M  不创建主目录 
            # -s  不允许登录  
            # -r  建立系统帐号。
            # -g  加入mysql组 

	'创建相关目录'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ cd /usr/local/mysql # 为后续初始化做准备
		[root@localhost ~]$ mkdir -p /usr/local/mysql/data
		[root@localhost ~]$ mkdir -p /usr/local/mysql/etc
		[root@localhost ~]$ mkdir -p /usr/local/mysql/tmp
		[root@localhost ~]$ mkdir -p /usr/local/mysql/logs

	'将目录所有权授予mysql用户和mysql组'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ chown -R mysql:mysql /usr/local/mysql

```

###### 初始化 MySQL
```bash

	'初始化'
	# -----------------------------------------------------------------------|
		# 使用 my.cnf 配置文件和 mysql 用户初始化数据目录
		[root@localhost ~]$ /usr/local/mysql/bin/mysqld \
		--defaults-file=etc/my.cnf \                             # 配置文件地址
		--initialize \                                           
		--basedir=/usr/local/mysql \                             # mysql地址
		--datadir=/usr/local/mysql/data \                        # 数据地址
		--socket=/usr/local/mysql/tmp/mysql.sock \               # 套接字地址
		--pid-file=/usr/local/mysql/tmp/mysql.pid \              # PID地址
		--log_error=/usr/local/mysql/logs/error.log \            # 错误日志地址
		--general_log_file=/usr/local/mysql/logs/general.log \   # 日志地址
		--slow_query_log_file=/usr/local/mysql/logs/slowq.log \  # 慢日志地址
		--user=mysql                                             # 用户
    
        '注意':初始化完成后会生成临时密码
        '注意':如忘记临时密码可通过error.log查询临时密码 # 根据 /etc/my.cnf 配置文件查看具体路径
         [root@localhost ~]$ cat /usr/local/mysql/logs/error.log | grep root@localhost:
	
	'关键路径' # 切记留存,为日后查看目录给予方便
	# -----------------------------------------------------------------------|
        basedir=/usr/local/mysql                                # MySQL安装根目录
        datadir=/usr/local/mysql/data                           # MySQL数据文件目录
        socket=/usr/local/mysql/tmp/mysql.sock                  # Unix套接字文件路径，默认/tmp/mysql.sock
        pid-file=/usr/local/mysql/tmp/mysql.pid                 # 服务进程pid文件路径
        log_error=/usr/local/mysql/logs/error.log               # 错误日志存放路径
        general_log_file=/usr/local/mysql/logs/general.log      # 通用查询日志存放路径
        slow_query_log_file=/usr/local/mysql/logs/slowq.log	 	# 慢查询日志存放路径
        
```

##### 配置环境变量

```bash

	'文件授权 授权 MySQL 安装文件'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ setfacl -m u:mysql:rwx -R /usr/local/mysql
		[root@localhost ~]$ setfacl -m d:u:mysql:rwx -R /usr/local/mysql

	'修改配置文件 /etc/my.cnf'
	# -----------------------------------------------------------------------|
		[mysqld]
		user = mysql
		port = 3306
		bind-address = 0.0.0.0
		basedir = /usr/local/mysql
		datadir = /usr/local/mysql/data
		socket = /usr/local/mysql/tmp/mysql.sock
		pid-file = /usr/local/mysql/tmp/mysql.pid
		character-set-server = utf8
		collation-server = utf8_general_ci
		max_connections = 1024
		log-error = /usr/local/mysql/logs/error.log
		default-time_zone = '+8:00'
		# skip-grant-tables # 清除登陆密码 如忘记密码可添加此项登陆mysql

	'配置环境/服务'
	# -----------------------------------------------------------------------|	
		[root@localhost ~]$ cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
		[root@localhost ~]$ chmod +x /etc/init.d/mysqld
		[root@localhost ~]$ systemctl daemon-reload
		# 配置MySQL环境变量
		[root@localhost ~]$ echo "export PATH=$PATH:/usr/local/mysql/bin">>/etc/profile
		[root@localhost ~]$ source /etc/profile 

```

<details> 
	<summary><mark class="hltr-cyan"> my.cnf 配置文件详情</mark></summary> 
	<pre class="language-bash">
		<code>
# 创建 my.cnf 配置文件
	vim etc/my.cnf # 默认路径
# 按 i 键进入INSERT模式，粘贴下面配置（本地测试环境虚拟机1C1G，我还总结了一些计算公式，篇幅太长就不粘贴了，欢迎讨论）
------------------------------------------------------------------------------------------------|
[client]                                                # 客户端设置
port=3306                                               # 服务器监听端口，默认为3306
socket=/usr/local/mysql/tmp/mysql.sock                  # Unix套接字文件路径，默认/tmp/mysql.sock
------------------------------------------------------------------------------------------------|
[mysqld]                                                # 服务端设置
## 一般配置选项
port=3306                                               # 服务器监听端口，默认为3306
basedir=/usr/local/mysql                                # MySQL安装根目录
datadir=/usr/local/mysql/data                           # MySQL数据文件目录
socket=/usr/local/mysql/tmp/mysql.sock                  # Unix套接字文件路径，默认/tmp/mysql.sock
pid-file=/usr/local/mysql/tmp/mysql.pid                 # 服务进程pid文件路径
character_set_server=utf8                               # 默认字符集
collation-server=utf8_general_ci						# 默认字符集校对
default_storage_engine=InnoDB                           # 默认InnoDB存储引擎
user=mysql
------------------------------------------------------------------------------------------------|
## 连接配置选项
max_connections=200                                     # 最大并发连接数
table_open_cache=400                                    # 表打开缓存大小，默认2000
open_files_limit=1000                                   # 打开文件数限制，默认5000
max_connect_errors=200                                  # 最大连接失败数，默认100
back_log=100                                            # 请求连接队列数
connect_timeout=20                                      # 连接超时时间，默认10秒
interactive_timeout=1200                                # 交互式超时时间，默认28800秒
wait_timeout=600                                        # 非交互超时时间，默认28800秒
net_read_timeout=30                                     # 读取超时时间，默认30秒
net_write_timeout=60                                    # 写入超时时间，默认60秒
max_allowed_packet=8M                                   # 最大传输数据字节，默认4M
thread_cache_size=10                                    # 线程缓冲区（池）大小
thread_stack=256K                                       # 线程栈大小，32位平台196608、64位平台262144
------------------------------------------------------------------------------------------------|
## 临时内存配置选项
tmpdir=/tmp                                             # 临时目录路径
tmp_table_size=64M                                      # 临时表大小，默认16M
max_heap_table_size=64M                                 # 最大内存表大小，默认16M
sort_buffer_size=1M                                     # 排序缓冲区大小，默认256K
join_buffer_size=1M                                     # join缓冲区大小，默认256K
------------------------------------------------------------------------------------------------|
## Innodb配置选项
#innodb_thread_concurrency=0							# InnoDB线程并发数
innodb_io_capacity=200                                  # IO容量，可用于InnoDB后台任务的每秒I/O操作数（IOPS），
innodb_io_capacity_max=400                              # IO最大容量，InnoDB在这种情况下由后台任务执行的最大IOPS数
innodb_lock_wait_timeout=50                             # InnoDB引擎锁等待超时时间，默认50（单位：秒）
innodb_buffer_pool_size=512M							# InnoDB缓冲池大小，默认128M
innodb_buffer_pool_instances=4                          # InnoDB缓冲池划分区域数
innodb_max_dirty_pages_pct=75							# 缓冲池最大允许脏页比例，默认为75
innodb_flush_method=O_DIRECT                            # 日志刷新方法，默认为fdatasync
innodb_flush_log_at_trx_commit=2                        # 事务日志刷新方式，默认为0
transaction_isolation=REPEATABLE-READ                   # 事务隔离级别，默认REPEATABLE-READ
innodb_data_home_dir=/usr/local/mysql/data              # 表空间文件路径，默认保存在MySQL的datadir中
innodb_data_file_path=ibdata1:128M:autoextend           # 表空间文件大小
innodb_file_per_table=ON                                # 每表独立表空间
innodb_log_group_home_dir=/usr/local/mysql/data         # redoLog文件目录，默认保存在MySQL的datadir中
innodb_log_files_in_group=2                             # 日志组中的日志文件数，默认为2
innodb_log_file_size=128M                               # 日志文件大小，默认为48MB
innodb_log_buffer_size=32M                              # 日志缓冲区大小，默认为16MB
------------------------------------------------------------------------------------------------|
## MyISAM配置选项
key_buffer_size=32M                                     # 索引缓冲区大小，默认8M
read_buffer_size=4M										# 顺序读缓区冲大小，默认128K
read_rnd_buffer_size=4M									# 随机读缓冲区大小，默认256K
bulk_insert_buffer_size=8M                              # 块插入缓冲区大小，默认8M
myisam_sort_buffer_size=8M								# MyISAM排序缓冲大小，默认8M
#myisam_max_sort_file_size=1G                           # MyISAM排序最大临时大小
myisam_repair_threads=1                                 # MyISAM修复线程
skip-external-locking                                   # 跳过外部锁定，启用文件锁会影响性能
------------------------------------------------------------------------------------------------|
## 日志配置选项
log_output=FILE                                         # 日志输出目标，TABLE（输出到表）、FILE（输出到文件）、NONE（不输出），可选择一个或多个以逗>号分隔
log_error=/usr/local/mysql/logs/error.log               # 错误日志存放路径
log_error_verbosity=1                                   # 错误日志过滤，允许的值为1（仅错误），2（错误和警告），3（错误、警告和注释），默认值为3。
log_timestamps=SYSTEM                                   # 错误日志消息格式，日志中显示时间戳的时区，UTC（默认值）和 SYSTEM（本地系统时区）
general_log=ON                                          # 开启查询日志，一般选择不开启，因为查询日志记录很详细，会增大磁盘IO开销，影响性能
general_log_file=/usr/local/mysql/logs/general.log      # 通用查询日志存放路径
## 慢查询日志配置选项
slow_query_log=ON                                       # 开启慢查询日志
slow_query_log_file=/usr/local/mysql/logs/slowq.log	    # 慢查询日志存放路径
long_query_time=2                                       # 慢查询时间，默认10（单位：秒）
min_examined_row_limit=100                              # 最小检查行限制，检索的行数必须达到此值才可被记为慢查询
log_slow_admin_statements=ON                            # 记录慢查询管理语句
log_queries_not_using_indexes=ON                        # 记录查询未使用索引语句
log_throttle_queries_not_using_indexes=5                # 记录未使用索引速率限制，默认为0不限制
log_slow_slave_statements=ON                            # 记录从库复制的慢查询，作为从库时生效，从库复制中如果有慢查询也将被记录
------------------------------------------------------------------------------------------------|
## 复制配置选项
server-id=1                                             # MySQL服务唯一标识
log-bin=mysql-bin                                       # 开启二进制日志，默认位置是datadir数据目录
log-bin-index=mysql-bin.index                           # binlog索引文件
binlog_format=MIXED                                     # binlog日志格式，分三种：STATEMENT、ROW或MIXED，MySQL 5.7.7之前默认为STATEMENT，之后默认为ROW
binlog_cache_size=1M                                    # binlog缓存大小，默认32KB
max_binlog_cache_size=1G                                # binlog最大缓存大小，推荐最大值为4GB
max_binlog_size=256M                                    # binlog最大文件大小，最小值为4096字节，最大值和默认值为1GB
expire_logs_days=7                                      # binlog过期天数，默认为0不自动删除
log_slave_updates=ON                                    # binlog级联复制
sync_binlog=1                                           # binlog同步频率，0为禁用同步（最佳性能，但可能丢失事务），为1开启同步（影响性能，但最安全不会丢失任何事务），为N操作N次事务后同步1次
relay_log=relay-bin                                     # relaylog文件路径，默认位置是datadir数据目录
relay_log_index=relay-log.index                         # relaylog索引文件
max_relay_log_size=256M                                 # relaylog最大文件大小
relay_log_purge=ON                                      # 中继日志自动清除，默认值为1（ON）
relay_log_recovery=ON                                   # 中继日志自动恢复
auto_increment_offset=1                                 # 自增值偏移量
auto_increment_increment=1                              # 自增值自增量
slave_net_timeout=60                                    # 从机连接超时时间
replicate-wild-ignore-table=mysql.%                     # 复制时忽略的数据库表，告诉从线程不要复制到与给定通配符模式匹配的表
skip-slave-start                                        # 跳过Slave启动，Slave复制进程不随MySQL启动而启动
------------------------------------------------------------------------------------------------|
## 其他配置选项
#memlock=ON                                             # 开启内存锁，此选项生效需系统支持mlockall()调用，将mysqld进程锁定在内存中，防止遇到操作系统导致mysqld交换到磁盘的问题
[mysqldump]                                             # mysqldump数据库备份工具
quick                                                   # 强制mysqldump从服务器查询取得记录直接输出，而不是取得所有记录后将它们缓存到内存中
max_allowed_packet=16M                                  # 最大传输数据字节，使用mysqldump工具备份数据库时，某表过大会导致备份失败，需要增大该值（大>于表大小即可）
[myisamchk]                                             # 使用myisamchk实用程序可以用来获得有关你的数据库表的统计信息或检查、修复、优化他们
key_buffer_size=32M                                     # 索引缓冲区大小
myisam_sort_buffer_size=8M                              # 排序缓冲区大小
read_buffer_size=4M                                     # 读取缓区冲大小
write_buffer_size=4M                                    # 写入缓冲区大小
# Esc（退出编辑模式）
# :wq（保存退出）
# :q（不想写了，直接退出）
# :q!（写了一半不想写了，不保存强制退出）
		</code>
	</pre>
</details>


##### 验证MySQL服务

```bash

	'开放防火墙 3306 端口'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ firewall-cmd --permanent --add-port=3306/tcp
		[root@localhost ~]$ firewall-cmd --reload

	'端口验证'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ systemctl start mysqld
		[root@localhost ~]$ netstat -antp | grep 3306

	'获取 MySQL 临时密码'
	# -----------------------------------------------------------------------|
        [root@localhost ~]$ grep "password" /usr/local/mysql/logs/error.log	

	'登录 MySQL'
	# -----------------------------------------------------------------------|
        # 登录mysql
            [root@localhost ~]$ mysql -uroot -p
        # 设置密码
            ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
        # 验证密码重新登录
            [root@localhost ~]$ /usr/local/mysql/bin/mysql -u root -p  
            [root@localhost ~]$ Enter password:   
            mysql > show databases;  
            mysql > use test;  
            mysql > show tables;  
            mysql > quit
	
	'配置远程登陆 MySQL'
	# -----------------------------------------------------------------------|
        # 用新密码登进去试试
        	[root@localhost ~]$ mysql -uroot -p
        # 授权 所有权限（all），所有数据库（*.*） 给 用户名（root），任何主机（%），密码（yourpassword）
        	mysql > grant all privileges on *.* to 'root'@'%' identified by 'yourpassword' with grant option;
        # 刷新权限使其立即生效
        	mysql > flush privileges;
        # 退出MySQL
        	mysql > quit
    
```

---

#### [glibc-荐]MySQL安装 

##### 安装 glibc / libaio
```bash

	'安装依赖项'
	# -----------------------------------------------------------------------|
        [root@localhost ~]$ rpm -qa | grep  libaio 
        [root@localhost ~]$ rpm -qa | grep  glibc
        [root@localhost ~]$ yum  -y install libaio-devel glibc-devel # 以一键安装

        '注'：要求Linux系统的glibc版本要比2.12新，可以使用ldd --version查看glibc版本

```

##### 安装 MySQL
```bash

	'解压安装包'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ tar -xf mysql-5.7.24-linux-glibc2.12-x86_64.tar
        # 拆包两项,分别解压归档
		[root@localhost ~]$ tar -zxf mysql-5.7.24-linux-glibc2.12-x86_64.tar.gz
        [root@localhost ~]$ tar -zxf mysql-test-5.7.24-linux-glibc2.12-x86_64.tar.gz 
        # 移动到目标目录
		[root@localhost ~]$ cp -r mysql-5.7.37-linux-glibc2.12-x86_64 /usr/local/mysql

	'初始化准备' 
	# -----------------------------------------------------------------------|
		# 为初始化 mysql 准备路径
		[root@localhost ~]$ mkdir -p /usr/local/mysql/data
		[root@localhost ~]$ mkdir -p /usr/local/mysql/etc
		[root@localhost ~]$ mkdir -p /usr/local/mysql/tmp
		[root@localhost ~]$ mkdir -p /usr/local/mysql/logs
		
		[root@localhost ~]$ cd /usr/local/mysql
		# 建立目录所有者,所属组.并修改权限
		[root@localhost ~]$ groupadd mysql
		[root@localhost ~]$ useradd -r -s /sbin/nologin mysql -g mysql
		[root@localhost ~]$ chown -R mysql:mysql /usr/local/mysql

	'初始化' 
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ /usr/local/mysql/bin/mysqld \
		--initialize \
		--basedir=/usr/local/mysql \
		--datadir=/usr/local/mysql/data \
		--socket=/usr/local/mysql/tmp/mysql.sock \
		--pid-file=/usr/local/mysql/tmp/mysql.pid \
		--log_error=/usr/local/mysql/logs/error.log \
		--general_log_file=/usr/local/mysql/logs/general.log \
		--slow_query_log_file=/usr/local/mysql/logs/slowq.log \
		--user=mysql
		# 详情查看源码包安装内容

	'修改配置文件'
	# -----------------------------------------------------------------------|	
		[root@localhost ~]$ vim /etc/my.cnf
		
			[mysqld]
			user = mysql
			port = 3306
			bind-address = 0.0.0.0
			basedir = /usr/local/mysql
			datadir = /usr/local/mysql/data
			socket = /usr/local/mysql/tmp/mysql.sock
			pid-file = /usr/local/mysql/tmp/mysql.pid
			character-set-server = utf8
			collation-server = utf8_general_ci
			max_connections = 1024
			log-error = /usr/local/mysql/logs/error.log
			default-time_zone = '+8:00'
			# skip-grant-tables # 取消登陆密码	
			[client] 
			port=3306  
			socket=/usr/local/mysql/tmp/mysql.sock
			
	'配置环境/服务'
	# -----------------------------------------------------------------------|	
		[root@localhost ~]$ cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
		[root@localhost ~]$ chmod +x /etc/init.d/mysqld
		[root@localhost ~]$ systemctl daemon-reload
		# 配置MySQL环境变量
		[root@localhost ~]$ echo "export PATH=$PATH:/usr/local/mysql/bin">>/etc/profile
		[root@localhost ~]$ source /etc/profile          	  # 重新加载配置文件
		[root@localhost ~]$ systemctl start mysqld

	'设定 / 修改密码'
	# -----------------------------------------------------------------------|	
		'获取临时登陆密码'
            [root@localhost ~]$ cat /usr/local/mysql/logs/error.log |grep root@localhost:
            [root@localhost ~]$ mysql -u root -p 
		
		# 设定密码
		# gblic模式,如首次登陆,使用如下命令,可跳过后段修改密码项
            mysql> ALTER USER USER() IDENTIFIED BY '123456'; 
            mysql> show databases;                    # 查看数据库内容 
            mysql> use mysql;                         # 进入mysql
		
		# 修改密码
            mysql> update user set authentication_string=password('123456') where user='root'; 
            mysql> select user, authentication_string, host from user;  # 查看密码
            mysql> flush privileges;                  					# 修改生效
            mysql> SELECT Host, User FROM mysql.user; 					# 查看数据库用户信息	
            mysql> quit                               					# 退出

		# 忘记登陆密码
            vim /etc/my.cnf 的段中 添加 skip-grant-tables 代码段,无密码进入 mysql 后重新设定
            '重新设定后,清除空密码状态'
            删除 /etc/my.cnf 的段中 skip-grant-tables 删除

	'开机自启'
	# -----------------------------------------------------------------------|	
		/sbin/chkconfig mysqld on
		
	'可选项' 
	# -----------------------------------------------------------------------|	
    # 授权 apache 对 mysql 的读写操作
        # 创建apache的进程守护者
        useradd -M -s /sbin/nologin apache
        # 文件授权 (授权 apache 安装文件)
        setfacl -m u:mysql:rwx -R /usr/local/apache2
        setfacl -m d:u:mysql:rwx -R /usr/local/apache2


```
---

#### [RPM包]MySQL 安装

##### 1. 安装准备
```shell

	'安装准备'
	# -----------------------------------------------------------------------|	
		# 安装依赖 wget libaio perl net-tools
			[root@localhost ~]$ yum install –y wget libaio perl net-tools
		# 下载mysql
			wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-xxx.rpm-bundle.tar
		# 若下载失败（下载成功此步跳过），可安装rz从本地上传mysql-5.7.xxx.rpm-bundle.tar
		# [root@localhost ~]$ yum -y install lrzsz
		# [root@localhost ~]$ rz # termius 不支持

		# 检查原有安装mysql或mariadb
			[root@localhost ~]$ rpm -qa |grep mysql
			[root@localhost ~]$ rpm -qa |grep mariadb
	
```

##### 2. 安装MySQL
```shell

	'解归档安装'
	# -----------------------------------------------------------------------|	
		# 建个目录存放解压文件
			[root@localhost ~]$ mkdir -p /usr/local/mysql
		# 解压缩
			[root@localhost ~]$ tar -xvf mysql-5.7.30-1.el7.x86_64.rpm-bundle.tar -C /usr/local/mysql
		# 进入目录准备安装
			[root@localhost ~]$ cd /usr/local/mysql
		# 开始安装，-ivh 其中i表示安装，v表示显示安装过程，h表示显示进度
			rpm -ivh mysql-community-common-5.7.37-1.el7.x86_64.rpm
			rpm -ivh mysql-community-libs-5.7.37-1.el7.x86_64.rpm
			rpm -ivh mysql-community-client-5.7.37-1.el7.x86_64.rpm
			rpm -ivh mysql-community-server-5.7.37-1.el7.x86_64.rpm

```

##### 3. 配置my.cnf
```shell

	# 具体配置方法可参考 [源码安装] 或 [glibc安装] 详解
	vi /etc/my.cnf
	# -----------------------------------------------------------------------|	
	# For advice on how to change settings please see
	# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html
	
	[mysqld]
	#
	# Remove leading # and set to the amount of RAM for the most important data
	# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
	# innodb_buffer_pool_size = 128M
	#
	# Remove leading # to turn on a very important data integrity option: logging
	# changes to the binary log between backups.
	# log_bin
	#
	# Remove leading # to set options mainly useful for reporting servers.
	# The server defaults are faster for transactions and fast SELECTs.
	# Adjust sizes as needed, experiment to find the optimal values.
	# join_buffer_size = 128M
	# sort_buffer_size = 2M
	# read_rnd_buffer_size = 2M
	datadir=/var/lib/mysql
	socket=/var/lib/mysql/mysql.sock
	
	# Disabling symbolic-links is recommended to prevent assorted security risks
	symbolic-links=0
	
	log-error=/var/log/mysqld.log
	pid-file=/var/run/mysqld/mysqld.pid
	
	# i（按i键进入INSERT模式，可开始编辑）
	# Esc（退出编辑模式）
	# :wq（保存退出）
	# :q（退出）
	# :q!（不保存强制退出）
	# -----------------------------------------------------------------------|	
	# 重启服务生效（另需注意：配置内路径需要自行创建目录及赋予权限）
	systemctl restart mysqld

```

##### 4. 配置MySQL
```shell

	'安装及启动服务'
	# -----------------------------------------------------------------------|	
		[root@localhost ~]$ rpm -ivh mysql-community-server-5.7.37-1.el7.x86_64.rpm
            #	1. - i (install)  安装
            #	2. - v (verbose)  显示详细信息
            #	3. - h (hash)     显示进度
            #	4. --nodeps       不检测依赖性
			
		# 启动mysqld服务
			[root@localhost ~]$ systemctl start mysqld
		# 下面列出其余systemctl命令（不用运行）
		# 查看mysqld服务状态
			[root@localhost ~]$ systemctl status mysqld
		# 停止mysqld服务
			[root@localhost ~]$ systemctl stop mysqld
		# 重新启动mysqld服务
			[root@localhost ~]$ systemctl restart mysqld
		# 配置mysqld开机自动启动
			[root@localhost ~]$ systemctl enable mysqld
		# 配置mysqld开机不自动启动
			[root@localhost ~]$ systemctl disable mysqld

	'登录MySQL修改密码'
	# -----------------------------------------------------------------------|	
		# 查询生成的临时密码
			[root@localhost ~]$ grep "password" /var/log/mysqld.log
		# 登录mysql
			[root@localhost ~]$ mysql -uroot -p
		    Enter password: #（输入查询到的临时密码）
		# 查询密码校验配置的系统变量
			mysql> SHOW VARIABLES LIKE 'validate_password%';
		# 设置密码校验策略（0 or LOW）
			mysql> set global validate_password_policy=0;
		# 设置密码校验长度
			mysql> set global validate_password_length=4;
		# 设置密码
			mysql> set password = password("1234");
		# 退出
			mysql> quit
			
```

#### 配置远程连接

```bash
	
	'配置远程连接'
	# -----------------------------------------------------------------------|	
		# 用新密码登进
			[root@localhost ~]$ mysql -uroot -p
		# 授权 所有权限（all），所有数据库（*.*） 给 用户名（root），任何主机（%），密码（123456）
			mysql > grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
		# 刷新权限使其立即生效
			mysql > flush privileges;
		# 退出MySQL
			mysql > quit
		# 连接测试
		
```

---

### 安装 PHP
#### 安装准备
##### 安装 libiconv 库
>​	libiconv库为需要做转换的应用提供了一个iconv()的函数，以**实现一个字符编码到另一个字符编码的转换**。

```bash

	'libiconv 安装配置'
	# -----------------------------------------------------------------------|	
		# 下载解压
			[root@localhost ~]$ wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz
			[root@localhost ~]$ tar -xf libiconv-1.14.tar.gz && cd libiconv-1.14
		# 编译准备.配置安装路径
			[root@localhost ~]$ ./configure --prefix=/usr/local/rely/libiconv
			[root@localhost ~]$ sed -i -e '/gets is a security/d' srclib/stdio.in.h
		# 编译安装
			[root@localhost ~]$ make && make install
		# 清除安装包	
			[root@localhost ~]$ cd ../ && rm -rf libiconv-1.14

```

#### 安装 PHP
```bash

	'下载PHP'
	# -----------------------------------------------------------------------|	
		[root@localhost ~]$ wget https://www.php.net/distributions/php-7.4.28.tar.gz
		[root@localhost ~]$ tar -zxvf php-7.4.28.tar.gz && cd php-7.4.28

	'安装依赖项' 
	# -----------------------------------------------------------------------|	
	# 依赖项根据具体配置模块进行选择
		[root@localhost ~]$ yum -y install xmlrpc-c-devel openssl-devel zlib-devel \
		 freetype-devel gd-devel libjpeg-turbo-devel libpng-devel libcurl-devel \ 
		 libxslt-devel libxml2-devel libmcrypt-devel libsq3-devel oniguruma oniguruma-devel

	'编译安装'
	# -----------------------------------------------------------------------|	
	# ./configure --help 查询具体对应信息
	
		./configure \
		--prefix=/usr/local/php7.4 \
		--with-config-file-path=/etc \
		--with-config-file-scan-dir=/etc/php.d \
		--with-apxs2=/usr/local/apache2/bin/apxs \
		--with-pdo-mysql=/usr/local/mysql \
		--with-xmlrpc \
		--with-openssl \
		--with-zlib \
		--with-freetype \
		--enable-gd \
		--with-jpeg \
		--with-png=/usr/include/ \
		--with-iconv-dir=/usr/local/rely/libiconv \
		--enable-short-tags \
		--enable-mbstring \
		--enable-sockets \
		--enable-soap \
		--enable-mbstring \
		--enable-static \
		--enable-mysqlnd \
		--with-mysqli=/usr/local/mysql/bin/mysql_config \
		--enable-mysqlnd-compression-support \
		--with-curl \
		--with-xsl \
		--enable-ftp \
		--with-libxml \

		[root@localhost ~]$ make -j12 && make install


	'选项详解' # 常见项解释 ./configure --help 查询具体对应信息
	# -----------------------------------------------------------------------|	
		--with-config-file-path=/usr/local/php/etc/     # 指定配置文件目录
		--with-apxs2=/usr/local/apache2/bin/apxs        # 指定 apache 动态模块位置
		--with-libxml-dir=/usr/local/libxml2/           # 指定 libxml 位置
		--with-jpeg-dir=/usr/local/jpeg6/               # 指定 jpeg 位置
		--with-png-dir=/usr/local/libpng/               # 指定 libpng 位置
		--with-freetype-dir=/usr/local/freetype/        # 指定 freetype 位
		--with-mcrypt=/usr/local/libmcrypt/             # 指定 libmcrypt 位置
		--with-mysqli=/usr/local/mysql/bin/mysql_config # 指定 mysqli 位置
		--with-gd                                       # 启用 gd 库 --enable-soap 支持 soap 服务
		--enable-mbstring=all                           # 支持多字节，字符串
		--enable-sockets                                # 支持套接字
		--with-pdo-mysql=/usr/local/mysql               # 启用 mysql 的 pdo 模块支持

```

#### 配置 php
```bash

	'拷贝 php 配置文件'
	# -----------------------------------------------------------------------|	
		# 提前创建配置文件目录
		[root@localhost ~]$ mkdir -p /usr/local/php7.4/etc
		# 将php编译后的配置信息,拷贝到安装目录
		# [root@localhost ~]$ cp php.ini-production /usr/local/php7.4/etc/php.ini
		# [root@localhost ~]$ cp php.ini-production /usr/local/php7.4/lib/
		[root@localhost ~]$ cp php.ini-development php.ini-production /opt/lamp/php7.4/lib/
		[root@localhost ~]$ cp php.ini-development /opt/lamp/php7.4/lib/php.ini
		
```

#### 配置环境变量
```bash
	vi /etc/profile
	export PATH=$PATH:/usr/local/php7.4/bin/
```

#### 其他内容(非必要内容)
##### 配置虚拟主机
```bash
    
    # 取掉 /usr/local/apache2/etc/httpd.confhttpd 配置文件中 
    # Include conf/extra/httpd-vhosts.conf 前面的注释 '#'
    vi extra/httpd-vhosts.conf
    <VirtualHost *:80>
        ServerName demo.com
        DocumentRoot /var/www/demo
        <Directory  "/var/www/demo">
            Options -Indexes +Includes +FollowSymLinks +MultiViews
            #AllowOverride All
            Require all granted
        </Directory>
    </VirtualHost>
    
```
---

### Apache-PHP整合
>​	修改 Apache 配置文件，使其识别*.php 文件，并能通过 php 模块调用 php 进行页面解析

#### 配置 Apache
```bash
	
	'修改 Apache 配置文件' 
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ vim /usr/local/apache2/etc/httpd.conf

	'让.php .html文件可以执行php'
	# -----------------------------------------------------------------------|
		添加如下二行
			AddType application/x-httpd-php .php .html
			AddType application/x-httpd-php-source .phps
		
		定位至DirectoryIndex index.html
			修改为：
				DirectoryIndex index.php index.html

	# 'Apache Rewrite 静态配置'
	# -----------------------------------------------------------------------|
		# Rewirte主要的功能就是实现URL的跳转和隐藏真实地址。
		# 平时帮助我们实现拟静态，拟目录，域名跳转，防止盗链等。
		支持httpd.conf 配置和目录 .htaccess配置
	    # 启用rewrite 
		    取掉 LoadModule :/ 前面的注释 ‘#’
	    # AllowOverride 参数和访问控制
		    修改 AllowOverride none 为All 此版本的apache在最后一个 
	    
			    <Directory />
			        AllowOverride All
			        Require all denied
			    </Directory>

	# '修改守护进程'
	# -----------------------------------------------------------------------|
		修改apache的进程守护者 (找到 User 将用户和组的名称改为 创建的apache用户)
			User apache
			Group apache

	'重启 Apache 服务'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ /usr/local/apache2/bin/apachectl restart
		
```

#### 配置 PHP

##### 安装 openssl 模块
>​	`OpenSSL 是一个强大的安全套接字层密码库`，囊括主要的密码算法、常用的密钥和证书封装管理功能 及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。

```bash

	'安装依赖库'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ yum -y install autoconf

	'安装 openssl 模块'
	# -----------------------------------------------------------------------|
		# openssl 模块在 php 下载目录
			[root@localhost ~]$ cd /opt/php-7.4.7/ext/openssl
		# 修改文件名
			[root@localhost ~]$ mv config0.m4 config.m4 
		# 调用已装好的 PHP 解析器,将所需文件调用进当前目录
			[root@localhost ~]$ /usr/local/php7.4/bin/phpize 
		# 编译安装 openssl 模块
			[root@localhost ~]$ ./configure --with-openssl \
					--with-php-config=/usr/local/php7.4/bin/php-config 
			[root@localhost ~]$ make && make install

```

##### 安装 memcache 模块
>​	`Memcache 是一个高性能的分布式的内存对象缓存(高速缓存)系统`，通过在内存里维护一个统一的巨大的hash 表，它能够用来存储各种格式的数据，包括图像、视频、文件以及数据库检索的结果等。简单的说就是将数据调用到内存中，然后从内存中读取，从而大大提高读取速度。

```bash

	'安装依赖库'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ yum -y install libevent autoconf

	'安装 memcache'
	# -----------------------------------------------------------------------|
	# 根据网上资料 php7 中 memcache 模块并不能完美支持,且yum源上未能找需要手动下载安装.
	# 特别需注意所要下载的 memcache 模块所对应的PHP版本 本次选用 memcache-4.0.5.2 
		下载:
			[root@localhost ~]$ wget https://pecl.php.net/get/memcache-4.0.5.2.tgz
			[root@localhost ~]$ tar -xf memcache-4.0.5.2.tar.gz && cd memcache-4.0.5.2
		编译安装
			[root@localhost ~]$ /usr/local/php7.4/bin/phpize
			[root@localhost ~]$ ./configure --with-php-config=/usr/local/php7.4/bin/php-config
			[root@localhost ~]$ make && make install		
		'安装完毕注意保持 memcache 所提供的模块地址'
			/usr/local/php7.4/lib/php/extensions/no-debug-zts-20190902/

```

##### 修改 php 配置文件
>​	使其识别并调用 openssl 和 memcache 两个模块

```bash

	'配置模块'
	# -----------------------------------------------------------------------|
		[root@localhost ~]$ vim /usr/local/php7.4/etc/php.ini
		# 取消分号注释,并添加以上路径(此路径来自于模块安装命令的结果)
			extension_dir="/usr/local/php7.4/lib/php/extensions/no-debug-zts-20190902/"
		# 添加以上两个库文件的调用	 
			extension=openssl.so;
			extension=memcache.so;

	'其他内容'
 	# -----------------------------------------------------------------------|
		php -m # 查看扩展方法
		php --ri memcache#组件详细配置信息

	# 重启 apache，刷新 phpinfo 页面，并查看是否有两个新增的模块
	# httpd restart
	
```

##### 安装 memcached 服务

```bash

	'安装依赖'
 	# -----------------------------------------------------------------------|
		[root@localhost ~]$ yum -y install libevent-devel

	'memcache 服务安装'
 	# -----------------------------------------------------------------------|
	 	wget https://github.com/php-memcached-dev/php-memcached/releases
		wget https://github.com/php-memcached-dev/php-memcached/archive/refs/tags/v3.2.0.tar.gz
		[root@localhost ~]$ cd /opt/memcached-1.4.17
		[root@localhost ~]$ ./configure --prefix=/usr/local/rely/memcache && make && make install
		# 添加 memcache 用户,此用户不用登陆,不设置密码
			[root@localhost ~]$ useradd -r -s /sbin/nologin memcache
		# 启动memcache 服务,并设置后台运行
			[root@localhost ~]$ /usr/local/rely/memcache/bin/memcached -umemcache &
		# 检查memcache是否正常启动,并监听了1121端口 
			[root@localhost ~]$ netstat -antp | grep :11211
		
```

##### 安装 phpMyAdmin (非必要内容)
>​	phpMyAdmin 是一个以 PHP 为基础，`以 Web-Base 方式架构在网站主机上的 MySQL 的数据库管理工具`，让管理者可用 Web 接口管理 MySQL 数据库。

```bash

	'phpMyAdmin 下载安装'
 	# -----------------------------------------------------------------------|		
		wget https://files.phpmyadmin.net/phpMyAdmin/5.1.3/phpMyAdmin-5.1.3-all-languages.tar.gz
		[root@localhost ~]$ tar -xf phpMyAdmin-5.1.3-all-languages.tar.gz
		[root@localhost ~]$ cp -a /opt/phpMyAdmin-5.1.3-all-languages /usr/local/apache2/htdocs/phpmyadmin

	'phpMyAdmin 服务安装'
 	# -----------------------------------------------------------------------|
        [root@localhost ~]$ cd /usr/local/apache2/htdocs/phpmyadmin
        [root@localhost ~]$ cp -a config.sample.inc.php config.inc.php # 将模板文件改为配置文件并修改
        [root@localhost ~]$ vim config.inc.php
        	# 找到如下行在其下方
            $cfg['Servers'][$i]['auth_type'] = 'cookie';
            # 添加如下行
            $cfg['Servers'][$i]['auth_type'] = 'http';

	'创建sock连接'
 	# -----------------------------------------------------------------------|
 	# 由于配置的 mysql.sock 地址与 php 所要求的 /tmp//mysql.sock 不一致.所以创建一条链接.
		ln -s /usr/local/mysql/tmp/mysql.sock /tmp/


	# 通过浏览器输入地址访问：`http://Apache服务器地址/phpmyadmin/index.php`
	# 用户名为 root ，密码为 MySQL 设置时指定的 root 密码 123456
	
```

#### 测试 Apache-php连通性
```bash

	'测试流程'
 	# -----------------------------------------------------------------------|
 		# 编写phpinfo()测试页面
        [root@localhost ~]$ vim /usr/local/apache2/htdocs/test.php
        <?php
            phpinfo();
        ?>
        通过浏览器输入地址访问：http://Apache服务器地址/test.php 测试	
        
```

>出现测试页面 `phpinfo() 没有 memcache 模块加载` 的问题,有两种可能性:
>​	1. 第一种情况是由于 php.ini 配置文件分别存放在 /usr/local/php7.4/etc/php.ini /usr/local/php7.4/lib/php.ini 所导致的.
>​		解决办法: # 创建链接,是两个配置文件相匹配
>​			[root@x ~]# ln -s /usr/local/php/etc/php.ini /usr/local/php/lib/php.ini
>
>
>
>	2. 第二种情况可能是由于 `php-pecl-memcache.x86_64` 这个安装rpm包所引起
>​		解决办法: # 删除rpm包 重启httpd
>​			[root@x ~]# yum remove php-pecl-memcache.x86_64 

#### 设置开机自启
>借助系统自带脚本`/etc/rc.local`，此脚本开机后会自动加载，我们可以将源码安装的服务启动命令写入该脚本，间接实现开机自启动

```bash

	[root@localhost ~]$ vim /etc/rc.local
		/usr/local/apache2/bin/apachectl start
		/usr/local/mysql/bin/mysqld_safe --user=mysql &
		/usr/local/rely/memcache/bin/memcached -umemcache &
		
```

#### 项目迁移
```bash

	1.  把 php 项目拷贝到网站默认目录下：`/usr/local/apache2/htdocs/**`
	2.  使用 phpMyAdmin 创建网站所需数据库  
	    **`注意事项：注意目录权限和归属，防止权限过大或者权限过小`**  
	    **`切记：做完 LAMP 环境后保存一个快照，后面 Apache 要使用！`**
	    
```

---

**至此 LAMP 手动搭建的流程手册算是完成.编译方式主要参考引用了如下作者所写文章.**

​	[源码编译安装lamp环境 (apache24+mysql57+php7.3) - hflxhn's space](http://hflxhn.com/art/9)

​	[web 平台搭建-LAMP-源码包（CentOS-7）_路人甲_passerby的博客-CSDN博客](https://blog.csdn.net/w918589859/article/details/109499919)
