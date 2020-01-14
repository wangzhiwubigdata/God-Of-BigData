
## Linux命令

我是小白，我从来没玩过Linux,请点这里：
```
http://www.runoob.com/linux/Linux-intro.html
```
## 推荐的一个Git仓库

我有些基础，推荐一个快速查询命令的手册，请点这里：
```
https://github.com/jaywcjlove/linux-command
```
## 必须学会的命令

#### 1.man和page


```
1.内部命令：echo
查看内部命令帮助：help echo 或者 man echo

2.外部命令：ls
查看外部命令帮助：ls --help 或者 man ls 或者 info ls

3.man文档的类型(1~9)
man 7 man
man 5 passwd

4.快捷键：
ctrl + c：停止进程

ctrl + l：清屏

ctrl + r：搜索历史命令

ctrl + q：退出

5.善于用tab键

```

#### 2.常用
```
说明：安装linux时，创建一个hadoop用户，然后使用root用户登陆系统

1.进入到用户根目录
cd ~ 或 cd

2.查看当前所在目录
pwd

3.进入到hadoop用户根目录
cd ~hadoop

4.返回到原来目录
cd -

5.返回到上一级目录
cd ..

6.查看hadoop用户根目录下的所有文件
ls -la

7.在根目录下创建一个hadoop的文件夹
mkdir /hadoop

8.在/hadoop目录下创建src和WebRoot两个文件夹
分别创建：mkdir /hadoop/src
		  mkdir /hadoop/WebRoot
同时创建：mkdir /hadoop/{src,WebRoot}

进入到/hadoop目录，在该目录下创建.classpath和README文件
分别创建：touch .classpath
		  touch README
同时创建：touch {.classpath,README}

查看/hadoop目录下面的所有文件
ls -la

在/hadoop目录下面创建一个test.txt文件,同时写入内容"this is test"
echo "this is test" > test.txt

查看一下test.txt的内容
cat test.txt
more test.txt
less test.txt

向README文件追加写入"please read me first"
echo "please read me first" >> README

将test.txt的内容追加到README文件中
cat test.txt >> README

拷贝/hadoop目录下的所有文件到/hadoop-bak
cp -r /hadoop /hadoop-bak

进入到/hadoop-bak目录，将test.txt移动到src目录下，并修改文件名为Student.java
mv test.txt src/Student.java

在src目录下创建一个struts.xml
> struts.xml

删除所有的xml类型的文件
rm -rf *.xml

删除/hadoop-bak目录和下面的所有文件
rm -rf /hadoop-bak

返回到/hadoop目录，查看一下README文件有多单词，多少个少行
wc -w README
wc -l README

返回到根目录，将/hadoop目录先打包，再用gzip压缩
分步完成：tar -cvf hadoop.tar hadoop
		  gzip hadoop.tar
一步完成：tar -zcvf hadoop.tar.gz hadoop
		  
将其解压缩，再取消打包
分步完成：gzip -d hadoop.tar.gz 或 gunzip hadoop.tar.gz
一步完成：tar -zxvf hadoop.tar.gz

将/hadoop目录先打包，同时用bzip2压缩，并保存到/tmp目录下
tar -jcvf /tmp/hadoop.tar.bz2 hadoop

将/tmp/hadoop.tar.bz2解压到/usr目录下面
tar -jxvf hadoop.tar.bz2 -C /usr/

```

2. 文件命令
```
1.进入到用户根目录
cd ~ 或者 cd
cd ~hadoop
回到原来路径
cd -

2.查看文件详情
stat a.txt

3.移动
mv a.txt /ect/
改名
mv b.txt a.txt
移动并改名
mv a.txt ../b.txt

4拷贝并改名
cp a.txt /etc/b.txt

5.vi撤销修改
ctrl + u (undo)
恢复
ctrl + r (redo)

6.名令设置别名(重启后无效)
alias ll="ls -l"
取消
unalias ll

7.如果想让别名重启后仍然有效需要修改
vi ~/.bashrc

8.添加用户
useradd hadoop
passwd hadoop

9创建多个文件
touch a.txt b.txt
touch /home/{a.txt,b.txt}

10.将一个文件的内容复制到里另一个文件中
cat a.txt > b.txt
追加内容
cat a.txt >> b.txt 


11.将a.txt 与b.txt设为其拥有者和其所属同一个组者可写入，但其他以外的人则不可写入:
chmod ug+w,o-w a.txt b.txt

chmod a=wx c.txt

12.将当前目录下的所有文件与子目录皆设为任何人可读取:
chmod -R a+r *

13.将a.txt的用户拥有者设为users,组的拥有者设为jessie:
chown users:jessie a.txt

14.将当前目录下的所有文件与子目录的用户的使用者为lamport,组拥有者皆设为users，
chown -R lamport:users *

15.将所有的java语言程式拷贝至finished子目录中:
cp *.java finished

16.将目前目录及其子目录下所有扩展名是java的文件列出来。
find -name "*.java"
查找当前目录下扩展名是java 的文件
find -name *.java

17.删除当前目录下扩展名是java的文件
rm -f *.java


```

