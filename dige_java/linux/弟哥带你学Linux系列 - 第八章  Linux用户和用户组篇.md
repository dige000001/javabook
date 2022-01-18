## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 认识/etc/passwd和/etc/shadow

## /etc/passwd

```
[root@localhost dir3]# cat /etc/passwd | head
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
```

-   第1个字段为用户名（如第1行中的root就是用户名)，它是代表用户账号的字符串。用户名中的字符可以是大小写字母、数字、减号(不能出现在首位)、点或下划线，其他字符不合法。虽然用户名中可以出现点，但不建议使用，尤其是首位。另外，减号也不建议使用，容易造成混淆。
-   第2个字段存放的是该账号的口令。这里为什么是x呢?早期的Unix系统口令确实存放在这里，但基于安全因素，后来就将其存放到/etc/shadow中了，这里只用一个x代替。

<!---->

-   第3个字段为一个数字，这个数字代表用户标识号，也称为uid。系统就是通过这个数字识别用户身份的。这里的0就是root，也就是说我们可以修改test用户的uid为0，那么系统会认为root和test为同一个账户。uid的取值范围是0~65535(但实际上已经可以支持到4294967294),0是超级用户( root )的标识号，CentOS 7的普通用户标识号从1000开始。如果我们自定义建立一个普通用户，你会看到该账户的标识号是大于或等于1000的。
-   第4个字段也是数字，表示组标识号，也称为gid。这个字段对应着/etc/group中的一条记录，其实/etc/group和/etc/passwd基本类似。

<!---->

-   第5个字段为注释说明，没有实际意义。通常记录该用户的一些属性，例如姓名、电话、地址等。我们可以使用chfn命令来更改这些信息,这在稍后会介绍。
-   第6个字段为用户的家目录，当用户登录时，就处在这个目录下。root的家目录是/root，普通用户的家目录则为/home/username，用户家目录是可以自定义的。比如，建立一个普通用户test1，要想让testl的家目录在/data目录下，只要将/etc/passwd文件中对应该用户那行中的本字段修改为/data即可。

<!---->

-   最后一个字段为用户的shell。用户登录后，要启动一个进程，用来将用户下达的指令传给内核，这就是shell。Linux的shell有sh、csh、ksh、tcsh、bash等多种，而Red Hat/CentOS的shell就是bash。查看/etc/passwd文件，该字段中除了/bin/bash，还有很多/sbin/nologin，它表示不允许该账号登录。如果想建立一个不允许登录的账号，可以把该字段改成/sbin/nologin，默认是/bin/bash。

## /etc/shadow

```
[root@localhost dir3]# cat /etc/shadow | head
root:$6$5XpRH4aChDahOuwA$.iP0/uXEyirf6HHuk9AAHWCbh/cwuVCQ5en17IVadJXPm3ALEKRytgArEdOfcvlYZl1BnNEMn9igtTeyRSeGH1::0:99999:7:::
bin:*:18353:0:99999:7:::
daemon:*:18353:0:99999:7:::
adm:*:18353:0:99999:7:::
lp:*:18353:0:99999:7:::
sync:*:18353:0:99999:7:::
shutdown:*:18353:0:99999:7:::
halt:*:18353:0:99999:7:::
mail:*:18353:0:99999:7:::
operator:*:18353:0:99999:7:::
```

-   第1个字段为用户名，与/etc/passwd对应。
-   第2个字段为用户密码，是该账号的真正密码。这个密码已经加密，但是有些黑客还是能够解密的。所以，将该文件属性设置为o00，但root账户是可以访问或更改的。

<!---->

-   第3个字段为上次更改密码的日期，这个数字以1970年1月1日和上次更改密码的日期为基准计算而来。例如，上次更改密码的日期为2012年1月1日，则这个值就是365*(2012-1970)+(2012-1970)/4+1=15341。如果是闰年，则有366天。
-   第4个字段为要过多少天才可以更改密码,默认是0，即不受限制。

<!---->

-   第5个字段为密码多少天后到期，即在多少天内必须更改密码。例如，这里设置成30，则30天内必须更改一次密码;否则，将不能登录系统。默认是99999，可以理解为永远不需要改。
-   第6个字段为密码到期前的警告期限。若这个值设置成7，则表示当7天后密码过期时，系统就发出警告，提醒用户他的密码将在7天后到期。

