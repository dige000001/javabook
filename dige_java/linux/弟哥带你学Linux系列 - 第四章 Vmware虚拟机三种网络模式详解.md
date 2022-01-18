## 公众号：“弟哥带你进大厂”，让你知识成体系的公众号，一定要关注，我们一起进大厂

## 一、Bridged（桥接模式）

在这种模式下，VMWare虚拟出来的操作系统就像是局域网中的一台独立的主机，它可以访问网内任何一台机器。在桥接模式下，你需要手工为虚拟 系统配置IP地址、子网掩码，而且还要和宿主机器处于同一网段，这样虚拟系统才能和宿主机器进行通信。同时，由于这个虚拟系统是局域网中的一个独立的主机 系统，那么就可以手工配置它的TCP/IP配置信息，以实现通过局域网的网关或路由器访问互联网。

使用桥接模式的虚拟系统和宿主机器的关系，就像连接在同一个Hub上的两台电脑。想让它们相互通讯，你就需要为虚拟系统配置IP地址和子网掩码，否则就无法通信。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9513f1fdb81d4b9b9e2f48190ca31d81~tplv-k3u1fbpfcp-zoom-1.image)

## 二、NAT（地址转换模式）

在NAT模式中，主机网卡直接与虚拟NAT设备相连，然后虚拟NAT设备与虚拟DHCP服务器一起连接在虚拟交换机VMnet8上，这样就实现了虚拟机联网。那么我们会觉得很奇怪，为什么需要虚拟网卡VMware Network Adapter VMnet8呢？原来我们的VMware Network Adapter VMnet8虚拟网卡主要是为了实现主机与虚拟机之间的通信。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6358311da16e496b941a0784b7485190~tplv-k3u1fbpfcp-zoom-1.image)

NAT模式下，子网ip由VmWare指定，网关地址可以随意设置，但要求在子网内

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50e6b16b703747d9a53e01e763481d38~tplv-k3u1fbpfcp-zoom-1.image)

## 三、Host-Only（仅主机模式）

Host-Only模式其实就是NAT模式去除了虚拟NAT设备，然后使用VMware Network Adapter VMnet1虚拟网卡连接VMnet1虚拟交换机来与虚拟机通信的，Host-Only模式将虚拟机与外网隔开，使得虚拟机成为一个独立的系统，**只与主机相互通讯。** 其网络结构如下图所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80320d93437c481b82b9041255ad7a1b~tplv-k3u1fbpfcp-zoom-1.image)