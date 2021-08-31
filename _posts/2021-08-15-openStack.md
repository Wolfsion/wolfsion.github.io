---
layout: post
title:  "OpenStack基础学习"
date:   2021-08-15
excerpt: "云计算初次学习——含openStack搭建"
tags: [云计算]
comments: true
---
# OpenStack

## 简述

OpenStack从一开始，就是为了云计算服务的。简单来说，它就是一个操作系统，一套软件，一套IaaS软件。



## 云计算

+ 概念：云化就是把每个人手中的独立资源集中起来，放在一个地方进行统一管理，然后动态分配给每个人使用。而云计算，就是把计算资源集中起来，这个计算资源，包括CPU、内存、硬盘等硬件，还有软件。
+ 云计算资源
  + 应用
  + 数据
  + 服务
+ 运作模式
  + laaS（基础设施即服务）：提供硬件相关的服务。比如虚拟化、服务器、存储和网络
  + PaaS（平台即服务）：提供平台或中间件的服务，比如操作系统和中间件
  + SaaS（软件及服务）：提供直接的应用服务，比如收发邮件，订购商品
+ 优势
  + 规模大：云计算的服务器是百万级别的
  + 高可靠性：云计算采用各种容灾措施
  + 灵活性：根据需求动态分配
  + 低成本：规模效应带来的低成本
+ 主流技术
  + openstack
  + k8s



## openstack概述

+ 概述：laaS软件

+ 关键特性：虚拟化和云计算

+ 核心组件

  + Identity——Keystone：服务认证及授权
  + Image Service——Glance：存储和检索虚拟机磁盘镜像
  + OBject Storage——Swift：存储和检索无结构数据对象
  + Dashboard——Horizon：人机交互界面
  + Compute——Nova：提供虚拟机服务，包括创建和热迁移
  + Block Storage——Cinder：提供块存储服务
  + Network——Neutron：提供网络连接服务，创建和管理虚拟网络

+ 基于MySQL和RabbitMQ

+ 常用命令：https://www.jianshu.com/p/a49b83ecbf10

+ 简要架构图

  ![openstack](D:\img\brain\openstack.png)

说明：关联最多的两个模块为：Horizon即与用户直接关联的UI层，负责调用其他模块提供的接口为用户提供相关的服务，而Keystone负责认证与服务授权，Openstack中的所有服务实习访问控制。而与Horizon最直接关联的为Nova，其负责虚拟机的整个生命周期管理，而拥有一个虚拟机还需要基于其运行的OS，所以我们需要到Glance上检索获取合适的OS镜像，而最终的镜像存储维护由Swift实现。有了拉取到的OS镜像需要调用Cinder的块服务部署配置OS，之后的可以基于Neutron来为创建的虚拟机提供网络连接配置。

+ 创建实例流程

  + stage1：user发出创建实例申请

  ```mermaid
  sequenceDiagram
  	User ->> keystone:获取认证信息
  	keystone ->> User:返回请求auth-token
  	User ->> nova-api:携带token发送boot instance请求
  	nova-api ->> keystone:验证token
  	keystone ->> nova-api:返回验证结果
  ```

  + stage2：创建物理主机，rpc简单表示为通过消息队列的调用

  ```mermaid
  sequenceDiagram
  	nova-api ->> MySQL:初始化新建虚拟机的数据库记录
  	nova-api ->> RabbitMQ:向scheduler请求是否具有创建虚机机的资源
  	RabbitMQ ->> nova-scheduler:侦听队列获取请求
  	nova-scheduler ->> MySQL:查询数据库中计算资源情况，更新物理主机信息
  	nova-scheduler ->> nova-compute:发出创建虚拟机请求(rpc)
  	nova-compute ->> nova-condictor:获取虚拟机信息(rpc)
  	nova-condictor ->> MySQL:查询信息
  	MySQL ->> nova-condictor:返回信息
  	nova-condictor ->> nova-compute:回送虚拟机信息(rpc)
  ```

  + stage3：配置虚拟机，VM为配置的虚拟化驱动

  ```mermaid
  sequenceDiagram
  	nova-compute ->> glance-api:请求镜像
  	glance-api ->> keystone:验证token
  	keystone ->> glance-api:返回结果
  	glance-api ->> nova-compute:返回镜像URL
  	nova-compute ->> neutron:请求虚拟机网络信息
  	neutron ->> keystone:验证token
  	keystone ->> neutron:返回结果
  	neutron ->> nova-compute:返回网络信息
  	nova-compute ->> cinder:请求虚拟机持久化存储信息
  	cinder ->> keystone:验证token
  	keystone ->> cinder:返回结果
  	cinder ->> nova-compute:返回持久化存储信息
  	nova-compute ->> VM:根据信息和资源通过驱动创建虚拟机
  ```

