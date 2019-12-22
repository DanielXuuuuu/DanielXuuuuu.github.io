---
title: Sevice Computing - Windows环境下使用VirtualBox搭建云桌面
date: 2019-09-01 12:44:38
categories: 
	- 服务计算
tags: 
	- 云桌面
	- 服务计算
	- CentOS
	- ssh
---

> 这是我在服务计算课程学习中的第一次实验，记录了我跟随老师的教程一步一步的在Windows环境下使用VirtualBox搭建云桌面的过程，过程虽并不难，但也遇到了一些小问题。

#### 实验目的

- 初步了解虚拟化技术，理解云计算的相关概念
- 理解系统工程师面临的困境
- 理解自动化安装、管理（DevOps）在云应用中的重要性

<!--more-->

#### 实验环境与要求

- 用户通过互联网，使用微软远程桌面，远程访问你在PC机上创建的虚拟机
- 主机：Windows 10；虚拟机软件：VirtualBox，虚拟机操作系统：Centos

#### 实验步骤及结果

##### 安装VirtualBox

- 首先安装好git客户端（git bash）以及VirtualBox，由于这两个我之前都已安装好，且安装过程较为简单，这里就不再赘述了。
- 创建虚拟机内部虚拟网络，使得Vbox内部虚拟机可以通过它，实现虚拟机之间、虚拟机与主机的通讯。
  - VirtualBox菜单 ：管理 -> 主机网络管理器，创建一块虚拟网卡，网址分配：192.168.100.1/24，如下图所示：![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901101648126.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
  - 点击应用后，在主机 Windows 命令行窗口输入 ipconfig 可以看到 VirtualBox Host-Only Network #2 的网卡：
    ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901102017272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)

##### 创建并配置Centos虚拟机

- 首先下载Centos镜像（只需Minimal ISO即可）：[Centos官网](https://www.centos.org/download/)
- 使用Vbox创建虚拟机，此过程与一般虚拟机创建流程一致，建议内存不低于2G，存储不低于30G，虚拟机的命名我设置为centos_base。
- 创建完成后，右键点击虚拟机选择设置，选择网络，第一块网卡必须是 NAT，然后启用第二块网卡，连接方式为 Host-Only，并选用之前创建的虚拟网卡。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901102850957.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 启动虚拟机，根据指令进行安装，并完成用户名，密码等一系列的设置。
- 登陆系统，通过 nmtui 指令进入配置网络的UI界面：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901103813602.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 选择“Edit a connection”，然后对其中的enp0s8进行编辑，修改其地址为192.168.100.5/24：![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901112024352.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 保存后返回上一级，选择“Activate a connection”，将enp0s3和enp0s8都激活，如下图所示：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901104028652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 退出配置界面，此时，ping Windows主机 和 ping 外网应该是都可以 ping 通的：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019090110433936.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 完成上述操作后，还可以进行OS系统的升级，依次执行命令`yum install wget`，`yum update`。这时，第一台虚拟机就配置好了，关闭该虚拟机。

##### 复制Centos虚拟机

- 右键点击centos_base虚拟机，选择复制，输入新虚拟机的名 centos_temp。注意必须选择为所有网卡重新生成 MAC 地址。然后在下一步中选择链接复制。
- 然后进行和之前相同的操作，使用 nmtui 修改主机名和第二块网卡IP地址。注意此时不可以和之前完全相同，例如我将地址设置为192.168.100.10/24，即前 24 位保持不变，使其位于同一子网下![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901112148510.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)

##### 测试两个虚拟机 

- 完成以上配置后，我们同时启动两个虚拟机。在Windows主机上，应能 ping 通它们：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901112448611.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 且能通过 ssh 访问两个虚拟机（使用git bash）。如下图所示，centos_base和centos_temp均可连接上：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901112914233.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)

##### 配置用远程桌面访问你的虚拟机

> 详细教程见：[如何设置VirtualBox虚拟机远程访问模式](https://www.jianshu.com/p/6f0f35fa2c4f)

- 简单来说，首先去[官网](https://www.oracle.com/virtualization/technologies/vm/downloads/virtualbox-downloads.html)下载VBox的拓展包
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901113137109.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 然后从VBox菜单的管理–>全局设定–>扩展中导入刚下载的扩展包。
- 然后右击虚拟机–>设置–>显示–>远程桌面，启用服务器并自行设置好一个端口（两台虚拟机需要设置不同的端口号避免冲突），保存设置，然后启动该虚拟机。如下图，我设置的端口号为2629：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901113440172.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 在Windows10中启动远程桌面程序，输入之前创建网卡的地址加上刚刚设置的端口号，连接，即可在宿主机上“远程使用”自己的centos虚拟机了。
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901113954687.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901114319665.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 同理，我另一个虚拟机使用的是3030端口：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190901114419523.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1doaXRlM3p6,size_16,color_FFFFFF,t_70)
- 可以看到，成功连接了centos_base和centos_temp两台虚拟机。
- 至此，实验完成。

#### 实验难点(“坑”)

+ 在使用Windows远程桌面连接虚拟机时，不可以想当然的去用我们刚刚为虚拟机配置的IP地址，而是要使用的是宿主机的IP地址，不是子机，即我们最初为VirtualBox配置的网卡IP地址：192.168.100.1。经过测试，我发现使用主机的自身IP地址（例如无线网卡地址也是可以的）
+ 在虚拟机中对于 enp0s8 设置IP地址时，不要直接设置为我们最初设置的网卡地址192.168.100.1/24，而应该使其位于同一网段下即可，即保持前24位一致。否则，会出现ssh时无法连接的情况。

#### 实验心得

- 之前也接触并使用过linux操作系统，但都仅限于编程，没有进行过如上所述的一些网络配置，收获还是挺大的，并且完成了远程云桌面的设置，对于实验的结果比较满意。
- 虽然整体的实验并不难，但也在一些小细节上卡了几次，比如就像连接远程桌面的时候，我使用了centos中设置的IP地址，导致无法连接上。在仔细阅读了老师给出的资料后，才发现原来是要使用宿主机的IP地址。
- 通过本次实验，让我体会到了配置环境的重要性，这是以后进一步学习的基石，让我收获很大。