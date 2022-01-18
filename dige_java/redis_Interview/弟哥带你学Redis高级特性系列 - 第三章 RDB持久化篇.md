## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

关于save和bgsave相关在“使用篇”已补充

# RDB文件结构

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3942fcd4eb584027ad186ebb425cc1f5~tplv-k3u1fbpfcp-zoom-1.image)

注：

1.  全大写表示常量
1.  RDB文件保存的是二进制数据




-   **db_version长度为4字节**，它的值是一个字符串表示的整数，这个整数**记录了RDB文件的版本号**，比如"0006"就代表RDB文件的版本为第六版。本章只介绍第六版RDB文件的结构。
-   **databases**部分**包含着零个或任意多个数据库，以及各个数据库中的键值对数据**：

<!---->

-   -   如果服务器的数据库状态为空（所有数据库都是空的)，那么这个部分也为空，长度为0字节。
    -   如果服务器的数据库状态为非空（有至少一个数据库非空)，那么这个部分也为非空，根据数据库所保存键值对的数量、类型和内容不同，这个部分的长度也会有所不同。

<!---->

-   **EOF常量**的**长度为1字节，这个常量标志着RDB文件正文内容的结束**，当读入程序遇到这个值的时候，它知道所有数据库的所有键值对都已经载入完毕了。
-   **check _sum**是一个**8字节长的无符号整数**，保存着一个校验和，这个校验和是程序通过对REDIS、db_version、databases、EOF四个部分的内容进行计算得出的。服务器在载入RDB文件时，会将载入数据所计算出的校验和与check_sum所记录的校验和进行对比，以此来检查RDB文件是否有出错或者损坏的情况出现。

## databases

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/126d4fcdfdfc4970934ab63cb1913bc9~tplv-k3u1fbpfcp-zoom-1.image)

每个非空数据库在RDB文件中都可以保存为SELECTDB、db_number、key_value_pairs三个部分，如图10-13所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2251f3cfab724d278209debefcef3af2~tplv-k3u1fbpfcp-zoom-1.image)

-   SELECTDB长度为1字节，表明程序接下来读到的是数据库序号。
-   db_number保存着一个数据库号码，根据号码的大小不同，这个部分的长度可以是1字节、2字节或者5字节。当程序读入 db_number部分之后，服务器会调用SELECT命令，根据读入的数据库号码进行数据库切换，使得之后读入的键值对可以载入到正确的数据库中。

<!---->

-   key_value_pairs部分保存了数据库中的所有键值对数据，如果键值对带有过期时间，那么过期时间也会和键值对保存在一起。根据键值对的数量、类型、内容以及是否有过期时间等条件的不同，key_value_pairs部分的长度也会有所不同。

### key_value_pairs

RDB文件中的每个key_value_pairs部分都保存了一个或以上数量的键值对，如果键值对带有过期时间的话，那么键值对的过期时间也会被保存在内。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ca0f6f106eab4f4ba512ec4936d03894~tplv-k3u1fbpfcp-zoom-1.image)

若不带过期时间，则只有TYPE、key、value三个属性。

#### TYPE

TYPE记录了value的类型，长度为1字节,值可以是:

-   REDIS_RDB_TYPE_STRING
-   REDIS_RDB_TYPE_LIST

<!---->

-   REDIS_RDB_TYPE_SET
-   REDIS_RDB_TYPE_ZSET

<!---->

-   REDIS_RDB_TYPE_HASH
-   REDIS_RDB_TYPE_LIST_ZIPLIST

<!---->

-   REDIS_RDB_TYPE_SET_INTSET
-   REDIS_RDB_TYPE_ZSET_ZIPLIST

<!---->

-   REDIS RDB_TYPE_HASH_ZIPLIST

以上列出的每个TYPE常量都代表了一种对象类型或者底层编码，当服务器读入RDB文件中的键值对数据时，程序会根据TYPE的值来决定如何读入和解释value 的数据。key和 value分别保存了键值对的键对象和值对象：

其中 key总是一个字符串对象，它的编码方式和REDIS_RDB_TYPE_STRING类型的value 一样。根据内容长度的不同，key的长度也会有所不同。