+ 术语
  + RPC(Remote Procedure Call)远程过程调用：一个节点请求另一个节点提供的服务
  + RESTful API(Representational State Transfer API)表述性状态传递应用程序接口：网络中client和server的一种交互的形式
  + CURD：数据库增删改查
+ links：

  + RPC：https://www.jianshu.com/p/7d6853140e13
  + REST：
    + https://www.jianshu.com/p/84568e364ee8
    + https://zhuanlan.zhihu.com/p/97978097



## 部署方式

### RDO by PackStack

+ 系统版本：CentOS7.8
+ 安装版本：train版
+ 官方文档：https://www.rdoproject.org/
+ 配置步骤——all in one by packstack

1. 配置host文件，ip 主机名 域名

2. 配置网络：使用桥接模式，配置虚拟机OS的网络IP地址为静态地址，网络地址，网关和DNS服务器均与宿主机中相符。

   `桥接模式`：https://www.cnblogs.com/haoabcd2010/p/8683656.html

   `Linux网络配置`：https://blog.csdn.net/weixin_41072205/article/details/90488464

3. 配置内存和存储：从硬盘存储中创建一个新分区，并分配8G以上内存

   `Linux分区配置`：https://www.cnblogs.com/kerrycode/p/4612925.html

4. 更换yum源为国内源，配置可查看对应的使用说明

5. **根据主机环境提前安装好依赖的包和运行所需的服务**

`分析 /var/log/ 下对应组件的日志定位错误，各个组件的自动化配置文件位于/etc/ 下`

6. 通过packstack命令执行安装

```sh
packstack --allinone --os-neutron-l2-agent=openvswitch --os-neutron-ml2-mechanism-drivers=openvswitch --os-neutron-ml2-tenant-network-types=vxlan --os-neutron-ml2-type-drivers=vxlan,flat --provision-demo=n --os-neutron-ovs-bridge-mappings=extnet:br-ex --os-neutron-ovs-bridge-interfaces=br-ex:ens33
```

+ 安装过程中问题汇总
  + pip包安装
    + yum install python-pip
    + pytz
    + pyyaml
  + 循环导入（循环依赖）问题
    + yum downgrade leatherman
  + https服务问题
    + yum install httpd
    + SELinux：setenforce 0
  + mysql问题
    + 数据库链接时用户认证错误
  + 创建实例时一直schedule——MQ等待错误
    + 重启nova相关服务
    + 所有openstack服务的命名格式为openstack-*
  + 创建实例错误
    + 在VM中修改虚拟机配置支持CPU虚拟化
    + 开启虚拟化
  + links：
    + 循环依赖：https://blog.csdn.net/weixin_43905458/article/details/104898789
    + mysql密码变更：https://blog.csdn.net/weixin_34562922/article/details/116633739?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.control&spm=1001.2101.3001.4242
    + nova密码配置：https://blog.csdn.net/Miracle1203/article/details/102730311
    + nova-compute启动管理等：https://www.cnblogs.com/goldsunshine/p/8675462.html
    + 开启主板虚拟化：

```sh
# 运行脚本，通过packstack安装前运行
yum install python-pip
pip install pytz pyyaml

yum downgrade leatherman

yum install httpd
setenforce 0   #建议永久关闭selinux
systemctl start httpd
systemctl enable httpd

yum install tuned
systemctl start tuned
systemctl enable tuned

#虚拟机开启CPU虚拟化
```



### Kolla

+ kolla：容器镜像构建
+ kolla-ansible：容器部署
+ kolla-kubernetes：容器部署与管理



其他安装部署方式：https://cloud.tencent.com/developer/article/1491077



### 附加服务

#### Barbican

+ 功能：管理加密密钥，为cinder加密卷提供密钥