3.系统命令
```
1.查看主机名
hostname

2.修改主机名(重启后无效)
hostname hadoop

3.修改主机名(重启后永久生效)
vi /ect/sysconfig/network

4.修改IP(重启后无效)
ifconfig eth0 192.168.12.22

5.修改IP(重启后永久生效)
vi /etc/sysconfig/network-scripts/ifcfg-eth0

6.查看系统信息
uname -a
uname -r

7.查看ID命令
id -u
id -g

8.日期
date
date +%Y-%m-%d
date +%T
date +%Y-%m-%d" "%T

9.日历
cal 2012

10.查看文件信息
file filename

11.挂载硬盘
mount
umount
加载windows共享
mount -t cifs //192.168.1.100/tools /mnt

12.查看文件大小
du -h
du -ah

13.查看分区
df -h

14.ssh
ssh hadoop@192.168.1.1

15.关机
shutdown -h now /init 0
shutdown -r now /reboot

```

4.用户和组

```
添加一个tom用户，设置它属于users组，并添加注释信息
分步完成：useradd tom
          usermod -g users tom
	      usermod -c "hr tom" tom
一步完成：useradd -g users -c "hr tom" tom

设置tom用户的密码
passwd tom

修改tom用户的登陆名为tomcat
usermod -l tomcat tom

将tomcat添加到sys和root组中
usermod -G sys,root tomcat

查看tomcat的组信息
groups tomcat

添加一个jerry用户并设置密码
useradd jerry
passwd jerry

添加一个交america的组
groupadd america

将jerry添加到america组中
usermod -g america jerry

将tomcat用户从root组和sys组删除
gpasswd -d tomcat root
gpasswd -d tomcat sys

将america组名修改为am
groupmod -n am america

```

5. 权限

```
创建a.txt和b.txt文件，将他们设为其拥有者和所在组可写入，但其他以外的人则不可写入:
chmod ug+w,o-w a.txt b.txt

创建c.txt文件所有人都可以写和执行
chmod a=wx c.txt 或chmod 666 c.txt

将/hadoop目录下的所有文件与子目录皆设为任何人可读取
chmod -R a+r /hadoop

将/hadoop目录下的所有文件与子目录的拥有者设为root，用户拥有组为users
chown -R root:users /hadoop

将当前目录下的所有文件与子目录的用户皆设为hadoop，组设为users
chown -R hadoop:users *

```

6.目录属性

```
1.查看文件夹属性
ls -ld test

2.文件夹的rwx
--x:可以cd进去
r-x:可以cd进去并ls
-wx:可以cd进去并touch，rm自己的文件，并且可以vi其他用户的文件
-wt:可以cd进去并touch，rm自己的文件

ls -ld /tmp
drwxrwxrwt的权限值是1777(sticky)

```

7.软件安装

```
1.安装JDK
	*添加执行权限 
		chmod u+x jdk-6u45-linux-i586.bin
	*解压
		./jdk-6u45-linux-i586.bin
	*在/usr目录下创建java目录
		mkdir /usr/java
	*将/soft目录下的解压的jdk1.6.0_45剪切到/usr/java目录下
		mv jdk1.6.0_45/ /usr/java/
	*添加环境变量
		vim /etc/profile
		*在/etc/profile文件最后添加
			export JAVA_HOME=/usr/java/jdk1.6.0_45
			export CLASSPATH=$JAVA_HOME/lib
			export PATH=$PATH:$JAVA_HOME/bin
	*更新配置
		source /etc/profile
		
2.安装tomcat
	tar -zxvf /soft/apache-tomcat-7.0.47.tar.gz -C /programs/
	cd /programs/apache-tomcat-7.0.47/bin/
	./startup.sh
	
3.安装eclipse
	
		 
```
8.vim
```
i
a/A
o/O
r + ?替换

0:文件当前行的开头
$:文件当前行的末尾
G:文件的最后一行开头
1 + G到第一行 
9 + G到第九行 = :9

dd:删除一行
3dd：删除3行
yy:复制一行
3yy:复制3行
p:粘贴
u:undo
ctrl + r:redo

"a剪切板a
"b剪切板b

"ap粘贴剪切板a的内容

每次进入vi就有行号
vi ~/.vimrc
set nu

:w a.txt另存为
:w >> a.txt内容追加到a.txt

:e!恢复到最初状态

:1,$s/hadoop/root/g 将第一行到追后一行的hadoop替换为root
:1,$s/hadoop/root/c 将第一行到追后一行的hadoop替换为root(有提示)


```

