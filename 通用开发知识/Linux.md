> 25 年 9-6 ，唉，还是逃不掉啊，因为 mac 系统的跟 linux 都差不多，还是得熟悉熟悉
   标记为【粗学】的，是没有认真学的，甚至是视频都没有看过的，如果之后有遇到在回来看

# linux 目录结构

Linux 的世界中，所有的东西都被看成 **文件（所有的硬件资源，在 linux 中会变成文件来进行管理）**

linux 系统中，有一个根目录（/），来统领所有的子目录，每个目录都有其独特的作用，都安排了不同的功能。

`/bin`：存放**命令**相关文件

`/dev`：**设备管理器**，把所有的硬件变成软件，存储到该目录

`/home`：存放**用户**的信息，每一个用户都单独开一个文件

`/media`：一些 U盘、驱动信息，linux 会自动识别，将其挂载在这个目录下

`/opt`：存放**额外的安装软件**

`/root`：**系统管理员**的用户主目录

`/usr`：用户的**应用程序和文件**

`/boot`：linux 的**核心文件**

`/etc`：所有的**系统需要的配置文件**

`/lib`：**系统开机**所需的动态连接共享库

`/lost+found`：一般是空的，当系统遇到**非法关机**时，会产生一些文件

`/mnt`：让用户临时挂载别的文件系统

`/proc`：【不能动】虚拟目录，系统内存的映射

`/sys`：【不能动】系统文件

`/srv`：【不能动】一些服务启动后，需要提取的数据

`/tmp`：存放**临时文件**

`/var`：将**经常被修改目录**放在这个目录，日志文件

`/usr/local`：这是另一个**安装软件**的目录，一般通过**编译源码**的方式进行安装

# vi / vim 模式

文本编辑模式，允许开发者直接在 linux 系统上编辑文本

需要掌握一些快捷键，和三种模式

## 模式

1. 一般模式【不可编辑，只能看】
2. 命令行模式【输入: 或 /】
3. 编辑模式【输入 i、I、insert】--> wq：保存并退出 q!：强制退出（不保存）

各个模式之间，都是通过 **ESC** 来进行切换的

## 一些快捷键

1. 拷贝 ---> 在一般模式下，输入 yy （复制），在输入 p（粘贴）
2. 删除 ---> 在一般模式下，输入 dd
3. 查找 ---> 在命令行模式下，输入 /关键字，回车查找，输入 n 找下一个
4. 设置行号 ---> 在命令行模式下，输入 :set nu 和 :set nonu
5. 撤销 ---> 在一般模式下，输入 u
6. 在一般模式下，达到文件最末行：G，达到首行：gg，移动到任意行：行号 + shift + g

# 关机和重启

也是一些命令

关机：shutdown -h now 立即关机

shutdown -h 1 1 分钟后关机

halt

重启：shutdown -r now 立即重启计算机

reboot

同步数据：sync （将内存中的数据同步到硬盘中）

注意：建议在关机或重启的时，最好执行一遍 sync 指令

# 用户管理

在 linux 中，root 系统管理员，权限最大，可以创建其他用户。所以，登录时尽量少用 root 账号登录，避免操作失误。可以用普通用户登录，在使用“su - root”来切换成系统管理员身份

## **切换身份**

su - 用户名

## 注销用户

logout

## 添加用户

useradd 用户名

## 指定密码

passwd 用户名 （为那个用户，指定密码）

## 删除用户

userdel 用户名（删除用户，但保留 home 下面的工作目录）/ userdel -r 用户名（彻底删除）

## 查看用户信息

id 用户名

## 查看登录用户信息

who am i

## 分组

系统可以对有共性的多个用户进行统一的管理

- 新增组：groupadd 组名
- 删除组：groupdel 组名
- 增加用户时指定组名：useradd -g 用户组 用户名
- 修改用户组名：usermod -g 用户组 用户名（如果没有指定组名，默认是用户会创建一个同名的组）

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757218414847-28f0b6c5-8b38-4de7-8d62-8176992b4f96.png)

