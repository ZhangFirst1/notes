## 0. 用户管理

#### 创建用户

```
useradd 用户名
useradd -d 指定目录 用户名
```

#### 修改密码

```
passwd 用户名
```

#### 删除用户

```
userdel 用户名		# 删除用户，但保留家目录
userdel -r 用户名 	# 删除用户及用户家目录
```

#### 查询用户信息

```
id 用户名
```

#### 切换用户

```
su - 用户名	# 权限低到高需要输入密码，反之不需要
```

#### 查看当前用户

```
whoami 或 who am i
```



## 1. 运行级别

#### 七种运行级别

```
0 halt 关机，代表系统停机状态，默认情况下，系统运行级别不能设置为0，否则电脑一开机就进入关机模式，电脑将不能正常启动
1 Single user mode 单用户模式，只支持root账户，主要用于系统维护，禁止远程登陆，类似于Windows下的安全模式
2 Multiuser，without NFS 它是多用户模式，没有网络文件系统支持
3 Full Multiuser mode 完全多用户模式，有网络文件系统，用户登录后进入控制台命令行模式，在没有网络的环境下等同于运行级别2
4 unused 系统未使用，用作保留，一般不用，在一些特殊情况下可以用它来做一些事情，例如：在笔记本电脑的电池用尽时，可以切换到这一模式来做一些设置
5 X11 图形界面的多用户模式用户登录后直接进入X-Window系统
6 Reboot 重启，默认情况下，运行级别不能设为6，否则电脑一开机就进入重启模式会一直不停地重启，系统将不能正常的启动
```

#### 运行级别

```
查看当前系统运行级别：systemctl get-default 或 runlevel	//二者返回不同 前者如graphical.target 后者如N 5
设置开机时运行级别3：systemctl set-default multi-user.target
设置开机时运行级别5：systemctl set-default graphical.target
```

#### 关机命令

```
init 0	//关机，调用系统的0级别
halt	//关机
poweroff	//关机
shutdown -h 0 //等同于shutdown -h now
shutdown -h +15 //15分钟后关机
```

#### 重启命令

```
init 6
reboot
shutdown -r 0
shutdown -r +15
shutdown -r 16:30	//16：30重启 占用前台
shutdown -r 16:30&	//16：30重启 占用后台
```

#### 取消shutdown

```
shutdown -c
```



## 2. 文件目录

#### 显示路径

```
pwd	# 显示当前工作目录的绝对路径
ls [选项] [目录或文件]	
# 常用：-a 显示所有文件和目录，包括隐藏的
# 	   -l 以列表的方式显示信息
#		* 查找当前目录中*后字符的文件
```

#### 切换路径

```
cd [dirName]
cd /home	# 切换到/home目录
cd ~ 		# 切换到自己的home目录
cd ../..	# 切换到当前目的上上两层
```

#### 创建目录

```
mkdir /home/dog		# 创建单级目录
mkdir -p /home/animal/tiger		# 创建多级目录

mkdir -p {foo,bar}/{1..5}	# 创建foo/1 foo/2 ... bar/5
```

#### 删除目录

```
rmdir /home/dog		# 删除目录 如此只可以删除空目录
rm -rf /home/animal	# 删除整个目录 f表示不经过确认
rm -rf /			# 删库跑路（bushi）
```

#### 创建空文件

```
touch /home/hello.txt # 在/home目录下创建hello.txt的空文件
```

#### 拷贝

```
cp [options] source dest
常用：-r 递归复制整个文件夹 
若有多个文件 不希望依次确认 可以用：
\cp -r /home/bbb /opt/
```

#### 重命名与移动文件

```
mv oldName newName	# 重命名
mv /temp/movefile /targetfolder # 移动文件
```

#### 查看文件

