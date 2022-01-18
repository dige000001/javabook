## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂
# 目录结构

在linux中，/ 为根目录

**/bin:**  bin是Binary的缩写,该目录下存放的是最常用的命令。

/boot:该目录下存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。

/dev: dev是Device(设备)的缩写。该目录下存放的是Linux的外部设备。在Linux中，访问设备的方式和访问文件的方式是相同的。

**/etc:** 该目录下存放的是所有系统管理所需要的配置文件和子目录。

**/home**:这是用户的家目录。在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。

**/lib和/lib64**:这两个目录下存放的是系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件，几乎所有的应用程序都需要用到这些共享库。其中/lib64为64位的软件包的库文件所在目录。

/media:系统会自动识别一些设备（如U盘、光驱等），当识别后，Linux会把识别的设备挂载到该目录下。

/mnt:系统提供该目录是为了让用户临时挂载别的文件系统。我们可以将光驱挂载到/mnt)上，然后进入该目录查看光驱里的内容。

/opt:这是给主机额外安装软件所设置的目录，该目录默认为空。比如，你要安装一个Oracle数据库，可以放到该目录下。

**/proc**:该目录是一个虚拟目录，是系统内存的映射，可以直接访问它来获取系统信息。该目录的内容在内存里，我们可以直接修改里面的某些文件。比如可以通过下面的命令来屏蔽主机的ping命令，使其他人无法ping你的机器。在日常工作中，你会经常用到类似的用法:

#####  # echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all

**/root:** 该目录是系统管理员的用户家目录。

/run:这个目录其实和/var/run是同一个目录，这里面存放的是一些服务的pid。一个服务启动完后，是有一个pid文件的。

/sbin: s就是Super User的意思，该目录存放的是系统管理员使用的系统管理程序。

/srv:该目录存放的是一些服务启动之后需要提取的数据。

/sys:该目录存放的是与硬件驱动程序相关的信息。

/tmp:该目录用来存放一些临时文件。

/usr:这是一个非常重要的目录，类似于Windows下的Program Files目录，用户的很多应用程序和文件都存放在该目录下

/usr/bin:该目录存放的是系统用户使用的应用程序。

/usr/sbin:该目录存放的是超级用户使用的比较高级的管理程序和系统守护程序。

/usrlsrc :该目录是内核源代码默认的放置目录。

**/var:** 该目录存放的是不断扩充且经常修改的目录，包括各种日志文件或者pid文件，其中刚刚提到的/var/run就是在这个目录下面。

# 文件属性

\


一个Linux目录或者文件，都会有一个所有者和所属组。所有者是指文件的拥有者，而所属组指的是这个文件属于哪一个用户组

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8bdbe4e30e414caea82c4f834e1cf22c~tplv-k3u1fbpfcp-zoom-1.image)
在上例中，用ls -l命令查看当前目录下的文件时，共显示了9列内容（用空格划分列)，它们都代表什么含义呢?

#### 第1列：包含该文件的类型、所有者、所属组以及其他用户对该文件的权限。

第1列共11位，其中第1位用来描述该文件的类型。上例中我们看到的文件类型-，其实还有d、l、b、c、s等，具体描述如下所示：

```
d表示该文件为目录。
-表示该文件为普通文件。
l表示该文件为链接文件(link file )
b表示该文件为块设备，比如/dev/sda就是这样的文件，磁盘分区文件就是这种类型。
c表示该文件为串行端口设备文件(又称字符设备文件)，比如键盘、鼠标、打印机、tty终端等都是这样的文件。
s表示该文件为套接字文件( socket )，用于进程之间的通信,后面讲到MySQL时会用到该类型的文件。
```

文件类型后面的9位，每3位为一组，上例中均为rwx这3个参数的组合。其中，r代表可读，w代表可写，x代表可执行。**前3位为所有者(user)的权限，中间3位为所属组(group)的权限，最后3位为其他非本群组用户(others)的权限。** 下面举例来说明一下:

假设一个文件的属性为-rwxr-xr-，它代表的意思是，该文件为普通文件，文件拥有者可读、可写且可执行，文件所属组对其可读、不可写但可执行，其他用户对其只可读。**对于一个目录来讲，打开这个目录即为执行这个目录，所以任何一个目录必须要有x权限才能打开并查看该目录下的内容**。例如，一个目录的属性为drwxr--r--，其所有者为root，那么除root之外的其他用户是不能打开这个目录的。

