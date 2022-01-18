## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

本章将采用类似C语言结构体的伪结构来描述class 文件格式。为了避免与类的字段、类的实例等概念产生混淆，我们用项( item)来称呼class文件格式各结构体中的内容。 Class文件是一组以8个字节为基础单位的二进制流 ，**各项按照严格顺序连续存放的，它们之间没有用任何填充或对齐作为各项间的分隔符号**。

**表( table)由任意数量的可变长度的项组成**，用于表示class文件内容的一系列复合结构。为了便于区分，所有表的命名 都习惯性地以“_info”结尾。 表是由多个无符号数或者其他表作为数据项构成的可变长复合数据类型 (表中每项的长度不固定)，因此无法直接把表格索引转换为偏移量，并以此来访问表中的项。

# classFile结构

每个class 文件对应一个如下所示的ClassFile 结构。

**注：un的n那个数字，表示该项有几个字节**

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e40b87ae38f421f838233d0c8d64e6b~tplv-k3u1fbpfcp-zoom-1.image)

在ClassFile 结构中，各项的含义描述如下:

**Magic(魔数)**

Magic的唯一作用是确定这个文件是否为一个能被虚拟机所接受的class文件。魔数值固定为OxCAFEBABE，不会改变。合理怀疑这个魔数预示着现在java“咖啡”的logo

**minor_version(副版本号)、major_version(主版本号)**

minor_version和major_version的值分别表示class 文件的副、主版本。它们共同构成了class文件的格式版本号。比如，某个class文件的主版本号为M，副版本号为m，那么这个class文件的格式版本号就确定为M.m。class文件格式版本号可按字面顺序来排序，例如:1.5<2.0<2.1。

假设一个class文件的格式版本号为v，那么，当且仅当Mi.0 ≤v ≤ Mj.m成立时，这个class文件才可以被此Java 虚拟机支持。Java虚拟机实现会遵从Java SE平台的某个发行版级别( the release level of the Java SE platform)，而这个发行版级别决定了本虚拟机所能支持的版本范围。

**constant_pool_count（常量池计数器)**

constant_pool_count 的值等于**常量池表中的成员数加1**。常量池表的索引值只有在大于0且小于constant_pool_count时才会认为是有效的°，对于long和double类型有例外情况，参见《java虚拟机规范》4.4.5小节。

**constant_pool[]（常量池)**

constant_pool是一种表结构， 常量池中每一项常量都是一个表 (见《java虚拟机规范》4.4节)，它包含class文件结构及其子结构中引用的所有字符串常量、类或接口名、字段名和其他常量等。常量池中的每一项都具备相同的特征——第1个字节作为类型标记，用于确定该项的格式，这个字节称为tagbyte(标记字节、标签字节)。

常量池以1 ~ constant_pool_count-1为索引。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ccba91cdbab64d8480a8ba8147a685c4~tplv-k3u1fbpfcp-zoom-1.image)

**access_flags(访问标志)**

access_flags是一种由标志所构成的掩码，用于表示某个类或者接口的访问权限及属性。每个标志的取值及其含义如表4-1所示。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/72cd01fa68f64689a2fe936551ae3224~tplv-k3u1fbpfcp-zoom-1.image)

具体含义（见《java虚拟机规范》4.1节)

**interfaces_count(接口计数器)**

interfaces_count项的值表示当前类或接口的直接超接口数量。

**interfaces [ ](接口表)**

interfaces[]中**每个成员的值必须是对常量池表中某项的有效索引值**，它的长度为interfaces_count。每个成员interfaces[i]必须为CONSTANT_Class_info结构(见4.4.1小节)，其中0≤i<interfaces_count。在interfaces[]中,各成员所表示的接口顺序和对应的源代码中给定的接口顺序(从左至右)一样，即interfaces [0]对应的是源代码中最左边的接口。

**fields_count(字段计数器)**

fields_count的值表示当前class文件fields表的成员个数。fields表中每个成员都是一个field_info结构(见4.5节)，用于表示该类或接口所声明的类字段或者实例字段°。

**fields[](字段表)**

fields表中的每个成员都必须是一个fields_info结构(见4.5节)的数据项，用于表示当前类或接口中某个字段的完整描述。fields表描述当前类或接口声明的所有字段，但不包括从父类或父接口继承的那些字段。

**methods_count (方法计数器)**

methods_count的值表示当前class文件methods表的成员个数。methods表中每个成员都是一个method_info结构（见4.5节)。

**methods[](方法表)**

methods表中的每个成员都必须是一个method_info结构(见4.6节)，用于表示当前类或接口中某个方法的完整描述。如果某个method_info结构的access_flags项既没有设置ACC_NATIVE标志也没有设置AcC_ABSTRACT标志，那么该结构中也应包含实现这个方法所用的Java虚拟机指令。

method_info结构可以表示类和接口中定义的所有方法，包括实例方法、类方法、实例初始化方法（见2.9节）和类或接口初始化方法(（见2.9节)。methods表只描述当前类或接口中声明的方法，不包括从父类或父接口继承的方法。

**attributes_count(属性计数器)**

attributes_count 的值表示当前class文件属性表的成员个数。属性表中每一项都是一个attribute_info结构（见4.7节)。

**attributes[](属性表)**

属性表的每个项的值必须是attribute_info结构（见4.7节或《深入理解jvm》6.3.7)

由本规范所定义，且能够出现在ClassFile结构体attributes表中的各项属性( attribute)，列在**表4-8**里。与classFile结构体attributes表里预定义的各项属性有关的规则，列在**4.7节**中。

与classFile结构体attributes表里非预定义的( non-predefined)各项属性有关的规则，列在**4.7.1小节**中。