```
cat [选项] 文件		# cat只能查看 无法修改
常用：-n 显示行号
	 | more 以全屏的形式按页显示文本
cat -n /etc/profile | more 

less 指令
	与more类似，但不是一次性加载整个文件，而是根据显示加载需要的内容
空白键/pgup/pudn：翻页
/字串	向下搜索【字串】，n：向下查找 N:向上查找
?字串	向上搜索【字串】
q：退出
```

#### 显示与输出

```
echo：输出内容到控制台
输出环境变量：echo $HOSTNAME
echo "$foo"		# 输出$foo的值
echo '$foo'		# 输出字符串$foo

head：显示文件开头部分内容，默认显示前十行
head -n 5 fileName	# 查看文件前五行内容

tail：显示文件尾部内容，默认十行 用法同head
tail -1n  			# 显示最后一行
tail -f fileName	# 实时追踪文件内容的更新

> 输出重定向和 >> 追加
常用：
ls -l > fileName	# 列表的内容写入文件（覆盖）
ls -al >> fileName	# 列表的内容追加到文件结尾
cat 文件1 > 文件2	 # 文件1的内容覆盖到文件2
echo "内容" >> fileName	# 内容追加到文件中

tee：读取标准输入的数据，并将其内容输出成文件
tee [option] [file]
	-a	追加而非覆盖
	-i	忽略中断信号
ls -l | tee -a ls.log	# 同时打印到屏幕和文件里
```

#### 链接

```
ln指令：软连接（符号链接），存放了链接其他文件的路径
ln -s [源文件或目录] [软连接名] 
如：在/home目录下创建一个软连接myroot，链接到/root目录
ln -s /root /home/myroot
rm /home/myroot	# 删除链接
```



## 3. 时间日期

#### date指令

```
cal 		# 显示日历
date		# 显示当前时间
date +%Y	# 显示当前年份
date +%m	# 月份
date +%d	# 日期
date "+%Y-%m-%d %H:%M:%S"	# 年月日时分秒
date -s 字符串时间	# 设置时间
date -s "2023-1-10 15:16:00"
```



## 4. 查找指令

#### find指令

```
find [搜索范围] [选项]
指令从指定目录向下递归地遍历其各个子目录，将目标显示在终端
1.find /home -name hello.txt	# 查找/home目录下的hello.txt文件
2.find /opt -user nobody		# 查找/opt目录下，用户名为nobody的文件
3.find / -size +200M			# 查找所有大于200M的文件
```

#### locate指令

```
	locate可以快速定位文件路径，使用事先建立的系统中所有文件名称及路径的locate数据库实现快速定位给给定的文件。需要定期更新locate时刻。
	第一次运行前，需使用updatedb创建数据库
locate [fileName]
```

#### which

```
which 指令名称	# 查看某个指令在哪个目录下
如：which ls
```

#### gerp指令和管道符号|

```
grep过滤查找，管道符“|”表示将前一个命令的处理结果输出传递给后面的命令处理。
grep [选项] 查找内容 源文件
选项：-n	显示匹配行及行号
	 -i	  忽略字母大小写
cat /home/hello.txt | grep -n "yes" # 在/home/hello.txt中查找yes并返回行号
grep -n "yes" /home/hello.txt		# 同上
```

#### sed命令

```
利用脚本处理文本文件
sed [选项] 文本文件
a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
d ：删除，因为是删除啊，所以 d 后面通常不接任何东东；
i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
p ：打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
s ：取代，可以直接进行取代的工作

sed '2,$d'	# 2-最后一行行删除
sed '2a drink tea' # 第二行后加入drink tea
sed 's/被取代的字符串/新的字符串/g'
```



## 5. 压缩和解压缩

#### gzip/gunzip(单个文件)

```
gzip 文件		# 压缩文件，只能压缩为*.gz文件
gunzip 文件	# 解压文件
gzip -d *.gx	# 解压文件
gzip -c 文件名 > 压缩后名称	# 压缩并保留源文件且重命名
```

#### zip和unzip(用于目录)

```
zip [选项] xxx.zip [fileName]	
选项：-r	递归压缩，即压缩目录
unzip [选项] xxx.zip	
选项：-d[目录]	指定解压后的文件目录
```