# 运行级别

默认是 5 这个级别

那么在 linux 中，有很多的运行级别，运用的比较多的是，3（多用户、有网络终端） 和 5（图形化） 这两个级别

可以通过 `init [0123456]`这个指令来指定使用哪个级别，

例如：使用 3

`init 3`

# 帮助指令

如果不知道，某个指令是什么意思，可以使用这里面的指令

1. man 指令名：man ls
2. help 指令名：help ls

补充：在 linux 系统下，以 `.` 开头的是隐藏文件

# 文件目录指令

## pwd 指令

显示当前工作目录的绝对路径（跟 node.js 中的 __dirname 差不多）

## ls 指令

`ls [ 选项 ][ 目录或文件 ]`

- -a ：显示目录下的所有的文件和目录信息，包括隐藏的
- -l ：以列表的方式显示信息

## **cd 指令**

`cd [ 参数 ]` 切换到指定目录

- cd ~ 切换到用户的家目录 ，比如：root 的家目录 是 /root
- cd .. 回到上一级目录

## mkdir 指令

`mkdir [ 选项 ]` 创建目录

- -p 创建多级目录

## rmdir 指令

`rmdir [ 选项 ]` 删除的空目录

注意：rmdir 删除的是**空目录**，如果目录下有内容时无法删除的

但，可以强制删除 `rm -rf [ 目录 ]`使用 rm 指令

## touch 指令

`touch 文件名`创建空白文件

## cp 指令

`cp [ 选项 ] source dest`

- -r 递归复制整个文件夹
- \cp 强制覆盖不提示

## rm 指令

`rm [ 选项 ] 要删除的文件或目录`

- -r：递归删除整个文件夹
- -f：强制删除不提示

## mv 指令

移动文件或重命名

- `mv oldNameFile newNameFile` 在同一级别 【重命名】
- `mv /tmp/aaa /home` 绝对定位 【移动文件】

## cat 指令

`cat [ 选项 ] 文件路径`查看文件内容

- -n：显示行号

## >指令和>>指令

- > **覆盖**文件
- >> **追加**内容

例如：

```
cat 文件1 > 文件2 // 将文件1的内容覆盖到文件2
echo "内容" >> 文件路径 // 将内容追加到文件中
```

## **ln 指令**

`ln -s [ 连接的文件/目录 ][ 软连接名 ]`类似于**快捷方式**

例如：

```
# 在 /home 目录下创建一个软连接 myroot，连接到 /root 目录
cd /home
ln -s /root myroot
# 删除软连接 myroot
rm -f myroot # 不要加最后的 myroot/ 会被认为是目录，从而删除不了 
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757753032684-085eac11-6407-4739-8496-db493868541b.png)

## **history 指令**

查看已经执行过的历史命令

例如：

```
# 显示所有的命令
history
# 显示最近使用过的10个命令
history 10
# 执行历史编号为 5 的指令
!5
```

# 时间指令

**date 指令：**`date`查看当前时间

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757753824516-f31032fc-be21-4e4e-8410-cbb102fe19ab.png)

**cal 指令：**`cal`查看当前月份 `cal 2025`查看 2025 年所有日历

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757753837636-b53e179c-87ba-48ea-a666-b971e291b8af.png)

# 压缩和解压

## **gzip/gunzip 指令**

gzip 压缩**文件（不能压缩****目录****）**，gunzip 解压

例如：

```
# gzip压缩，将home下的hello.txt文件进行压缩
gzip /home/hello.txt

# gunzip压缩，将/home下的hello.txt.gz文件进行解压缩
gunzip /home/hello.txt.gz
```

## **zip/unzip 指令**

`zip [ 选项 ] 压缩后的名字 压缩文件/目录路径` zip 用于压缩**文件/目录**，unzip 用于解压

- -r：【zip】递归压缩，压缩目录
- -d：【unzip】指定解压后文件的存放目录

例如：

```
# 将home下的所有文件进行压缩成myhome.zip
zip -r myhome.zip /home