注：linux用颜色表示文件类型

```
白色：表示普通文件
蓝色：表示目录
绿色：表示可执行文件
红色：表示压缩文件
浅蓝色：链接文件
红色闪烁：表示链接的文件有问题
黄色：表示设备文件
灰色：表示其他文件
```

第1列最后1位为“.”,要特别说明一下。老版本CentOS 5是没有这个点的，这主要是因为新版本的ls添加了SELinux或者acl的属性。如果文件或者目录使用了SELinux context的属性,这里会是一个点“.";如果设置了acl的属性，这里会是一个加号“+”。关于SELinux和acl，不再详细介绍

#### 第2列:表示该文件占用的节点( inode )，如果是目录，那这个数值与该目录下是子目录数量有关。

#### 第3列:表示该文件的所有者。

#### 第4列:表示该文件的所属组。

#### 第5列:表示该文件的大小。

#### 第6列、第7列和第8列:表示该文件最后一次被修改的时间(mtime)，依次为月份、日期以及时间。

#### 第9列:表示文件名。
# 更改文件的权限

\


## chown

**chown ( change owner的简写)命令可以更改文件的所有者**，其格式为: 

chown [-R] 账户名 文件名 或者

chown [-R] 账户名:组名 文件名

这里的-R选项只适用于目录，作用是级联更改，即不仅更改当前目录，连目录里的目录或者文件也全部更改。示例命令如下:

```
[root@localhost ~]# mkdir dir3
[root@localhost ~]# useradd user1
[root@localhost ~]# touch dir3/test3
[root@localhost ~]# ls -ld dir3
drwxr-xr-x 2 root root 19 8月  29 10:20 dir3 //此时文件夹所有者和所属组都是root
[root@localhost ~]# chown user1 dir3
[root@localhost ~]# ls -ld dir3
drwxr-xr-x 2 user1 root 19 8月  29 10:20 dir3 //此时文件夹所有者是dir3
[root@localhost ~]# ls -l dir3               
-rw-r--r-- 1 root root 0 8月  29 10:20 test3  //但是文件夹里面的文件所有者和所属组还是root
[root@localhost ~]# groupadd testgroup
[root@localhost ~]# chown -R user1:testgroup dir3
[root@localhost ~]# ls -ld dir3
drwxr-xr-x 2 user1 testgroup 19 8月  29 10:20 dir3 //文件夹所有者和所属组改变了
[root@localhost ~]# ls -l dir3
总用量 0
-rw-r--r-- 1 user1 testgroup 0 8月  29 10:20 test3 //文件夹里所有文件所有者和所属组改变了
```

## chmod

**命令chmod( change mode的简写)用于改变用户对文件/目录的读写执行权限**，其格式为: 

chmod [-R] xyz 文件名 (这里的xyz表示数字)。

其中，-R选项的作用等同于chown命令的-R选项，也表示级联更改。值得注意的是，在Linux系统中，为了方便更改文件的权限，Linux使用数字代替rwx，具体规则为:r等于4，w等于2，x等于1，-等于O。例如，rwxrwx---用数字表示就是770,其具体算法为: rwx=4+2+1=7，rwx=4+2+1=7，---=0+0+O=0。

一个目录的默认权限为755，而一个文件的默认权限为644。下面我们举例说明一下:

```
[root@localhost ~]# ls -ld dir3
drwxr-xr-x 2 root root 19 8月  29 10:20 dir3 	//目录的默认权限为755
[root@localhost ~]# ls -l dir3
总用量 0
-rw-r--r-- 1 root root 0 8月  29 10:20 test3   //文件的默认权限为644
[root@localhost ~]# chmod 750 dir3
[root@localhost ~]# ls -ld dir3
drwxr-x--- 2 root root 19 8月  29 10:20 dir3   //修改目录的权限为750
[root@localhost ~]# ls -l dir3
总用量 0
-rw-r--r-- 1 root root 0 8月  29 10:20 test3   //不影响文件权限
[root@localhost ~]# chmod -R 700 dir3  				 //修改目录和目录下的文件的权限为700
[root@localhost ~]# ls -ld dir3
drwx------ 2 root root 19 8月  29 10:20 dir3 
[root@localhost ~]# ls -l dir3
总用量 0
-rwx------ 1 root root 0 8月  29 10:20 test3
```