#### tar

```
tar [选项] xxx.tar.gz 打包内容
选项：-c	产生.tar打包文件
	 -v	  显示详细信息
	 -f	  指定压缩后的文件名
	 -z	  打包同时压缩
	 -x	  解包.tar文件
tar -zcvf pc.tar.gz /home/a.txt /home/b.txt	# 将a和b压缩成pc.tar.gz
tar -zxvf /home/pc.tar.gz -c /home			# 将pc.tar.gz解压到/home
# 注意 f后面要接文件名 所以f应该放在选项的最后边
# 若使用z（gzip支持）文件名最好为*.tar.gz
# 使用j（bzip2支持）文件名最好为 *.tar.bz2
```



## 6. 组

1. Linux中每个用户必须属于一个组，不能独立于组外。

2. Linux中每个文件有所有者、所在组、其他组的概念，其他组与所在组对文件有不同的权限。

- ​	所有者：初始为文件创建人，也可以通过`chown 用户名 文件名`改变所有者

- ​	所在组：创建人所在的组，可以用过`chgrp 组名 文件名`修改文件所在的组

- ​	其他组：除文件的所有者和所在组的用户外，系统其他用户都是文件的其他组

  ​	`usermod -g 新组名 用户名`改变用户所在组，若`-g`则改变用户初始登陆组（用户需要有权限）

#### 组的创建

```
groupadd 组名			# 创建一个组
cat /etc/group		 # 查看所有组
useradd -g 组名 用户名 # 创建一个用户 并放到指定组中
groupdel 组名			# 删除一个组
getent group | grep 组名	# 验证是否成功删除
id 用户名				# 查找用户相关信息
```

#### 修改用户组

```
usermod -g 用户组 用户名
```



## 7. 权限

ll显示内容：drwxr-xr-x. 2 root root 4096 1月   8 16:40 公共

1. 第0位确定文件类型（d, -, l, c, b）

   ```
   l：链接，相当于快捷方式
   d：目录，相当于文件夹
   -：普通文件
   c：字符设备文件，如鼠标键盘
   b：块设备，如硬盘
   ```

​	2.第1-3位确定所有者权限

​	3.第4-6位确定所属组权限

​	4.7-9位确定其他组的权限

```
rwx作用到文件：
	1.【r】可读，可以读取，查看
	2.【w】可写，可以修改，但未必可以删除，需要对文件所在的目录有可写权限
	3.【x】可执行
rwx作用到目录：
	1.【r】可读：可以读取，ls查看
	2.【w】可写：可以修改，对目录内创建、删除、重命名
	3.【x】可执行：可以进入该目录
如果有某个文件的写入权限，但没有文件所在目录的写入权限，则不能删除该文件

```

#### 修改权限

```
第一种：+ - =变更权限
u：所有者 g：所有组 o：其他人 a：所有人
chmod u=rwx,g=rx,o=rx abc	# 给abc文件所有者读写执行，所在组读执行，其他组读执行
chmod u-x,g+w				# 给abc所有者去除执行，增加组写

第二种：通过数字变更权限
r：4 w：2 x:1
chmod 755 abc				# 给abc文件所有者读写执行，所在组读执行，其他组读执行
```

#### 修改文件所有者/所在组

```
修改所有者：
chown newowner 文件/目录
chown newowner:newGroup 文件/目录
如:chown zy /home/a.txt
   chown zy -r zy /home/test	# 将/home/test 目录下所有的文件和目录所有者修改为zy 

修改所在组：
group newGroup 文件/目录
# 用法同上
```

#### 配置文件隐藏属性

```
chattr [+-=] [ASacdistu]

lsattr [-adR] 文件或目录		# 查看隐藏属性
-a:隐藏文件的属性也显示出来
-d:因列出目录本身的属性而非目录内的文件名
-R:连同子目录一起
```



## 8. crond

