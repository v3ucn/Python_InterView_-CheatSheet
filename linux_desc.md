# linux命令一定得结合场景，不能列流水账

一般情况下，我会使用ssh(ssh user@host)命令来登录公司的服务器或者Docker容器，我主要的日常工作是对于日志的一些操作，比如针对日志内容进行检索的grep(-i)命令，还有针对日志名称进行检索的find(-name)命令，有时候对两份日志进行简单的比对操作使用diff命令，使用tail(-f)命令对实时更新的日志进行监控，cat命令打印详细日志等等。

另外，对于系统的进程和资源占用情况，我也经常关注，比如 ps(-aux | grep)命令查看所有运行的进程，free命令查看内存占用，top命令查看资源占用最多的进程，df(-h)命令查看磁盘占用，如果有些进程出现内存溢出的情况或者假死，我也会手动使用kill(-9)命令将其杀死，最后，对于一些年代久远的日志文件我们会对其进行定期删除，一般公司会用rm命令，但是我们公司对于这种危险操作是非常慎重的(您听说过删库跑路吧？那就是对于危险命令没有进行权限限制导致的)，普通的QA级别账号是无法进行rm操作的，所以我会用mv命令将其移动到指定的文件夹，再通过crontab命令对其进行定期删除，基本上，这就是我日常使用的一些命令。


## 重要

1、top：查看内存/显示系统当前进程信息

2、df -h：查看磁盘储存状况 (如果出现磁盘报警和实际有出入，可能因为文件操作符或者文件句柄未释放导致)

3、iotop：查看IO读写（yum install iotop安装）

4、iotop -o：直接查看比较高的磁盘读写程序

5、netstat -tunlp | grep 端口号：查看端口号占用情况（1）

6、lsof -i:端口号：查看端口号占用情况（2）

7、uptime：查看报告系统运行时长及平均负载

8、ps aux：查看进程

9、 free 查看内存交换区

## 基础
1、查看目录与文件：ls

ls -la：显示当前目录下所有文件的详细信息

2、切换目录：cd

cd /home 进入 ‘/ home’ 目录

cd … 返回上一级目录

cd …/… 返回上两级目录

3、显示当前目录：pwd

pwd

4、创建空文件：touch

touch desc.txt：在当前目录下创建文件desc.txt

5、创建目录：mkdir

mkdir test：在当前目录下创建test目录

mkdir -p /opt/test/img：在/opt/test目录下创建目录img，若无test目录，先创建test目录

6、查看文件内容：cat

cat desc.txt：查看desc.txt的内容

7、分页查看文件内容：more

more desc.txt：分页查看desc.txt的内容

8、查看文件尾内容：tail

tail -100 desc.txt：查看desc.txt的最后100行内容

9、拷贝：cp

cp desc.txt /mnt/：拷贝desc.txt到/mnt目录下

cp -r test /mnt/：拷贝test目录到/mnt目录下

10、剪切或改名：

mv desc.txt /mnt/：剪切文件desc.txt到目录/mnt下

mv 原名 新名

11、删除：rm

rm -rf test：删除test目录，-r递归删除，-f强制删除。危险操作，务必小心，切记！

12、搜索文件：find

find /opt -name ‘*.txt’：在opt目录下查找以.txt结尾的文件

13、显示或配置网络设备：ifconfig

ifconfig：显示网络设备情况

14、显示网络相关信息：netstat

netstat -a：列出所有端口

netstat -tunlp | grep 端口号：查看进程端口号

15、显示进程状态：ps

ps -ef：显示当前所有进程

ps-ef | grep java：显示当前所有java相关进程

16、查看目录使用情况：du

du -h /opt/test：查看/opt/test目录的磁盘使用情况

17、查看磁盘空间使用情况：df

df -h：查看磁盘空间使用情况

18、显示系统当前进程信息：top

top：显示系统当前进程信息

19、杀死进程：kill

kill -s 9 27810：杀死进程号为27810的进程，强制终止，系统资源无法回收

20、压缩和解压：tar

tar -zcvf test.tar.gz ./test：打包test目录为test.tar.gz文件，-z表示用gzip压缩

tar -zxvf test.tar.gz：解压test.tar.gz文件

21、改变文件或目录的拥有者和组：chown

chown nginx:nginx desc.txt：变更文件desc.txt的拥有者为nginx，用户组为nginx

chown -R nginx:nginx test：变更test及目录下所有文件的拥有者为nginx，用户组为nginx
22、改变文件或目录的访问权限：chmod

chmod u+x test.sh：权限范围：u(拥有者)g(郡组)o(其它用户)， 权限代号：r(读权限/4)w(写权限/2)x(执行权限/1)#给文件拥有者增加test.sh的执行权限

chmod u+x -R test：给文件拥有者增加test目录及其下所有文件的执行权限

23、文本编辑：vim

vim三种模式：命令模式，插入模式，编辑模式。使用ESC或i或：来切换模式。

命令模式下:q退出 :q!强制退出 :wq!保存退出 :set number显示行号 /java在文档中查找java yy复制 p粘贴

vim desc.txt：编辑desc.txt文件

24、关机或重启：shutdown

shutdown -h now：立刻关机

shutdown -r -t 60：60秒后重启

shutdown -r now：重启(1)

reboot：重启(2)

25、帮助命令：man

man ls：查看ls命令的帮助文档

help
## 快捷键

Ctrl + a 光标到开头

Ctrl + c 中断当前程序

Ctrl + d 退出当前窗口或当前用户

Ctrl + e 光标到结尾

Ctrl + l 清屏 相当与clear

Ctrl + u 剪切、删除（光标以前的）内容- - Ctrl + k 剪切、删除（光标以后的）内容- - Ctrl + r 查找（最近用过的命令）

tab 所有路径以及补全命令

Ctrl+shift+c 命令行复制内容

Ctrl+shift+v 命令行粘贴内容

Ctrl + q 取消屏幕锁定

Ctrl + s 执行屏幕锁定