## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

# 压缩

Linux下最常见的压缩文件通常都是.tar.gz格式的，除此之外，还有.tar 、.gz 、.bz2 、.zip等格式。Linux下的文件后缀名可加可不加，但压缩文件最好加上后缀名。这是为了判断压缩文件是由哪种压缩工具所压缩的，而后才能正确地解压缩这个文件。下面介绍Linux下常见的后缀名所对应的压缩t具。

```
.gz ：表示由gzip压缩工具压缩的文件。
.bz2 ：表示由bzip2压缩工具压缩的文件。
.tar ：表示由tar打包程序打包的文件（tar并没有压缩功能，只是把一个目录合并成一个文件）。
.tar.gz ：可以理解为先由tar打包，然后再由gzip压缩。
.tar.bz2 ：可以理解为先由tar打包，然后再由bzip2压缩。
.tar.xz ：可以理解成先由tar打包，然后再xz压缩。
```

## gzip压缩工具

gzip命令的格式为**gzip [-d#] filename**，其中＃为1-9的数字。

```
-d：该参数在解压缩时使用。
-#：表示压缩等级，l为最差，9为最好 6为默认。
```

gzip不支持压缩目录，压缩目录时会报错
Linux压缩保留源文件的方法： 

```
[root@localhost dir3]# gzip aaa
[root@localhost dir3]# ls
11887022.html  aaa.gz  g  test  test2  test3  testh //会将原文件删除

[root@localhost dir3]# gzip -c aaa > aaa.gz    //这样不会删除原文件
[root@localhost dir3]# ls
11887022.html  aaa  aaa.gz  g  test  test2  test3  testh

解压缩同理
[root@localhost dir3]# gzip -cd aaa.gz > gi //这样就会保留原压缩文件
[root@localhost dir3]# ls
11887022.html  aaa.gz  g  gi  test  test2  test3  testh
```

## bzip2压缩工具

bzip2命令的格式为**bzip2 [-dz] filename**，它只有－z（压缩）和－d（解压缩）两个常用选项。压缩级别有1~9，默认级别是9。压缩时，加或不加－z选项都可以压缩文件。

使用和gzip一样，都不能压缩文件夹

## xz压缩工具

和bzip2一样，都不能压缩文件夹

## 使用zip压缩

zip 压缩包在Windows和Linux中都比较常用，它可以压缩目录和文件，压缩目录时，需要指定目录下的文件。示例命令如下：

zip后面先跟目标文件名，即压缩后的自定义压缩包名，然后跟要压缩的文件或者目录。

当目录下还有二级目录甚至更多级目录时， zip命令仅仅是把二级目录本身压缩而已。如果想要一并压缩二级目录下的文件，必须加上-r选项，如下所示

```
[root@localhost dir3]# zip  tesallt.zip g33
  adding: g33/ (stored 0%)

[root@localhost dir3]# zip -r tesallt.zip g33
  adding: g33/ (stored 0%)
  adding: g33/test2 (stored 0%)
  adding: g33/test3 (stored 0%)
  adding: g33/file1.tar (deflated 100%)
  adding: g33/g44/ (stored 0%)
  adding: g33/g44/g55/ (stored 0%)
  adding: g33/g44/g55/g66/ (stored 0%)
  adding: g33/g44/g55/g66/g77/ (stored 0%)
  adding: g33/g44/g55/g66/g77/ji (stored 0%)
```

解压.zip格式文件时并不用zip命令，而是用unzip命令

# tar打包工具

tar本身就是一个打包工具，可以把目录打包成一个文件，它把所有文件整合成一个大文件，方便复制或者移动。该命令的格式为**tar [-zjxcvfpP] filename.tar file1 file2 file3 ...** ，

```
-z:表示同时用gzip压缩。解压时加上该参数表示解压该类型文件
-j:表示同时用bzip2压缩。解压时加上该参数表示解压该类型文件
-J:表示同时用xz压缩。解压时加上该参数表示解压该类型文件
-x:表示解包或者解压缩。
-t:表示查看tar包里的文件。
-c:表示建立一个tar包或者压缩文件包。
-v:表示可视化。
-f:后面跟文件名(即-f filename，表示压缩后的文件名为filename，或者解压文件f需要注意的是，如果是多个参数组合的情况下，请把-f参数写到最后面。
-p:表示使用原文件的属性，压缩前什么属性压缩后还什么属性。(不常用)
-P:表示可以使用绝对路径。(不常用)
--exclude filename:表示在打包或压缩时，不要将filename文件包括在内。(不常用
```

其实不管是打包还是解包，原来的文件是不会删除的，而且它会覆盖当前已经存在的文件或者目录。

# 查看压缩文件

上面介绍了使用－t选项可以查看tar压缩包的文件列表。对于gzip2或者bzip2压缩格式的文本文档，我们也可以使用zcat、bzcat命令直接查看文档内容。示例命令如下：

```
[root@localhost dir3]# bzcat test.bz2
asdasda
asd
ad
asd
```

若有收获，就点个赞吧