+ 配置步骤

  + 数据库及openstack预配置

    1. 创建数据库

    2. 创建用户和角色

    3. 创建endpoint

    ```sh
    #数据库相关
    $ mysql -u root -p
    
    CREATE DATABASE barbican;
    
    GRANT ALL PRIVILEGES ON barbican.* TO 'barbican'@'localhost' \
      IDENTIFIED BY 'BARBICAN_DBPASS';
    GRANT ALL PRIVILEGES ON barbican.* TO 'barbican'@'%' \
      IDENTIFIED BY 'BARBICAN_DBPASS';
    
    exit;
    
    #创建用户和角色
    $ source admin-openrc
    $ openstack user create --domain default --password-prompt barbican   #barbican_pwd
    $ openstack role add --project services --user barbican admin
    $ openstack role create creator
    $ openstack role add --project services --user barbican creator
    $ openstack service create --name barbican --description "Key Manager" key-manager
    
    #创建endpoint
    $ openstack endpoint create --region RegionOne key-manager public http://controller:9311
    $ openstack endpoint create --region RegionOne key-manager internal http://controller:9311
    $ openstack endpoint create --region RegionOne key-manager admin http://controller:9311
    ```
    
    
    
  + 安装及配置Barbican

    1. 安装Barbican组件
    2. 编辑barbican.conf配置文件
    3. 执行数据库迁移

    4. 启动barbican-api服务

    ```sh
    #barbican.conf配置文件
    [DEFAULT]
    
    #API settings
    bind_host = 0.0.0.0
    bind_port = 9311
    host_href = http://controller:9311
    log_file = /var/log/barbican/api.log
    
    #Database connection
    sql_connection = mysql+pymysql://barbican:BARBICAN_DBPASS@controller/barbican
    
    #RabbitMQ connection
    transport_url = rabbit://guest:guest@controller:5672/
    
    [oslo_policy]
    policy_file = /etc/barbican/policy.json
    policy_default_rule = default
    
    [secretstore]
    namespace = barbican.secretstore.plugin
    enabled_secretstore_plugins = store_crypto
    
    [crypto]
    namespace = barbican.crypto.plugin
    enabled_crypto_plugins = simple_crypto
    
    [simple_crypto_plugin]
    kek = 'dGhpcnR5X3R3b19ieXRlX2tleWJsYWhibGFoYmxhaGg='
    
    # Keystone Authentication
    [keystone_authtoken]
    www_authenticate_uri = http://controller:5000/
    auth_url = http://controller:5000
    memcached_servers = controller:11211
    auth_type = password
    project_domain_name = Default
    user_domain_name = Default
    project_name = services
    username = barbican
    password = barbican_pwd
    ```

    

    ```sh
    #安装Barbican
    yum install openstack-barbican
    
    #数据库迁移
    su -s /bin/bash barbican -c "barbican-manage db upgrade"
    
    #启动并启用服务
    systemctl enable --now openstack-barbican-api
    ```

    

  + 测试验证

    ```sh
    . keystonerc_admin
    #Barbican服务创建测试密码
    openstack secret store --name mysecret --payload mysecretkey
    
    #确认secret已创建
    openstack secret list
    
    #印有secret的href可以在以后获取键值
    openstack secret get [secret herf] --payload
    ```



#### Volume Encryption

+ 功能：对存储卷提供加密服务

+ 配置步骤

  + 修改cinder.conf配置文件，之后重启cinder-api服务
  + 修改nova.conf配置文件，之后重启nova-compute服务

  ```sh
  [key_manager]
  api_class = castellan.key_manager.barbican_key_manager.BarbicanKeyManager
  ```

  + 创建加密卷类型

  ```sh
  openstack volume type create LUKS
  cinder encryption-type-create --cipher aes-xts-plain64 --key_size 256 \
    --control_location front-end LUKS nova.volume.encryptors.luks.LuksEncryptor
  ```

  + 测试验证

  ```sh
  #创建虚拟机实例
  openstack server create --image Cirror0.4.0 --flavor m1.tiny TESTVM
  
  #创建普通卷和未加密卷并挂载
  openstack volume create --size 1 'unencrypted volume'
  openstack volume create --size 1 --type LUKS 'encrypted volume'
  openstack volume list
  openstack server add volume --device /dev/vdb TESTVM 'unencrypted volume'
  openstack server add volume --device /dev/vdc TESTVM 'encrypted volume'
  openstack volume list
  
  #在VM中在普通卷和加密卷中存储信息
  echo "Hello, world (unencrypted /dev/vdb)" >> /dev/vdb
  echo "Hello, world (encrypted /dev/vdc)" >> /dev/vdc
  sync && sleep 2
  sync && sleep 2
  
  #刷新测试字符串写入情况
  sync && sleep 2
  sync && sleep 2
  strings /dev/stack-volumes/volume-* | grep "Hello"
  #result:Hello, world (unencrypted /dev/vdb)
  ```
  
  



## 运行使用

### 网络配置

+ 桥接器br-ex——连接外网

```shell
ovs-vsctl show
```

+ 配置过程

1. 创建外部网络，并划分子网和创建IP地址池

```sh
neutron net-create external_network --provider:network_type flat --provider:physical_network extnet  --router:external
neutron subnet-create --name public_subnet --enable_dhcp=False --allocation-pool=start=192.168.1.90,end=192.168.1.100 --gateway=192.168.1.1 external_network 192.168.1.0/24
```

