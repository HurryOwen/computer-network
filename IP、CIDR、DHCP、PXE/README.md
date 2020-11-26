# IP
**IP 地址是一个网卡在网络世界的通讯地址**，相当于我们现实世界的门牌号码。

IPv4 有 32 位，分为以下 5 类。
![](https://static001.geekbang.org/resource/image/fa/5e/fa9a00346b8a83a84f4c00de948a5b5e.jpg)

A、B、C 三类地址所包含的主机数量：
![](https://static001.geekbang.org/resource/image/6d/4c/6de9d39a68520c8e3b75aa919fadb24c.jpg)

问题：C 类地址能包含的最大主机数实在太少了，只有 254 个，而 B 类地址能包含的最大主机数又太多了，6 万多台机器放在一个网络下面，一般的企业基本达不到这个规模，闲着的地址就是浪费。

## 无类型域间选路（CIDR）
无类型域间选路，简称CIDR。打破了原来设计的几类地址的做法，将 32 位的 IP 地址一分为二，前面是**网络号**，后面是**主机号**。

例如，10.100.122.2/24，这个 IP 地址中有一个斜杠，斜杠后面有个数字 24。这种地址表示形式，就是 CIDR。后面 24 的意思是，32 位中，前 24 位是网络号，后 8 位是主机号。

伴随着 CIDR 存在的，一个是广播地址，10.100.122.255。如果发送这个地址，所有 10.100.122 网络里面的机器都可以收到。另一个是子网掩码，255.255.255.0。

**将子网掩码和 IP 地址按位计算 AND，就可得到网络号。**

## 公有 IP 地址和私有 IP 地址
虽然在日常的工作中，我们不分 A、B、C 类了，但我们还是要注意公有 IP 地址和私有 IP 地址。
![](https://static001.geekbang.org/resource/image/df/a9/df90239efec6e35880b9abe55089ffa9.jpg)

## MAC 地址
MAC 地址，是一个网卡的物理地址，用十六进制，6 个 byte 表示。

MAC 地址是一个很容易让人“误解”的地址。因为 MAC 地址号称全局唯一，不会有两个网卡有相同的 MAC 地址，而且网卡自生产出来，就带着这个地址。很多人看到这里就会想，既然这样，整个互联网的通信，全部用 MAC 地址好了，只要知道了对方的 MAC 地址，就可以把信息传过去。这样当然是不行的。 一个网络包要从一个地方传到另一个地方，除了要有确定的地址，还需要有定位功能。 而有门牌号码属性的 IP 地址，才是有远程定位功能的。

**MAC 地址更像是身份证，是一个唯一的标识。**它的唯一性设计是为了组网的时候，不同的网卡放在一个网络里面的时候，可以不用担心冲突。从硬件角度，保证不同的网卡有不同的标识。

MAC 地址是有一定定位功能的，只不过范围非常有限，局限在一个子网里面。

# 动态主机配置协议（DHCP）
**分配 IP 步骤：**

第一步，**DHCP Discover**。新来的机器使用 IP 地址 `0.0.0.0` 发送了一个广播包，目的 IP 地址为 `255.255.255.255`。广播包封装了 UDP，UDP 封装了 BOOTP。在这个广播包里面，新人大声喊：我是新来的（Boot request），我的 MAC 地址是这个，我还没有 IP，谁能给租给我个 IP 地址！

格式就像这样：

![](https://static001.geekbang.org/resource/image/90/81/90b4d41ee38e891031705d987d5d8481.jpg)

第二步，**DHCP Offer**。DHCP Server 能立刻知道来了一个新 MAC，需要给他提供一个 IP。DHCP Server 仍然使用广播地址作为目的地址，因为，此时请求分配 IP 的新人还没有自己的 IP。DHCP Server 回复说，我分配了一个可用的 IP 给你，你看如何？除此之外，服务器还发送了子网掩码、网关和 IP 地址租用期等信息。

格式如下：
![](https://static001.geekbang.org/resource/image/a5/6b/a52c8c87b925b52059febe9dfcd6be6b.jpg)

第三步，新机器选择其中一个 DHCP Offer，并像网络发送一个 DHCP Request 广播数据包，包中包含客户端的 MAC 地址、接受的租约中的 IP 地址、提供此租约的 DHCP 服务器地址等，并告诉所有 DHCP Server 它将接受哪一台服务器提供的 IP 地址。

此时，由于还没有得到 DHCP Server 的最后确认，客户端仍然使用 0.0.0.0 为源 IP 地址、255.255.255.255 为目标地址进行广播。在 BOOTP 里面，接受某个 DHCP Server 的分配的 IP。

格式如下：
![](https://static001.geekbang.org/resource/image/cd/fa/cdbcaad24e1a4d24dd724e38f6f043fa.jpg)

第四步，当 DHCP Server 接收到客户机的 DHCP request 之后，会广播返回给客户机一个 DHCP ACK 消息包，表明已经接受客户机的选择，并将这一 IP 地址的合法租用信息和其他的配置信息都放入该广播包，发给客户机。

格式如下：

![](https://static001.geekbang.org/resource/image/cc/a9/cca8b0baa4749bb359e453b1b482e1a9.jpg)

## 预启动执行环境（PXE）
可以通过 PXE 自动安装操作系统。

操作系统启动过程：
- 启动 BIOS。这是一个特别小的小系统，只能干特别小的一件事情。就是读取硬盘的 MBR 启动扇区，将 GRUB 启动起来；
- 将权力交给 GRUB，GRUB 加载内核、加载作为根文件系统的 initramfs 文件；
- 将权力交给内核，内核启动，初始化整个操作系统。

那我们安装操作系统的过程，只能插在 BIOS 启动之后了。因为没安装系统之前，连启动扇区都没有。因而这个过程叫做**预启动执行环境**（Pre-boot Execution Environment），简称 **PXE**。

PXE 协议分为客户端和服务器端，由于还没有操作系统，只能先把客户端放在 BIOS 里面。当计算机启动时，BIOS 把 PXE 客户端调入内存里面，就可以连接到服务端做一些操作了。

首先，PXE 客户端自己也需要有个 IP 地址。因为 PXE 的客户端启动起来，就可以发送一个 DHCP 的请求，让 DHCP Server 给它分配一个地址。除了分配地址之外，还可以配置 `next-server`，指向 PXE 服务器的地址，以及配置初始启动文件 `filename`。

DHCP Server 的样例配置：
```
ddns-update-style interim;
ignore client-updates;
allow booting;
allow bootp;
subnet 192.168.1.0 netmask 255.255.255.0
{
option routers 192.168.1.1;
option subnet-mask 255.255.255.0;
option time-offset -18000;
default-lease-time 21600;
max-lease-time 43200;
range dynamic-bootp 192.168.1.240 192.168.1.250;
filename "pxelinux.0";
next-server 192.168.1.180;
}
```
这样 PXE 客户端启动，发送 DHCP 请求之后，除了能得到一个 IP 地址，还可以知道 PXE 服务器在哪里，也可以知道如何从 PXE 服务器上下载某个文件，去初始化操作系统。

**工作过程：**
第一步，启动 PXE 客户端。通过发送 DHCP 请求，得到自己的 IP 地址，以及 PXE 服务器的地址、启动文件 `pxelinux.0`。

第二步，PXE 客户端知道要去 PXE 服务器下载这个文件后，就可以初始化机器。于是便开始下载，下载的时候使用的是 TFTP 协议。

第三步，PXE 客户端收到这个文件后，就开始执行这个文件。这个文件会指示 PXE 客户端，向 TFTP 服务器请求计算机的配置信息 pxelinux.cfg。TFTP 服务器会给 PXE 客户端一个配置文件，里面会说内核在哪里、initramfs 在哪里。PXE 客户端会请求这些文件。

第四步，启动 Linux 内核。

![](https://static001.geekbang.org/resource/image/bb/8e/bbc2b660bba0ad00b5d1179db158498e.jpg)

# 总结
- DHCP 协议主要是用来给客户租用 IP 地址，和房产中介很像，要商谈、签约、续租，广播还不能“抢单”；
- DHCP 协议能给客户推荐“装修队”PXE，能够安装操作系统，这个在云计算领域大有用处。