根据TYPE类型的不同，以及保存内容长度的不同，保存value 的结构和长度也会有所不同

#### EXPIRETIME_MS和ms

-   EXPIRETIME_MS常量的长度为1字节，它告知读入程序，接下来要读入的将是一个以毫秒为单位的过期时间。
-   ms是一个8字节长的带符号整数，记录着一个以毫秒为单位的UNIX时间戳，这个

时间戳就是键值对的过期时间。

####

## value

### value的编码

#### 字符串对象

如果TYPE 的值为REDIS_RDB_TYPE_STRING，那么value保存的就是一个字符串对象，字符串对象的编码可以是REDIS_ENCODING_INT 或者REDIS_ENCODING_RAW。

如果字符串对象的编码为REDIS_ENCODING_INT，那么说明对象中保存的是长度不超过32位的整数，这种编码的对象将以图10-20所示的结构保存。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dd2e9bf74dd1430baf879745a049c98e~tplv-k3u1fbpfcp-zoom-1.image)

其中，ENCODING的值可以是REDIS_RDB_ENC_INT8、REDIS_RDB_ENC_INT16或者REDIS_RDB_ENC_INT32三个常量的其中一个，它们分别代表RDB文件使用8位( bit)、16位或者32位来保存整数值integer。

如果字符串对象的编码为REDIS_ENCODING_RAW，那么说明对象所保存的是一个字符串值，根据字符串长度的不同，有压缩和不压缩两种方法来保存这个字符串:

-   如果字符串的长度小于等于20字节，那么这个字符串会直接被原样保存。
-   如果字符串的长度大于20字节，那么这个字符串会被压缩之后再保存。

具体信息可以参考redis.conf文件中关于rdbcompression选项的说明。

对于没有被压缩的字符串，RDB程序会以图10-22所示的结构来保存该字符串。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bd5e51549404a4183de5fcfc13368ef~tplv-k3u1fbpfcp-zoom-1.image)

其中，string部分保存了字符串值本身，而len保存了字符串值的长度。对于压缩后的字符串，RDB程序会以图10-23所示的结构来保存该字符串。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70b492a5c4e24408b1b625de61af0580~tplv-k3u1fbpfcp-zoom-1.image)

其中，REDIS_RDB_ENC_LZF常量标志着字符串已经被LZF算法 压缩过了，读入程序在碰到这个常量时，会根据之后的compressed_len、origin_len和 compressed_string三部分，对字符串进行解压缩：其中compressed_len记录的是字符串被压缩之后的长度，而origin_len记录的是字符串原来的长度，compressed_string记录的则是被压缩之后的字符串。

#### 列表对象

如果TYPE的值为`REDIS_RDB_TYPE_LIST`，那么value保存的就是一个`REDIS_ENCODING_LINKEDLIST`编码的列表对象，RDB文件保存这种对象的结构如图10-26所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d97c3bad77ac46faadfafdd5342b6d9f~tplv-k3u1fbpfcp-zoom-1.image)

list_length记录了列表的长度，它记录列表保存了多少个项(item)，读入程序可以通过这个长度知道自己应该读入多少个列表项。

图中以item开头的部分代表列表的项，因为每个列表项都是一个字符串对象，所以程序会以处理字符串对象的方式来保存和读入列表项。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f519fbcfa4874a3bbc0dedcf5236a1d7~tplv-k3u1fbpfcp-zoom-1.image)

#### 集合对象

如果TYPE的值为`REDIS_RDB_TYPE_SET`，那么value保存的就是一个`REDIS_ENCODING_HT`编码的集合对象，RDB文件保存这种对象的结构如图10-28所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/64f44e36bff3471e8fb6c07e4b866c43~tplv-k3u1fbpfcp-zoom-1.image)

其中，set_size是集合的大小，它记录集合保存了多少个元素，读人程序可以通过这个大小知道自己应该读入多少个集合元素。

图中以elem开头的部分代表集合的元素，因为每个集合元素都是一个字符串对象，所以程序会以处理字符串对象的方式来保存和读入集合元素。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcc477e102eb408c95e8953e87543975~tplv-k3u1fbpfcp-zoom-1.image)

#### 哈希对象