```
设置个人任务调度，执行crontab -e 
输入任务到调度文件，如* /1 * * * * ls -l /etc/ > /tmp/to.txt	# 每分钟执行一次

crontab -r	# 终止任务调度
crontab -l	# 列出当前所有任务调度
service crond restart	# 重启任务调度
```

| 项目      | 含义               | 范围                |
| --------- | ------------------ | ------------------- |
| 第一个'*' | 一小时中的第几分钟 | 0-59                |
| 第二个'*' | 一天中的第几小时   | 0-23                |
| 第三个'*' | 一月中的第几天     | 1-31                |
| 第四个'*' | 一年中的第几月     | 1-12                |
| 第五个'*' | 一周中的星期几     | 0-7(0和7都为星期天) |

| 特殊符号 |                             含义                             |
| :------: | :----------------------------------------------------------: |
|    *     |      代表任何时间，如第一个*表示一小时中每分钟执行一次       |
|    ,     | 代表不连续时间，如"0 8,12 * * *"表示每天的8点，12点各执行一次 |
|    -     |  代表连续时间，如"0 5 * * 1-6"表示周一到周六凌晨5点执行一次  |
|   */n    | 代表每隔多久执行一次，如"*/10 * * * *"表示每隔10分钟执行一次 |

#### at定时任务

```
at [选项] [日期时间]	# at命令指定的任务指挥执行一次
-f：指定包含具体指令的任务文件
-q：指定新任务的队列名称
-l：显示待执行任务的列表
-d：删除指定的待执行任务
-m：任务执行完成后向用户发送 E-mail

需要启动at守护进程
service atd start 
启动后atd会60秒检测一次是否到达任务时间

atq				# 查看任务
atrm [任务编号]	 # 删除任务

如： at 5pm + 2 days	# 两天后的下午五点执行ls /home
	at> ls /home	
at now + 2 minutes  # 两分钟后将date写入log
at> date > /root/mydate.log	
```



## 9. 硬盘操作

#### Linux分区

1. **Linux 是通过目录树的方式来管理文件的**。每个分区都是组成整个文件系统的一部分。

2. 采用**“载入”**的处理方法，整个文件系统包含了一整套的文件和目录，且讲一个分区与一个目录联系起来。要载入的一个分区将使它的存储空间在一个目录下获得。

#### 硬盘说明

1. Linux硬盘分**IDE硬盘**和**SCSI硬盘**，主流SCSI硬盘。

2. 对于IDE硬盘，**“hdx~”**，“hd”表示分区所在设备的类型（SCSI为sd）“x”为盘号（a为基本盘，b为基本从属盘，c为辅助主盘，d为辅助从属盘），“~”表示分区，前四个分区用1~4表示，他们是主分区或扩展分区，从5开始是逻辑分区。

3. 查看所有设备挂载情况`lsblk -f`

4. 增加一块硬盘

   ```
   1.虚拟机添加硬盘：虚拟机设置种添加硬盘，重启系统。
   2.分区：fdisk /dev/sdb
   	m: 显示命令列表
   	p: 显示磁盘分区，同fdisk -l
   	n: 新增分区
   	d: 删除分区
   	w: 写入并退出
   3.格式化：mkfs -t ext4 /dev/sdb1	# ext4是分区类型
   4.挂载：mount 设备名称 挂载目录
   	如：mount /dev/sdb1 /newdisk
     卸载：umount 设备名称 或 挂载目录
   5.永久挂载：修改/etc/fstab文件
   ```

5. 查询磁盘使用情况
   ```
   1.查询系统整体磁盘使用情况：df -h
   2.查询指定目录磁盘使用情况：du -h
   	-s：指定目录占用大小汇总
   	-h：带计量单位
   	-a：含文件
   	--max-depth=1：子目录深度
   	-c：列出明细的同时，总价汇总值
   如：统计/opt问价下文件的个数
   	ls -l/opt | grep "^-" | wc -l
   	统计/opt文件下目录的个数，包括子文件夹
   	ls -lR /opt | grep "^d" | wc -l
   3.tree 目录
   4.blkid 	# 显示设备的UUID（全局唯一标识符）
   ```