chmod还支持使用rwx的方式来设置权限。从之前的介绍中可以发现，基本上就9个属性。我们可以使用u、g和o来分别表示user、group和others的属性，用a代表all(即全部)。下面举例来介绍它们的用法，示例命令如下:

```
[root@localhost ~]# ls -l dir3
总用量 0
-rwxr----- 1 root root 0 8月  29 10:20 test3
[root@localhost ~]# chmod u=rwx,o=r dir3/test3
[root@localhost ~]# ls -l dir3
总用量 0
-rwxr--r-- 1 root root 0 8月  29 10:20 test3
```

可以根据u、a、o、g直接增加后减少某个权限：


```
[root@localhost ~]# ls -l dir3
总用量 0
-rwxr--r-- 1 root root 0 8月  29 10:20 test3
[root@localhost ~]# chmod g-r,o-r dir3/test3  //group和others取消读权限
[root@localhost ~]# ls -l dir3
总用量 0
-rwx------ 1 root root 0 8月  29 10:20 test3
```

## umask

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/616ab8edea944e939e9276f955636de5~tplv-k3u1fbpfcp-zoom-1.image)

**注意：不要用数字直接减，应该换成rwx去减**

关于umask的计算方法，有的朋友喜欢换算成数字去做减法，比如 rwxrwxrwx - ----w--w-=777-022=755。乍一看这好像没有任何问题，但有时会出错，比如当umask值为033时，如果使用单纯的减法，文件的默认权限则为666-033=633，但实际权限应该为rw-rw-rw- - -----wx-wx=rw-r--r--=644。

\


## 修改文件的特殊属性

### chattr

命令chattr ( change attribute）的格式为: 

chattr [+-=][Asaci][文件或者目录名]，其中，+、和=分别表示增加、减少和设定。各个选项的含义如下。

```
A:增加该属性后，表示文件或目录的atime将不可修改。
s:增加该属性后，会将数据同步写入磁盘中。
a:增加该属性后，表示只能追加不能删除,非root用户不能设定该属性。
c:增加该属性后，表示自动压缩该文件,读取时会自动解压。
i:增加该属性后，表示文件不能删除、重命名、设定链接、写入以及新增数据。
```

常用a和i两个属性，举例一下：

#### i 

```
目录：
[root@localhost ~]# chattr +i dir3
[root@localhost ~]# touch dir3/test
touch: 无法创建"dir3/test": 权限不够
[root@localhost ~]# chattr -i dir3
[root@localhost ~]# touch dir3/test
[root@localhost ~]# chattr +i dir3
[root@localhost ~]# rm -f dir3/test
rm: 无法删除"dir3/test": 权限不够

文件：
[root@localhost ~]# echo 1111 >> dir3/test
[root@localhost ~]# cat dir3/test
1111
[root@localhost ~]# chattr +i dir3/test
[root@localhost ~]# echo 1111 >> dir3/test
-bash: dir3/test: 权限不够
```

#### a

```
目录：
[root@localhost ~]# chattr -i dir3
[root@localhost ~]# chattr +a dir3
[root@localhost ~]# touch dir3/test2
[root@localhost ~]# rm -f dir3/test2
rm: 无法删除"dir3/test2": 不允许的操作


文件：
[root@localhost ~]# chattr +a dir3/test2
[root@localhost ~]# echo 111111 >> dir3/test2
[root@localhost ~]# echo 111111 > dir3/test2
-bash: dir3/test2: 不允许的操作
[root@localhost ~]# rm -f dir3/test2
rm: 无法删除"dir3/test2": 不允许的操作
```

### lsattr

lsattr ( list attribute)，该命令用于读取文件或者目录的特殊权限，其格式为:

lsattr [-aR][文件/目录名]。

下面先来看看-a和-R这两个选项的含义。

-a:类似于ls的-a选项,即连同隐藏文件一同列出。

-R:连同子目录的数据一同列出。默认有这个选项

```
[root@localhost ~]# lsattr dir3
---------------- dir3/test3
----i----------- dir3/test
-----a---------- dir3/test2

[root@localhost ~]# lsattr -R dir3
---------------- dir3/test3
----i----------- dir3/test
-----a---------- dir3/test2

[root@localhost ~]# lsattr -a dir3
-----a---------- dir3/.
---------------- dir3/..
---------------- dir3/test3
----i----------- dir3/test
-----a---------- dir3/test2
```
### set uidset gid和sticky bit。

前面介绍权限的时候，我们一直都是用3位数(如700，755)，其实最前面还有一位,那就是下面要讲的set uidset gid和sticky bit：

-   **set uid:** 该权限针对二进制可执行文件，使文件在执行阶段具有文件所有者的权限。比如,passwd这个命令就具有该权限。当普通用户执行passwd命令时，可以临时获得root权限，从而可以更改密码。
-   **set gid:** 该权限可以作用在文件上（二进制可执行文件)，也可以作用在目录上。当作用在文件上时，其功能和set uid一样，它会使文件在执行阶段具有文件所属组的权限。目录被设置这个权限后，任何用户在此目录下创建的文件都具有和该目录所属的组相同的组。