2. 创建内部网络
3. 创建路由器，并配置接口连接保证内外网互通



### 磁盘管理

+ 由cinder从cinder-volumes划分
+ 步骤

1. 新建卷然后关联到Instance上扩展硬盘
2. 重启虚拟机
3. 创建分区
4. ...



### 镜像配置

+ 获取云端镜像
  + 官方Url：https://download.cirros-cloud.net/
+ 创建镜像



### 用户认证及管理

+ Browser登录：浏览器中访问本机ip地址即可跳转到dashboard界面

+ 命令行登录

```sh
#默认的登录配置文件路径为/root/keystonerc_admin
source /root/keystonerc_admin
```

+ 创建用户

1. 新建一个Project，并限制物理资源配置
2. 新建一个User，并分配到该Project，设置Role(权限的抽象表述)

+ 创建公私钥对
  + 私钥文件路径：/tmp/mozilla_root0
+ 创建安全组——配置类似防火墙



### 实例创建

+ 创建主机类型Flavor
+ 创建实例Instance
  + 创建时配置内容
    + 镜像
    + 主机类型
    + 网络
    + 安全策略
    + SSH Key
  + 分配外网IP
  + Console链接或SSH链接



## 最终配置功能

+ 描述：最终在一台CentOS服务器上配置好了Openstack(Train版)的所有基本服务，包括(Keystone、Swift、Horizon、Nova、Cinder、Neutron)，并额外配置了Barbican服务，基于此配置了卷加密功能

+ 部分展示

  + 所有endpoint

  ![image-20210823103719991](C:\Users\Jison\AppData\Roaming\Typora\typora-user-images\image-20210823103719991.png)

  + 实例创建与连接

  ![image-20210823105449667](C:\Users\Jison\AppData\Roaming\Typora\typora-user-images\image-20210823105449667.png)

  

  ![image-20210823105721696](C:\Users\Jison\AppData\Roaming\Typora\typora-user-images\image-20210823105721696.png)

  + 密钥管理

    ![image-20210823105948074](C:\Users\Jison\AppData\Roaming\Typora\typora-user-images\image-20210823105948074.png)

  + 加密卷

    ![image-20210823110218700](C:\Users\Jison\AppData\Roaming\Typora\typora-user-images\image-20210823110218700.png)

  

# 拓展延伸

+ 描述：云计算的另外一种主流技术

+ Docker

  + 概述

    + Docker 是一个开源的应用容器引擎，基于Go 语言并遵从 Apache2.0 协议开源。
    + Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。
    + 容器是完全使用沙箱机制。
    + 容器性能开销极低。

  + 关键机制

    + NameSpace——命名空间
    + CGroup——资源限制

  + 常用命令

    + docker相关

    ```sh
    systemctl status docker
    docker info
    systemctl start docker
    systemctl stop docker
    ```

    + 查看镜像

    ```sh
    docker images
    ```

    + 查看容器

    ```sh
    docker ps -a
    docker stats
    ```

    + 运行容器

    ```sh
    docker run <image_name>
    ```

    `-i`:交互式运行，让容器的标准输入打开

    `-t`:让docker分配一个伪终端并绑定到容器的标准输入上

    `-d`:后台运行

    `--name`:指定容器名

    `-v`:映射目录

    `-p`:映射端口

    + 进入容器

    ```shell
    docker exec <container_name>
    ```

    + 查看IP地址

    ```sh
    cat /etc/hosts
    ```

    + 退出容器

    ```sh
    exit
    ```

+ k8s

  + 概述：容器编排管理工具，基于Go语言Google开源
  + 基本概念
    + pod：调度的最小单位——docker+pause
    + deployment：维持pod数量
    + service：将多个pod抽象为同一个服务，并负责负载均衡
    + ingress：解析服务，将公网IP映射为内网虚拟IP
  + 机制
    + master：控制管理节点
      + apiserver：提供面向用户和node的接口
      + ETCD：维护各节点信息的数据库
      + controller-manager：协调调度，资源自动化控制中心
      + scheduler：调度接口的实施
    + node：普通节点
      + kubelet：与apiserver通信，调度容器
      + kubeproxy：创建虚拟网卡
      + docker：容器
  + 特性
    + 适用于业务变化快的应用场景
    + 启动快速，消耗资源少
    + 应用云化与微服务架构

+ k8s与openstack对比

  + openstack是IaaS，而k8s偏向于PaaS
  + openstack组件依赖复杂，k8s组件较少且得益于容器耦合较低
  + openstack隔离性优于k8s
  + ...