## 10. 进程

#### ps：查看当前系统中哪些程序正在执行以及执行状况

```
-a: 显示当前终端所有进程信息（执行）
-e: 显示所有进程（无论是否执行）
-f: 全格式显示
-u: 以用户的格式显示
-x: 显示后台进程运行的参数
-w: 显示加宽可以显示较多的资讯
ps -aux | grep xxx	# 查找xxx服务

USER: 行程拥有者
PID: pid
PPID: 父进程
%CPU: 占用的 CPU 使用率
%MEM: 占用的记忆体使用率
VSZ: 占用的虚拟记忆体大小
RSS: 占用的记忆体大小
TTY: 终端的次要装置号码 (minor device number of tty)
STAT: 该行程的状态:
	D: 无法中断的休眠状态 (通常 IO 的进程)
	R: 正在执行中	S: 静止状态
	T: 暂停执行		 Z: 不存在但暂时无法消除
    W: 没有足够的记忆体分页可分配
    <: 高优先序的行程
    N: 低优先序的行程
    L: 有记忆体分页分配并锁在记忆体内 (实时系统或捱A I/O)
START: 行程开始时间
TIME: 执行的时间
COMMAND:所执行的指令

pstree [选项]		# 以树形查看进程
-p：显示pid
-u：显示进程所属用户
```

#### kill和killall

```
kill [选项] 进程号	
killall 进程名称	# 终止进程和所有子进程
-9：表示强迫进程立即停止

如：种植远程登录服务sshd，并重启
kill sshd 进程号 	# 进程号通过ps -aux | grep sshd 查询
/bin/systemctl start sshd.service	# 重启进程
```



## 11. 服务管理

#### chkconfig指令

​	可以给服务的各个运行级别设置自启动或关闭	chkconfig指令管理的服务在/etc/init.d 查看

```
1.查看服务 chkconfig --list [| grep xxx]
2.chkconfig 服务名 --list
3.chkconfig --level 5 服务名 on/off
如：chkconfig --level 3 network off	# 把network在3运行级别关闭自启动
```

#### systemctl指令

​	指令管理在/usr/lib/systemd/system 查看

```
systemctl [start/stop/restart/status] 服务名
设置服务的自启动状态
systemctl list-unit-files [| grep 服务名]	# 查看服务开机启动状态，grep进行过滤
systemctl enable 服务名	# 设置服务开机启动
systemctl disable 服务名	# 关闭服务开机启动
systemctl is-enable 服务名	# 查询某个服务启动状态
如：查看当前防火墙状态，关闭和重启防火墙
systemctl status firewalld	
systemctl stop firewalld
systemctl start firewalld
若希望永久生效 systemctl [enable/disable] 服务名
```

#### firewall指令

```
打开端口：firewall-cmd --permanent --add-port=端口号/协议
关闭端口：firewall-cmd --permanent --remove-port=端口号/协议
重新载入才能生效：firewall-cmd --reload
查询端口是否开放；firewall-cmd --query-port-端口/协议
```

#### 动态监控进程

 ```
 top [选项]
 -d 秒数	# top命令每隔几秒更新，默认3秒
 -i		  # 不显示任何限制或僵死进程
 -p		  # 指定监控进程ID来仅监控某个进程状态
 交互操作：
 P：以cpu使用率排序，默认如此
 M：以内存使用率排序
 N：以PID排序
 u：监控用户
 k：终止进程
 q：退出top
 ```

#### 监控网络状况

```
netstat [选项]
-an 按一定顺序排列输出
-p 显示哪个进程在调用
```



## 12. rpm

​	rpm用于互联网下载包的打包及安装工具，类似windows的setup.exe