<!---->

-   **sticky bit**:可以理解为防删除位。文件是否可以被某用户删除，主要取决于该文件所在的目录是否对该用户具有写权限。如果没有写权限，则这个目录下的所有文件都不能删除，同时也不能添加新的文件。如果希望用户能够添加文件但不能删除该目录下其他用户的文件，则可以对父目录增加该权限。设置该权限后，就算用户对目录具有写权限，也不能删除其他用户的文件。

```
[root@localhost ~]# ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 27856 4月   1 2020 /usr/bin/passwd

[root@localhost ~]# ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 27856 4月   1 2020 /usr/bin/passwd
```

可以发现，passwd显示的是rws而非传统的rwx，用数字表示为4755。/tmp/显示的rwt而非rwx，用数字表示为1777。那么，这个4和1是如何计算出来的呢?当有特殊权限时，第一位数字可以是0、1( --t)、2( -s-)、3 ( -st)、4( s--)、5 ( s-t)、6 (ss-)或7 ( sst)。再回过头来看passwd，它是s--，所以是4;而/tmp/是--t，所以是1。

注：s-- 意思是所有者，所属组，其他这三个权限中，只有所有者有s，其他都没有

配置这些特殊权限的方法和之前一样。比如，我想给一个文件增加set uid权限,那么命令为chmodu+s filename，而去掉这个权限的命令则为chmod u-s filename。同理，想设置set gid权限的命令为chmod g+s dirname，设置sticky bit权限的命令为chmod o+t dirname。

有时候，你可能会发现set_uid上的权限为大写的S，而不是小写的s，比如rwS，这是因为该文件没有x权限所致,不管是大写的s还是小写的s,都表示它存在set_uid或者set_gid权限,同理sticky bit也一样。

# 搜索文件

\


### which

**which只能用来查找PATH环境变量中出现的路径下的可执行文件**。这个命令比较常用，有时我们不知道某个命令的绝对路径，用which查找就很容易知道了。例如，查找vi和cat的绝对路径,示例命令如下:

```
[root@localhost ~]# which vi
/usr/bin/vi
[root@localhost ~]# which cat
/usr/bin/cat
```

### whereis

whereis命令通过预先生成的一个文件列表库查找与给出的文件名相关的文件，其格式为

whereis[-bms][文件名称]，其中各选项的含义如下所示。

```
-b:只查找二进制文件。
-m:只查找帮助文件（在man目录下的文件)。
-S:只查找源代码文件。

[root@localhost ~]# whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
```

### find(重要)

其格式为:  **find[路径][参数]。** 下面介绍常用的几个参数。

```
-atime +n/-n:表示访问或执行时间大于或小于n天的文件。
-ctime +n/-n:表示写入、更改inode属性（如更改所有者、权限或者链接）的时间大于或小于n天的文件。
-mtime +n/-n:表示写入时间大于或小于n天的文件,该参数用得最多。
```

```
[root@localhost ~]# find dir3 -atime +1
dir3/test3
dir3/test
dir3/test2
```

上例中，-atime +1表示atime在1天以上的文件。还有一种用法-mmin -10，表示mtime在10分钟内的文件。有时候，也可以不加+或者-，比如-mtime 10，这表示正好为10天,这种用法就少了。

```
[root@localhost dir3]# find -mtime +1
./test3
./test
./test2
可以看到，修改天数大于一天的文件包含test

[root@localhost dir3]# vi test
[root@localhost dir3]# find -mtime +1
./test3
./test2

修改过后，就没了
```

文件的access time (即atime)是在读取文件或者执行文件时更改的。

文件的modified time(即mtime )是在写入文件时随文件内容的更改而更改的。