# 将myhome.zip解压到/opt/tmp目录下
mkdir /opt/tmp
unzip -d /opt/tmp/ myhome.zip
```

## **tar 指令**

`tar [ 选项 ] 打包后的名字 压缩/被压缩的文件/目录路径`

- -c：产生.tar 打包文件
- -v：显示详细信息
- -f：指定压缩后的文件名
- -z：打包同时压缩
- -x：解压.tar 文件

一般压缩：`-zcvf` 解压：`-zxvf`

例如：

```
# 压缩多个文件，将/home/pig.txt和/home/cat.txt压缩成pc.tar.gz
tar -zcvf pc.tar.gz /home/pig.txt /home/cat.txt

# 将/home的文件夹压缩成myhome.tar.gz
tar -zcvf myhome.tar.gz /home

# 将pc.tar.gz解压到当前目录
tar -zxvf pc.tar.gz

# 将myhome.tar.gz解压到/opt/tmp2目录下
mkdir /opt/tmp2
tar -zxvf myhome.tar.gz -C /opt/tmp2 # -C 指定解压目录
```

# 组管理和权限管理【粗学】

在 Linux 系统内，有 组、所有者、其他组 的概念

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757818895603-b95fb647-baae-462a-b2e6-218980365616.png)

## 查看**所有者**

`ls -ahl`

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757819000682-3c70f2e1-f883-4d8c-97d4-b7688b27aaf9.png)

## **修改**所有者

`chown 用户名 文件名`

```
# 使用 root 创建一个文件apple.txt，然后修改成 fxl 为所有者
su - root
touch apple.txt
chown fxl apple.txt
```

## 查看**所在组**

`ls -ahl`

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757819840541-1fdd7db6-d165-4604-99ba-284704edb52a.png)

## **修改组**

`chgrp 组名 文件名`

```
# 使用 root 用户创建文件 orange.txt，修改到 test 组
groupadd test
su - root
touch orange.txt # --> 这个文件会默认是 root 组的
chgrp test orange.txt # --> 将orange.txt修改到 test 组
```

## **rwx 权限**

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757835234854-8406192f-e435-47ca-b09e-035cbf187336.png)

通过`ll 或 ls -l`展示的文件信息，前面 9 位是权限说明

具体内容：

1. 第 0 位确定文件类型（d, -, l, c, b）

2. **l** 是链接，由`ln -s`指令创建
3. **d** 是目录，由`mkdir`指令创建
4. **c** 是字符设备文件，比如鼠标、键盘
5. **b** 是块设备，比如硬盘
6. **-** 是普通文件

7. 第 1-3 位说明**所有者**的权限
8. 第 4-6 位说明**所在组**的权限
9. 第 7-9 位说明**其他组**拥有的权限

例如：

**-****rw-****r--****r--**

-：说明是普通文件

rw-：说明文件所有者的权限是：能读能写，但不可执行

r--：说明文件所在组的其他用户权限是：只能读

r--：说明其他组的用户权限是，只能读

具体来说：

- rwx 作用到**文件**

- 【r】：可以读取，查看
- 【w】：可以修改，但不一定能删除文件，删除文件的前提条件是，对该文件所在的**目录由****写****权限**，才能删除该文件
- 【x】：代表是否可执行该文件

- rwx 作用到**目录**

- 【r】：可以读取
- 【w】：可以修改，对目录内创建 + 删除 + 重命名目录
- 【x】：可以进入该目录

## **修改权限**

`**+ -**`

前提：u：所有者，g：所在组，o：其他人，a：所有人

1. `chmod u=rwx, g=rx, o=x 文件/目录`
2. `chmod o+w 文件/目录`
3. `chmod a-x 文件/目录`

例如：

```
# 给 abc 文件的所有者 读写执行的权限，给所在组 读执行权限，给其他组 执行权限
chmod u=rwx, g=rx, o=x
```

# 调度任务

## crontab

`crontab [ 选项 ]`

- -e：**编辑** crontab 定时任务
- -l：**查询** crontab 任务
- -r：**终止**任务调度

例如：

```
# 每隔1分钟，将当前日期和日历都追加到/home/mycal文件中
# 写一个shell脚本，在让 crond 执行
vim cal.sh --> date >> /home/mycal cal >> /home/mycal
chmod u+x # 给用户执行的权限，不然crond不能执行
crondtab -e --> */1 * * * * /home/cal.sh
```

`*/1 * * * * /home/cal.sh`

`*/1`：多少分钟

`*`：24 小时

`*`：日/号

`*`：月

`*`：周

`*/1 * * * *`每天每时刻 过一分钟 执行以下 /home 目录下的 cal.sh 脚本

## at【粗学】

一次性执行定时计划任务

`at [ 选项 ][ 时间 ]`

`Ctrl + D`结果 at 命令输入

查看所有任务：`atq`

删除任务：`atrm 命令编号`

# 磁盘分区【粗学】

## 查看磁盘分区情况

`lsblk`/ `lsblk -f`

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757856462459-b4b644b6-6b9c-49d7-b69c-994b2114fbb2.png)

# 网络配置

在 linux 上查询，网络相关配置 `ifconfig`

在 Windows 上查询，网络相关配置`ipconfig`

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757860007286-5343c263-4de1-4f07-a211-499a1068ebf1.png)

在 Linux 中，获取 IP 是通过 DHCP 动态获取 IP 的。可以改成 static 静态的，后续方便用户查找服务器（后端应用会搭在 linux 服务上）

那我这里，不是专业的，就稍微了解即可。

如果要修改 linux 的 IP，那么要记得将 Linux 的网关地址改了，改成同一网段的。同时，宿主机的 IP（VMnet8） 也需要改成同一网段的。

这里，记住一个命令即可，

**重启网络服务**指令：`service network restart`

## 设置主机名

这就需要一点网络知识了，当然主包还是很强的，计算机网络的知识还是到位了。

其实这里就是说的，域名和 IP 的关系，在后面一点有原理图

为了方便记忆，可以给 linux 系统设置主机名

**查看主机名**指令：`hostname`

**修改主机名**：修改文件`etc/hostname`重启生效

那么，如何通过主机名能够 ping 到某个 linux 系统呢？

修改`hosts`文件即可，将 IP 与主机名对应起来

**在 windows 下，**

- 修改`c:\Windows\System32\drivers\etc\hosts`
- 增加一行 IP <-> 主机名：192.168.200.130 fxl

**在 Linux 下，**

- 修改`/etc/hosts`
- 增加一行 IP <-> 主机名：192.168.200.1 fxl

## IP 与域名的原理图

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1757946165514-1b933c87-2164-49e6-8c36-6501a62e8c09.png)

# 进程管理

那么这一块也是比较重要的

## 查看进程信息

`**ps [ 选项 ]**`

- -a：显示当前终端的所有进程信息
- -u：以用户的格式显示进程信息
- -x：显示后台进程运行的参数

`**pstree [ 选项 ]**`

- -p：显示进程的 PID
- -u：显示进程的所属用户

**指令**：`ps-aux | grep xxx`【使用一个管道指令 grep 过滤 xxx 关键字】

比如我看看有没有sshd服务：`ps -aux | grep sshd`

**指令说明**

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1758336451944-d42d106b-0253-4c27-bfb3-98dcc13155fe.png)

`USER`：用户名称

`PID`：进程号

`%CPU`：进程占用cpu的百分比

`%MEM`：进程占用物理内存的百分比

`VSZ`：进程占用的**虚拟**内存大小（单位：kb)

`RSS`：进程占用的**物理**内存大小（单位：kb）

`TTY`：终端名称缩写

`STAT`：进程状态，其中s-睡眠，s.-表示该进程是会话的先导进程，n-表示进程拥有比普通优先级更低的优先级，r-正在运行，d-短期等待，z-死进程，t-被跟踪或者被停止等等

`START`：进程的启动时间

`TIME`：进程使用cpu的总时间

`COMMAND`：启动进程所用的命令和参数，如果过长会被截断显示

## 查看父进程

其实就是查看某个进程父进程的 pid，就能知道父进程是谁了

`ps -ef`以全格式显示当前所有的进程

- -e：显示所有进程
- -f：全格式

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1758340248176-3df515ba-e88a-41e6-8946-eb08dd69e3b8.png)  
`uid`：用户id  
`pid`：进程id  
`ppid`：父进程id  
`c`：cpu用于计算执行优先级的因子。**数值越大**，表明进程是cpu密集型运算，执行**优先级会降低**；数值越小表明进程是/o密集型运算，执行优先级会提高  
`stime`：进程启动的时间  
`tty`：完整的终端名称  
`tme`：cpu时间  
`cmd`：启动进程所用的命令和参数

## 终止进程

有两个指令：

`kill [ 选项 ] 进程号`：终止某个进程，如果终止不了，可以用选项

- -9：强制终止进程

`killall 进程名称`：通过进程名称终止进程，这个要注意的是，下面的子进程也会被杀死

# 服务管理

说的更直白一点，可以理解为应用程序，每一个服务（守护进程）都会监听一个端口，与外部的客户端通信。

在 linux 中，服务管理的命令，有两种：

- `service 服务名 [start | stop | restart | reload | status]`（centos7 以下）
- `systemctl`

service 能管理的服务，可以在`/etc/init.d`这个目录下查看，

centos8 这个版本，已经不多了

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1758360891826-20d9f6e6-929b-4ba3-8bd4-998cc4e7b6ab.png)

查看全部的服务，

```
# 查看所有系统服务（包括运行和停止的）
systemctl list-unit-files --type=service