<!---->

-   第7个字段为账号失效期限。如果这个值设置为3，则表示密码已经到期，然而用户并没有在到期前修改密码，那么再过3天，这个账号便失效，即锁定。
-   第8个字段为账号的生命周期。跟第3个字段一样，这个周期是按距离1970年1月1日多少天算的。它表示的含义是，账号在这个日期前可以使用，到期后账号将作废。

<!---->

-   最后一个字段作为保留用的，没有什么意义。

# 用户和用户组管理

## 添加用户组groupadd

```
[root@localhost dir3]# groupadd gi
[root@localhost dir3]# tail -n1 /etc/group
gi:x:1002:

[root@localhost dir3]# groupadd -g 1008 g3  //通过-g指定groupid
[root@localhost dir3]# tail -n2 /etc/group  
gi:x:1002:
g3:x:1008:
```

## 删除用户组groupdel

```
[root@localhost dir3]# groupdel g3
[root@localhost dir3]# tail -n1 /etc/group
gi:x:1002:
```

注意：当用户组里有用户时无法删除用户组

## 增加用户useradd

从字面意思上来看，useradd就是增加用户，该命令的格式为

**useradd [-u UID] [-g GID] [-d HOME][-M][-s] username**，其中各个选项的具体含义如下。

```
-u:表示自定义UID。
-g:表示使新增用户属于已经存在的某个组，后面可以跟组id，也可以跟组名。
-d:表示自定义用户的家目录。
-M:表示不建立家目录。即/home下没有 username这个文件夹
-s:表示自定义shell。
```

```
[root@localhost dir3]# useradd -u 114514 -g 1002 ts
[root@localhost dir3]# tail -n1 /etc/passwd
ts:x:114514:1002::/home/ts:/bin/bash
[root@localhost dir3]# ls /home
ts  user1  wu
```

## 删除用户userdel

命令userdel的格式为**userdel [-r] username**，其中-r选项的作用是，当删除用户时，一并删除该用户的家目录。示例命令如下:

```
[root@localhost dir3]# userdel ts
[root@localhost dir3]# ls /home
ts  user1  wu

[root@localhost dir3]# userdel -r ts
[root@localhost dir3]# ls /home
user1  wu
```

## 修改密码passwd

为用户设置密码时，可以使用命令passwd，其格式为**passwd [username]** 。该命令后面若不加用户名字，则是为自己设定密码，示例命令如下:

## 切换用户su

命令su的格式为**su [-] username**，后面可以跟-，也可以不跟。普通用户的su命令不加username时，就是切换到root用户。当然，root用户同样可以使用su命令切换到普通用户。该命令加上-后，会初始化当前用户的各种环境变量。

## 超级命令sudo

sudo可以让普通用户对某一指令拥有root权限，比如说切换到普通用户时，在/root文件夹下是不能使用ls的，而使用sudo ls则可以，使用sudo命令需要root在/etc/sudoers中为普通用户配置权限：

```
[root@localhost ~]# visudo
//在里面添加 username ALL=(ALL) 	ALL
```

就可以让普通用户拥有sudo的特权。从左到右,第一段username这里为一个用户，指定让哪个用户有sudo特权；第二段ALL=(ALL)比较难理解，左边的ALL指的是所有的主机，右边的ALL指的是获取哪个用户的身份，第二段几乎都不用配置；第三段设定可以使用sudo的命令有哪些。

此时普通用户可以在/root文件夹下使用sudo

```
[root@localhost ~]# su ts
[ts@localhost root]$ sudo ls
[sudo] ts 的密码：
anaconda-ks.cfg  dir3  file1
```

如果每增加一个用户就设置一行，这样太麻烦了，所以可以这样设置:把#%wheel ALL=(ALL)ALL前面的#去掉，让这一行生效。它的意思是，wheel这个组的所有用户都拥有了sudo的权利。接下来，只要把需要设置sudo权限的所有用户加入到wheel这个组中即可。如下所示:

```
%wheel ALL=(ALL) ALL
```