文件的change time(即ctime )是在写入文件、更改所有者、权限或链接设置时随inode内容的更改而更改的。

其中，inode(索引节点）用来存放档案及目录的基本信息，包含时间信息、文档名、所有者以及所属组等。inode是Unix操作系统中的一种数据结构，其本质是结构体，在文件系统创建时生成,且个数有限。在Linux下，可以通过命令df -i来查看各个分区的inode总数以及使用情况。

因此，更改文件的内容即会更改mtime和ctime，但是文件的ctime可能会在mtime未发生任何变化时更改。例如，更改了文件的权限，但是文件内容没有变化,那么，如何获得一-个文件的atime、 mtime以及ctime呢? stat命令可用来列出文件的atime、ctime和mtime，示例命令如下:

```
[root@localhost dir3]# stat test
  文件："test"
  大小：24        	块：8          IO 块：4096   普通文件
设备：fd00h/64768d	Inode：100664227   硬链接：1
权限：(0700/-rwx------)  Uid：(    0/    root)   Gid：(    0/    root)
最近访问：2021-09-04 19:42:26.357349254 +0800
最近更改：2021-09-04 19:42:20.001332220 +0800
最近改动：2021-09-04 19:42:20.001332220 +0800
创建时间：-
```

atime不一定在访问文件之后被修改，因为在使用ext3文件系统时，如果mount使用了noatime参数，那么就不会更新atime的信息。总之，这三个time属性值都放在了inode中。若mtime、atime被修改，那么inode就一定会改,既然inode改了，那ctime也跟着要改了。

**find常用选项**

-   **-name filename** 表示直接查找该文件名的文件，这个选项比较常用，示例命令如下:

```
[root@localhost ~]# find . -name test3  //.表示在当前目录下寻找，不加也一样
./dir3/test3


[root@localhost ~]# cd /
[root@localhost /]# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
[root@localhost /]# find -name id_rsa
./root/.ssh/id_rsa
[root@localhost /]# 

太好用了！！！！！
```

-   -type filetype:表示通过文件类型查找文件。

[文件类型在前面已经简单介绍过]()，相信你已经基本了解了。filetype包含了f、b、c、 d、l、s等类型（注：这里用f代替-表示文件)，示例命令如下:

```
[root@localhost ~]# cd dir3
[root@localhost dir3]# mkdir g
[root@localhost dir3]# find -type d
.
./g
```

# linux文件类型

[文件类型在前面已经简单介绍过]()

### linux后缀名

在Linux系统中，**文件的后缀名没有具体意义**，加或者不加都无所谓。但是为了便于区分，我们习惯在定义文件名时加一个后缀名。这样当用户看到这个文件名时，就会很快知道它到底是一个什么文件，例如1.sh、2.tar.gz、my.cnf、test.zip等。

### 硬链接和软链接

-   **硬链接：** 给目标文件加上一个链接文件，链接文件直接指向目标文件，用户通过硬链接读取文件时，实际是读取硬链接文件的inode信息，并根据该信息到目标文件所在区域进行文件访问。一个目标文件可以有多个链接文件，这样删除一个链接文件时并不会删除目标文件。硬链接有两个限制: (1)不能跨文件系统，因为不同的文件系统有不同的inode table; (2)不能链接目录。
-   **软链接：** 与硬链接不同，软链接是建立一个独立的文件，当读取这个链接文件时，它会把读取的行为转发到该文件所链接的文件上。例如，现在有一个文件a，我们做了一个软链接文件b(只是一个链接文件，非常小)，b指向了a。当读取b时，b就会把读取的动作转发到a上，这样就读取了文件a。当我们删除文件a时，链接文件b不会被删除;但如果再次读取b时，会提示无法打开文件。然而,当我们删除b时，a是不会有任何影响的。

#### 创建链接

ln命令的格式为: **ln [-s][来源文件] [目的文件]** ，该命令常用的选项是-s。如果不加-s选项就是建立硬链接，加上-s选项就建立软链接。示例命令如下:

```
[root@localhost dir3]# du -k test
4	test
[root@localhost dir3]# du -k
0	./g
24	.
[root@localhost dir3]# ln test testh
[root@localhost dir3]# du -k    //创建硬链接后，目录下文件所占空间大小不变，说同一目标文件在目录下
																//只会占一份空间，硬链接只是inode信息而已
0	./g
24	.
```

若有收获，就点个赞吧