如果TYPE的值为`REDIS_RDB_TYPE_HASH`，那么value保存的就是一个`REDIS_ENCODING_HT`编码的集合对象，RDB文件保存这种对象的结构如图10-30所示:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d65e7b248bd64d85997e3efb9c04586a~tplv-k3u1fbpfcp-zoom-1.image)

hash_size记录了哈希表的大小，也即是这个哈希表保存了多少键值对，读入程序可以通过这个大小知道自己应该读入多少个键值对。

以key_value _pair开头的部分代表哈希表中的键值对，键值对的键和值都是字符串对象，所以程序会以处理字符串对象的方式来保存和读入键值对。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6de86ca690824f43ac42d607226816c1~tplv-k3u1fbpfcp-zoom-1.image)

#### 有序集合对象

如果TYPE的值为`REDIS_RDB_TYPE_ZSET`，那么value保存的就是一个`REDIS_ENCODING_SKIPLIST`编码的有序集合对象，RDB文件保存这种对象的结构如图10-34所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71e8697a943f481596df46bacf02d605~tplv-k3u1fbpfcp-zoom-1.image)

sorted_set_size记录了有序集合的大小，也即是这个有序集合保存了多少元素，读入程序需要根据这个值来决定应该读入多少有序集合元素。

以element开头的部分代表有序集合中的元素，每个元素又分为成员( member )和分值( score）两部分，成员是一个字符串对象，分值则是一个double类型的浮点数，程序在保存RDB文件时会先将分值转换成字符串对象，然后再用保存字符串对象的方法将分值保存起来。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1fbbde019d83444687e64cb962f7e0f3~tplv-k3u1fbpfcp-zoom-1.image)

#### INTSET编码的集合

如果TYPE的值为`REDIS_RDB_TYPE_SET_INTSET`，那么value保存的就是一个整数集合对象，RDB文件保存这种对象的方法是，先将整数集合转换为字符串对象，然后将这个字符串对象保存到RDB文件里面。

如果程序在读入RDB文件的过程中，碰到由整数集合对象转换成的字符串对象，那么程序会根据TYPE值的指示，先读入字符串对象，再将这个字符串对象转换成原来的整数集合对象。

#### ZIPLIST编码的列表、哈希表或者有序集合

如果TYPE的值为`REDIS_RDB_TYPE_LIST_ZIPLIST`、`REDIS_RDB_TYPE_HASH_ZIPLIST`或者`REDIS_RDB_TYPE_ZSET_ZIPLIST`，那么value 保存的就是一个压缩列表对象，RDB 文件保存这种对象的方法是:

1.  将压缩列表转换成一个字符串对象。
1.  将转换所得的字符串对象保存到RDB文件。

如果程序在读入RDB文件的过程中，碰到由压缩列表对象转换成的字符串对象，那么程序会根据TYPE值的指示，执行以下操作:

1.  读入字符串对象，并将它转换成原来的压缩列表对象。
1.  根据TYPE的值，设置压缩列表对象的类型：

-   -   如果TYPE的值为REDIS_RDB_TYPE_LIST_ZIPLIST，那么压缩列表对象的类型为列表;
    -   如果TYPE的值为REDIS_RDB TYPE_HASH_ZIPLIST，那么压缩列表对象的类型为哈希表;

<!---->

-   -   如果TYPE的值为REDIS_ RDB_TYPE_ZSET_ZIPLIST，那么压缩列表对象的类型为有序集合。

从步骤2可以看出，由于TYPE的存在，即使列表、哈希表和有序集合三种类型都使用压缩列表来保存，RDB读入程序也总可以将读入并转换之后得出的压缩列表设置成原来的类型。

# RDB文件分析

## 带有过期时间的字符串键的RDB文件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92b51450e92b4ebb9b0f2e390423c556~tplv-k3u1fbpfcp-zoom-1.image)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8ed718940fdb4bbb8d8228a2312287ea~tplv-k3u1fbpfcp-zoom-1.image)

## 包含集合键的RDB文件

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/10fb993e0e6041aca4e2193035b52a30~tplv-k3u1fbpfcp-zoom-1.image)

# 总结

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/70fd8560b96e42bd99e11baeeb2e8817~tplv-k3u1fbpfcp-zoom-1.image)