```
rpm -qa | grep xxx	# 查询当前系统是否安装了xxx
rpm -q firefox		# 查询火狐是否安装
rpm -qi xxx			# 查询xxx软件包信息
rpm -ql xxx			# 查询安装路径
rpm -qf xxx			# 查询文件所属的安装包
rpm -e 软件包		  # 删除 在-e后增加 --nodeps可强制删除

安装rpm包
rpm -ivh rpm包全路径名称
-i	安装
-v	提示
-h	进度条
```

#### yum

​	是一个shell前端软件包管理器，基于rpm包管理，能够从指定的服务器自动下载RPM包并安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包。

```
yum list | grep 软件列表	# 查询yum服务器是否有需要安装的软件
yum install xxx	下载安装    # 安装指定的yum包
yum换源参考：				 
https://blog.csdn.net/qq_18297675/article/details/52705288
```



## 12. shell

​	shell是一个命令行解释器，为用户提供了一个向linux内核发送请求以便运行程序的界面系统级程序，用户可以用shell启动、挂起、停止、编写一些程序。常用bash。

​	格式要求：1.以#!/bin/bash开头	2.要有可执行权限。

#### 变量

​	分为**系统变量**和**用户自定义变量**

```
1.系统变量：如$HOME $PWD $SHELL $USER等
显示当前shell中所有变量：set
2.shell变量定义
    基本语法：变量名=值
    "" 内的特殊字符如$可以保留原本的特性
    '' 内的特殊字符仅为一般字符
    撤销变量：unset 变量
    声明静态变量：readonly变量（不能被unset）
3.变量命名规则
	(1)变量名可由字母、数字、下划线组成，不能以数字开头
	(2)等号两侧不能有空格
	(3)变量名称一般习惯为大写
4.将命令的返回值赋给变量
	(1)A=`date`	# 运行反引号种的命令，并把返回值给A
	(2)A=$(date)	# 同上
5.设置环境变量
	(1)export 变量名=变量值	# 将shell变量输出为环境/全局变量
	(2)source 配置文件		 # 修改后的配置信息立即生效
	(3)echo $变量名		  # 查询环境变量的值
6.多行注释
	:<<! 
	内容 
	!
7.预定义变量
	(1)$$	# 当前进程的进程号（PID）
	(2)$!	# 后台运行的最后一个进程的进程号(PID)
	(3)$?	# 最后一次执行命令的返回状态。为0则正确执行，非0则不正确
```

#### 位置参数

```
向shell脚本种传递参数的占位符
$n	# $0代表本身 $1-9代表第1到9个参数 10以上参数用${10}
$*	# 代表命令行种所有参数，$*把所有参数看为一个整体
$@	# 命令行中所有参数，把每个参数区分对待
$#	# 命令行中所有参数个数
```

#### 运算符

```
1."$((运算式))" 或 "$[运算式]" 或 expr 运算式(使用"反引号将结果赋值给变量,运算式之间要有空格)
2.expr \* / %	乘 除 取余

TEMP='expr 2 + 3'	# 将2+3的值赋给TEMP
SUM=$[$1+$2]		# 将两个参数之和赋给给sum
```

#### 条件判断

```
1.[ condition ] #两侧有空格 非空返回true（true为0，>1为false）
2.常用判断语句
	1）=		字符串比较
	2）两个整数比较
		-lt	小于
		-le	小于等于
		-eq	等于
		-gt	大于
		-ge	大于等于
		-ne	不等于
	3）文件权限判断
		-r	有读的权限	-w 有写的权限	-x 有执行的权限
	4）文件类型判断
		-f 文件存在且是一个常规文件
		-e 文件存在
		-d 文件存在且是一个目录
```

```
if基本语法：

if [ 条件判断 ]
then
	代码1
elif [ 条件判断 ]
then
	代码2
else
then
	其他代码
fi
```

#### 流程控制

```
case语句基本语法：

case $变量名 in
"值1")
	语句1
;;
"值2")
	语句2
;;
*)
	其他语句
;;
esac
```