# 查看所有正在运行的服务
systemctl list-units --type=service --state=running
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1758362312371-f9792010-278d-4120-88d5-c967af2a4956.png)

## systemctl 指令

centos8 之后，大部分的服务都是用这个命令来管理的

基本语法：`systemctl [start | stop | restart | status] 服务名`

systemctl 指令管理的服务可以在`/usr/lib/systemd/system`查看

systemctl 设置服务的自启动状态：

- `systemctl list-unit-files` 查看全部服务开机启动状态【是自启动还是手动启动】

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1758420730865-f62f5972-3975-43db-a3fd-fa9e6f2314b9.png)

- `systemctl enable 服务名`设置某个服务自启动
- `systemctl disable 服务名`设置某个服务手动启动
- `systemctl is-enabled 服务名`查询某个服务的启动状态

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1758420763847-6197c0dc-b94f-4f2c-baff-16734f4dfe64.png)

例如：

```
# 将防火墙关闭
systemctl stop firewalld
# 用powershell命令测试，是否连通
Test-NetConnection -ComputerName 192.168.238.132 -Port 111
```

可以看到，只要关闭了防火墙，那么外部就可以随意连接【这样比较危险，生产环境不用】

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1758421812542-9f4b00ce-91e6-4783-9e74-3d4f5e217bc2.png)

## firewall 指令

**打开端口**：`firewall-cmd --permanent --add-port=端口号/协议`

**关闭端口**：`firewall-cmd --permanent --remove-port=端口号/协议`

执行完上面两个，一定要记得重载 `firewall-cmd --reload`

查询端口是否开放：`firewall-cmd --query-port=端口/协议`

补充一个：查看端口、协议对应关系`netstat -anp | more`

## 动态监控系统

一个跟 ps 指令差不多的指令，区别就是，ps 指令是静态的，top 指令是动态的

top 指令，可以监控某一段时间内，服务的变化

`top [ 选项 ]`

- -d（秒数）：指定 top 命令每隔几秒更新，默认是 3 秒
- -i：不显示任何闲置或僵死进程（僵死进程：进程已经不在工作，但依然占用内存）
- -p：通过 PID 指令监控某个进程的状态