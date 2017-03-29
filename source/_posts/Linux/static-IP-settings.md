---
categories: Linux
date: 2016-09-06 11:10:00 +0800
status: public
title: VMWare中Ubuntu静态IP的配置
---

## 背景
***
搭建Hadoop集群想要为每台虚拟机设置静态IP

## 1.理论基础
***
VMware在默认安装完成之后，会创建三个虚拟的网络环境：VMnet0、VMnet1和VMnet8。其类型分别为：桥接网络，Host-only和NAT。
### 三种方式的区别
  1. **bridged(桥接模式)**:默认使用VMnet0，不提供DHCP服务。
  &emsp;&emsp; 在桥接模式下，虚拟机和宿主计算机处于同等地位，虚拟机就像是一台真实主机一样存在于局域网中。因此在桥接模式下，我们就要像对待其他真实计算机一样为其配置IP、网关、子网掩码等等。
 &emsp;&emsp; 当我们可以自由分配局域网IP时，使用桥接模式就可以虚拟出一台真实存在的主机。
 
  2. **NAT(网络地址转换模式)**:默认使用VMnet8，提供DHCP服务
  &emsp;&emsp; 在NAT模式下，宿主计算机相当于一台开启了DHCP功能的路由器，而虚拟机则是内网中的一台真实主机，通过路由器(宿主计算机)DHCP动态获得网络参数。因此在NAT模式下，虚拟机可以访问外部网络，反之则不行，因为虚拟机属于内网。
  &emsp;&emsp; 使用NAT模式的方便之处在于，我们不需要做任何网络设置，只要宿主计算机可以连接到外部网络，虚拟机也可以。
  &emsp;&emsp; NAT模式通常也是大学校园网Vmware最普遍采用的连接模式，因为我们一般只能拥有一个外部IP。很显然，在这种情况下，非常适合使用NAT模式。

  3. **Host-only(主机模式)**:默认使用VMnet1，提供DHCP服务
  &emsp;&emsp; 在Host-only模式下，相当于虚拟机通过双绞线和宿主计算机直连，而宿主计算机不提供任何路由服务。因此在Host-only模式下，虚拟机可以和宿主计算机互相访问，但是虚拟机无法访问外部网络。
  &emsp;&emsp; 当我们要组成一个与物理网络相隔离的虚拟网络时，无疑非常适合使用Host-only模式。

## 2.设置VMWare网络环境
***
这里，我们选择NAT方式，来实现Ubuntu的静态IP地址配置。
1. 打开VMware，在顶部依次选择：编辑 > 虚拟网路编辑器，打开虚拟网路编辑器：
2. 首先，去掉如下图中的“使用本地DHCP服务将IP地址分配给虚拟机”，此外，这里设置子网IP为：192.168.101.0，子网掩码为：255.255.255.0。
3. 得到可用IP范围、网关。
  &emsp;&emsp; 选择VMnet8条目，点击NAT Settings按钮后可以看到我们的VMWare Workstation为NAT连接的虚拟机设定的默认网关，此处为192.168.101.2，以及子网掩码，此处为255.255.255.0。
  &emsp;&emsp; 点击DHCP Settings按钮，可以看到VMnet8为虚拟机分配的可用的子网IP范围。

## 3.设置Ubutnu静态IP
***
### 方法1：通过网路管理面板设置IP

  &emsp;&emsp; 在Ubuntu桌面的右上角，点击网络图标，然后选择“Edit Connections”：
1. 点击“Edit”按钮，打开编辑页面
2. Method：选择Manual
3. 将IP地址填入Addresses栏
这里，我们设置的IP地址为： IP： 192.168.101.200 子网掩码： 255.255.255.0 网关： 192.168.101.2
然后，选择保存。
最后，点击Ubuntu桌面右上角的网络图标，选择“Disconnect”，断开连接。然后再打开该菜单，选择"Connect"，即可连接上网。

### 方法2：通过配置文件
打开Ubuntu的终端，输入：
```bash
sudo gedit /etc/network/interfaces
```

表示使用gedit编辑器打开interfaces文件。 在打开的文件中，若有内容，先全部删除。然后输入如下代码：
```bash
auto lo
iface lo inet loopback
auto ens33 ##(此处名称需要通过命令 ifconfig 查看)
iface ens33 inet static
address 192.168.101.200
netmask 255.255.255.0
gateway 192.168.101.2
```
修改保存后，重启网络即可：
```bash
 sudo /etc/init.d/networking restart
 ```
> 注：
重启系统之后，发现网络无法使用，右上角的网络图标点击之后显示“device not managed（设备未托管）”

解决方法：
```bash
sudo gedit /etc/NetworkManager/NetworkManager.conf
```
打开该文件，将“`managed=false`”修改为“`managed=true`”。
重启network manager，即可解决问题。
```bash
sudo service network-manager restart
```