9.查找

```
1.查找可执行的命令：
which ls

2.查找可执行的命令和帮助的位置：
whereis ls

3.查找文件(需要更新库:updatedb)
locate hadoop.txt

4.从某个文件夹开始查找
find / -name "hadooop*"
find / -name "hadooop*" -ls

5.查找并删除
find / -name "hadooop*" -ok rm {} \;
find / -name "hadooop*" -exec rm {} \;

6.查找用户为hadoop的文件
find /usr -user hadoop -ls

7.查找用户为hadoop并且(-a)拥有组为root的文件
find /usr -user hadoop -a -group root -ls

8.查找用户为hadoop或者(-o)拥有组为root并且是文件夹类型的文件
find /usr -user hadoop -o -group root -a -type d

9.查找权限为777的文件
find / -perm -777 -type d -ls

10.显示命令历史
history

11.grep
grep hadoop /etc/password

```

10.打包与压缩

```
1.gzip压缩
gzip a.txt

2.解压
gunzip a.txt.gz
gzip -d a.txt.gz

3.bzip2压缩
bzip2 a

4.解压
bunzip2 a.bz2
bzip2 -d a.bz2

5.将当前目录的文件打包
tar -cvf bak.tar .
将/etc/password追加文件到bak.tar中(r)
tar -rvf bak.tar /etc/password

6.解压
tar -xvf bak.tar

7.打包并压缩gzip
tar -zcvf a.tar.gz

8.解压缩
tar -zxvf a.tar.gz
解压到/usr/下
tar -zxvf a.tar.gz -C /usr

9.查看压缩包内容
tar -ztvf a.tar.gz

zip/unzip

10.打包并压缩成bz2
tar -jcvf a.tar.bz2

11.解压bz2
tar -jxvf a.tar.bz2


```

11.正则

```
1.cut截取以:分割保留第七段
grep hadoop /etc/passwd | cut -d: -f7

2.排序
du | sort -n 

3.查询不包含hadoop的
grep -v hadoop /etc/passwd

4.正则表达包含hadoop
grep 'hadoop' /etc/passwd

5.正则表达(点代表任意一个字符)
grep 'h.*p' /etc/passwd

6.正则表达以hadoop开头
grep '^hadoop' /etc/passwd

7.正则表达以hadoop结尾
grep 'hadoop$' /etc/passwd

规则：
.  : 任意一个字符
a* : 任意多个a(零个或多个a)
a? : 零个或一个a
a+ : 一个或多个a
.* : 任意多个任意字符
\. : 转义.
\<h.*p\> ：以h开头，p结尾的一个单词
o\{2\} : o重复两次

grep '^i.\{18\}n$' /usr/share/dict/words

查找不是以#开头的行
grep -v '^#' a.txt | grep -v '^$' 

以h或r开头的
grep '^[hr]' /etc/passwd

不是以h和r开头的
grep '^[^hr]' /etc/passwd

不是以h到r开头的
grep '^[^h-r]' /etc/passwd

```

12.输入输出重定向及管道

```
1.新建一个文件
touch a.txt
> b.txt

2.错误重定向:2>
find /etc -name zhaoxing.txt 2> error.txt

3.将正确或错误的信息都输入到log.txt中
find /etc -name passwd > /tmp/log.txt 2>&1 
find /etc -name passwd &> /tmp/log.txt

4.追加>>

5.将小写转为大写（输入重定向）
tr "a-z" "A-Z" < /etc/passwd

6.自动创建文件
cat > log.txt << EXIT
> ccc
> ddd
> EXI

7.查看/etc下的文件有多少个？
ls -l /etc/ | grep '^d' | wc -l

8.查看/etc下的文件有多少个，并将文件详情输入到result.txt中
ls -l /etc/ | grep '^d' | tee result.txt | wc -l

```

13.进程控制

```
1.查看用户最近登录情况
last
lastlog

2.查看硬盘使用情况
df

3.查看文件大小
du

4.查看内存使用情况
free

5.查看文件系统
/proc

6.查看日志
ls /var/log/

7.查看系统报错日志
tail /var/log/messages

8.查看进程
top

9.结束进程
kill 1234
kill -9 4333

```