```
for语句：

for 变量 in 值1 值2 值3...
do
	代码
done

for (( 初始值;循环控制条件;变量变化))
do
	代码
done
```

```
while语法：
while [ 条件判断 ]	# 注意空格
do
	代码
done
```

#### 其他函数

```
alias cls='clear'	# 替换指令
```

```
read读取控制台输入
read (选项) (参数)
选项：
	-p	指定读取值时的提示符
	-t	指定等待时间（秒）
read -t 10 -p "请输入一个数NUM1" NUM1		# 读取控制台输入一个NUM1值，在10秒内输入
```

```
basename：返回完整路径最后 / 的部分，常用于获取文件名
basename [string] [suffix] # 删掉所有前缀，包括最后一个/字符，然后显示字符串

basename /home/test.txt .txt	# 返回test

dirname:返回路径最后 / 前面的部分
dirname 文件绝对路径
```

```
自定义函数：
function funname()
{
	action;
	return int;
}
funname 值	# 调用函数
```

```
ulimit [-SHacdfltu] [配额]	# 限制用户的某些系统资源
```



## 13. 日志

[linux系统下各种日志文件的介绍，查看，及日志服务配置 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/298335887)



## 14. 备份 

![image-20230129110409620](D:\typora\图片\image-20230129110409620.png)

````
dump 支持分卷和增量备份(只有分区才支持增量备份)
dump [-cnu][-0123456789][-b <区块大小>][-B <区块数目>][-d <密度>][-f <设备名称>][-h <层级>][-s <磁带长度>][-T <日期>][目录或文件系统] 或 dump [-wW]

-0123456789 　备份的层级。
-b<区块大小> 　指定区块的大小，单位为KB。
-B<区块数目> 　指定备份卷册的区块数目。
-c 　修改备份磁带预设的密度与容量。
-d<密度> 　设置磁带的密度。单位为BPI。
-f<设备名称> 　指定备份设备。
-h<层级> 　当备份层级等于或大于指定的层级时，将不备份用户标示为"nodump"的文件。
-n 　当备份工作需要管理员介入时，向所有"operator"群组中的使用者发出通知。
-s<磁带长度> 　备份磁带的长度，单位为英尺。
-T<日期> 　指定开始备份的时间与日期。
-u 　备份完毕后，在/etc/dumpdates中记录备份的文件系统，层级，日期与时间等。
-w 　与-W类似，但仅显示需要备份的文件。
-W 　显示需要备份的文件及其最后一次备份的层级，时间与日期。
如：
dump -0uj -f /opt/boot.bak0.bz2 /boot	# 在/boot目录下增加备份/opt/boot.bak0.bz2
````

```
restore [模式选项] [选项]

-C 使用对比模式，将备份的文件与现行的文件相互对比。
-i 使用互动模式，在进行还原操作时，restore指令将依序询问用户。
-r 进行还原操作。
-t 指定文件名称，若该文件已存在备份文件中，则列出它们的名称。
-y 不询问任何问题，一律以同意回答并继续执行指令。

如：
restore -C -f boot.bak0.bz2	# 与最新文件比较
如果有增量备份，需要把增量文件按顺序依次恢复
restore -r -f /opt/bootbak0.bz2	# 恢复到第一次完全备份状态
restore -r -f /opt/bootbak1.bz2 # 恢复到第二次增量备份状态
```

 

## 15.Vim

![vi-vim-cheat-sheet-sch](D:\typora\图片\vi-vim-cheat-sheet-sch.gif)

**多窗口功能：**

```
:sp [filename] 	# 打开一个新窗口
[ctrl + w] + j/k  # 切换窗口
[ctrl + w] + q	  # 关闭窗口
```





## 16.Git

[git 简明指南 (runoob.com)](https://www.runoob.com/manual/git-guide/)

[Git 教程 | 菜鸟教程 (runoob.com)](https://www.runoob.com/git/git-tutorial.html)

