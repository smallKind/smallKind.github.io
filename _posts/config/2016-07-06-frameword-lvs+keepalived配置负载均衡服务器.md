---
layout: post
title: LVS+KeepAlived配置负载均衡服务器
date: 2016-07-06 22:00:01 +8000
category: 配置
tags: LVS
---

* content
{:toc}


### 原理

#### 什么是LVS?

* [LVS项目介绍](http://www.linuxvirtualserver.org/zh/lvs1.html)
* [LVS集群的体系结构](http://www.linuxvirtualserver.org/zh/lvs2.html)
* [LVS集群中的IP负载均衡技术](http://www.linuxvirtualserver.org/zh/lvs3.html)
* [LVS集群的负载调度](http://www.linuxvirtualserver.org/zh/lvs4.html)

#### 什么是KeepAlived?

>KeepAlived观其名可知，保持存活，在网络里面就是保持在线了， 也就是所谓的高可用或热备，用来防止单点故障(单点故障是指一旦某一点出现故障就会导致整个系统架构的不可用)的发生，那说到KeepAlived时不得 不说的一个协议就是VRRP协议，可以说这个协议就是KeepAlived实现的基础

keepalived也是模块化设计，不同模块复杂不同的功能，下面是keepalived的组件：
core check vrrp libipvs-2.4 libipvs-2.6

* core：是keepalived的核心，复杂主进程的启动和维护，全局配置文件的加载解析等
* check：负责healthchecker(健康检查)，包括了各种健康检查方式，以及对应的配置的解析包括LVS的配置解析
* vrrp：VRRPD子进程，VRRPD子进程就是来实现VRRP协议的
* libipvs*：配置LVS会用到

注意，keepalived和LVS完全是两码事，只不过他们各负其责相互配合而已

![](/img/framework/003919424.png)

keepalived启动后会有三个进程

* 父进程：内存管理，子进程管理等等
* 子进程：VRRP子进程
* 子进程：healthchecker子进程

由图可知，两个子进程都被系统WatchDog看管，两个子进程各自复杂自己的事，healthchecker子进程复杂检查各自服务器的健康程度，例如HTTP，LVS等等，如果healthchecker子进程检查到MASTER上服务不可用了，就会通知本机上的兄弟VRRP子进程，让他删除通告，并且去掉虚拟IP，转换为BACKUP状态

#### KeepAlived配置文件详解

    global_defs {
       notification_email {#指定keepalived在发生切换时需要发送email到的对象，一行一个
		    monitor@3evip.cn
       }
       notification_email_from monitor@3evip.cn #指定发件人
       smtp_server stmp.3evip.cn #指定smtp服务器地址
       smtp_connect_timeout 30 #指定smtp连接超时时间
       router_id LVS_DEVEL #运行keepalived机器的一个标识
    }

    vrrp_sync_group VG_1{ #监控多个网段的实例
	    group {
		    inside_network #实例名
		    outside_network
	    }
	    notify_master /path/xx.sh #指定当切换到master时，执行的脚本
	    netify_backup /path/xx.sh #指定当切换到backup时，执行的脚本
	    notify_fault "path/xx.sh VG_1" #故障时执行的脚本
	    notify /path/xx.sh
	    smtp_alert #使用global_defs中提供的邮件地址和smtp服务器发送邮件通知
    }

    vrrp_instance inside_network {
        state BACKUP #指定那个为master，那个为backup，
        如果设置了nopreempt这个值不起作用，主备考priority决定
        interface eth0 #设置实例绑定的网卡
        dont_track_primary #忽略vrrp的interface错误（默认不设置）
        track_interface{ #设置额外的监控，里面那个网卡出现问题都会切换
		    eth0
		    eth1
        }
        mcast_src_ip #发送多播包的地址，如果不设置默认使用绑定网卡的primary ip
        garp_master_delay #在切换到master状态后，延迟进行gratuitous ARP请求
        virtual_router_id 50 #VPID标记
        priority 99 #优先级，高优先级竞选为master
        advert_int 1 #检查间隔，默认1秒
        nopreempt #设置为不抢占 注：这个配置只能设置在backup主机上，
        而且这个主机优先级要比另外一台高
        preempt_delay #抢占延时，默认5分钟
        debug #debug级别
        authentication { #设置认证
            auth_type PASS #认证方式
            auth_pass 111111 #认证密码
        }
        virtual_ipaddress { #设置vip
            192.168.36.200
        }
     }
     virtual_server 192.168.36.99 80 {
        delay_loop 6 #健康检查时间间隔
        lb_algo rr  #lvs调度算法rr|wrr|lc|wlc|lblc|sh|dh
        lb_kind DR  #负载均衡转发规则NAT|DR|RUN
        persistence_timeout 5 #会话保持时间
        protocol TCP #使用的协议
        persistence_granularity <NETMASK> #lvs会话保持粒度
        virtualhost <string> #检查的web服务器的虚拟主机（host：头）
        sorry_server<IPADDR> <port> #备用机，所有realserver失效后启用
	    real_server 192.168.200.5 23 {
                weight 1 #默认为1,0为失效
                inhibit_on_failure #在服务器健康检查失效时，将其设为0，
                而不是直接从ipvs中删除
                notify_up <string> | <quoted-string>
                #在检测到server up后执行脚本
                notify_down <string> | <quoted-string>
                #在检测到server down后执行脚本
			    TCP_CHECK {
				    connect_timeout 3 #连接超时时间
				    nb_get_retry 3 #重连次数
				    delay_before_retry 3 #重连间隔时间
				    connect_port 23  健康检查的端口的端口
				    bindto <ip>
			    }
			    HTTP_GET | SSL_GET{
				    url{ #检查url，可以指定多个
					    	path /
						    digest <string> #检查后的摘要信息
						    status_code 200 #检查的返回状态码
				    }
				    connect_port <port>
				    bindto <IPADD>
				    connect_timeout 5
				    nb_get_retry 3
				    delay_before_retry 2
			    }

			    SMTP_CHECK{
				    host{
					    	connect_ip <IP ADDRESS>
						    connect_port <port> #默认检查25端口
						    bindto <IP ADDRESS>
					    }
					connect_timeout 5
					retry 3
					delay_before_retry 2
					helo_name <string> | <quoted-string>
					#smtp helo请求命令参数，可选
			    }
			    MISC_CHECK{
					misc_path <string> | <quoted-string>#外部脚本路径
					misc_timeout #脚本执行超时时间
					misc_dynamic #如设置该项，
					则退出状态码会用来动态调整服务器的权重，
					返回0 正常，不修改；返回1，检查失败，权重改为0；
					返回2-255，正常，
					权重设置为：返回状态码-2
			}
        }
    }

### 配置

四台服务器，负载均衡服务器两台（搭建LVS+KeepAlived环境），应用服务器两台（搭建JDK8+Tomcat8环境）。所有服务器均为CentOS-7,KeepAlived版本为1.2.21。

IP地址:

* VIP(192.168.1.200)
* LVS_Master(192.168.1.143)
* LVS_Slave(192.168.1.145)
* WEB_1(192.168.1.144)
* WEB_2(192.168.1.125)

#### 1.关闭防火墙

CentOS-7采用的防火墙为firewall，为了测试直接使用端口,所有服务器关闭防火墙

    systemctl stop firewalld

#### 2.负载均衡服务器配置

##### 2.1.安装IPVSADM

IPVSADM理解为IPVS管理工具；LVS（Linux Virtual Server）的核心为[IPVS](http://baike.baidu.com/view/2428775.htm)（IP Virtual Server），从Linux内核版本2.6起，IPVS模块已经编译进了Linux内核

使用yum命令安装，系统会自动选择合适的版本

    yum -y install ipvsadm


#### 2.2.KeepAlived安装

安装到/usr/src目录下

    cd /usr/src

安装编译KeepAlived源文件需要工具，gcc,make系统安装

    yum -y install openssl-devel kernel-devel

下载KeepAlived

    wget http://www.keepalived.org/software/keepalived-1.2.21.tar.gz
    tar -xvf keepalived-1.2.21.tar.gz

编译KeepAlived

    cd keepalived-1.2.21
    ./configure
    make &&  make install

KeepAlived配置成系统服务

    cp /usr/local/etc/rc.d/init.d/keepalived /etc/rc.d/init.d/
    cp /usr/local/etc/sysconfig/keepalived /etc/sysconfig/
    mkdir /etc/keepalived
    cp /usr/local/etc/keepalived/keepalived.conf /etc/keepalived/
    cp /usr/local/sbin/keepalived /usr/sbin/

##### 2.3.打开IP Forward功能（LVS三种负载均衡技术都需要打开此功能）

    vi /etc/sysctl.conf

打开修改“net.ipv4.ip_forward = 1”。使设置立即生效：

    sysctl -p

##### 2.4.创建一个虚拟IP（VIP）

    ifconfig eth0:1 192.168.1.200 netmask 255.255.255.0 up

##### 2.5.配置KeepAlived配置文件

LVS_Master配置文件如下：

    global_defs {
         notification_email {
         acassen@firewall.loc
         failover@firewall.loc
         sysadmin@firewall.loc
    }
       notification_email_from Alexandre.Cassen@firewall.loc
       smtp_server 192.168.200.1
       smtp_connect_timeout 30
       router_id LVS_MASTER
       vrrp_skip_check_adv_addr
       vrrp_strict
       vrrp_garp_interval 0
       vrrp_gna_interval 0
    }

    vrrp_instance VI_1 {
        state MASTER
        interface eth0
        virtual_router_id 51
        priority 100
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 1111
        }
        virtual_ipaddress {
            192.168.1.200
        }
    }

    virtual_server 192.168.1.200 8080 {
        delay_loop 6
        lb_algo rr
        lb_kind DR
        protocol TCP

        sorry_server 192.168.200.200 1358

        real_server 192.168.1.125 8080 {
            weight 1
            TCP_CHECK {
                connect_timeout 3
                nb_get_retry 3
                delay_before_retry 3
            }
        }

        real_server 192.168.1.144 8080 {
            weight 1
            TCP_CHECK {
                connect_timeout 3
                nb_get_retry 3
                delay_before_retry 3
            }
        }
    }

LVS_Slave配置，MASTER与BACKUP配置仅三处不同：global_defs中的router_id、vrrp_instance中的state、priority。

    global_defs {
         notification_email {
         acassen@firewall.loc
         failover@firewall.loc
         sysadmin@firewall.loc
    }
        notification_email_from Alexandre.Cassen@firewall.loc
        smtp_server 192.168.200.1
        smtp_connect_timeout 30
        router_id LVS_BACKUP
        vrrp_skip_check_adv_addr
        vrrp_strict
        vrrp_garp_interval 0
        vrrp_gna_interval 0
    }

    vrrp_instance VI_1 {
        state BACKUP
        interface eth0
        virtual_router_id 51
        priority 50
        advert_int 1
        authentication {
            auth_type PASS
            auth_pass 1111
        }
        virtual_ipaddress {
            192.168.1.200
        }
    }

    virtual_server 192.168.1.200 8080 {
        delay_loop 6
        lb_algo rr
        lb_kind DR
        protocol TCP

        real_server 192.168.1.125 8080 {
            weight 1
            TCP_CHECK {
                connect_timeout 3
                nb_get_retry 3
                delay_before_retry 3
            }
        }

        real_server 192.168.1.144 8080 {
            weight 1
            TCP_CHECK {
                connect_timeout 3
                nb_get_retry 3
                delay_before_retry 3
            }
        }

    }

##### 2.6.启动KeepAlived

    chkconfig keepalived on
    service keepalived start

查看进程：

    ps aux | grep keepalived

Keepalived正常运行时，共启动3个进程，其中一个进程是父进程，负责监控其子进程；一个是vrrp子进程；另外一个是checkers子进程。

![](/img/framework/1042A3D3-3D58-44AD-ABC7-0E6A42F54206.png)

##### 2.7.查看虚拟IP是否加上

    ip a

![](/img/framework/8043ADE6-A08A-457E-9487-721F09E9BD0C.png)

如图说明VIP已经自动配置上

#### 3.配置WEB服务器

##### 3.1.配置配置虚拟IP启动脚本

    vim /etc/init.d/realserver.sh

在脚本中添加：

    #!/bin/bash
    SNS_VIP=192.168.1.200
    . /etc/rc.d/init.d/functions
    case "$1" in
    start)
    ifconfig lo:0 $SNS_VIP netmask 255.255.255.255 broadcast $SNS_VIP
    /sbin/route add -host $SNS_VIP dev lo:0
    echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
    echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
    echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
    echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
    sysctl -p >/dev/null 2>&1
    echo "RealServer Start OK"
    ;;
    stop)
    ifconfig lo:0 down
    route del $SNS_VIP >/dev/null 2>&1
    echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
    echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
    echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
    echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
    echo "RealServer Stoped"
    ;;
    *)
    echo "Usage: $0 {start|stop}"
    exit 1
    esac
    exit 0

启动Tomcat，为了测试负载均衡，辨识是哪个服务器，可以修改index.jsp。

##### 3.2.启动虚拟IP脚本

    service realserver start

##### 3.3.查看本机ip

    ifconfig

可以看到网络有个虚拟IP

![](/img/framework/2294B3FC-5098-4CD8-A3C4-B80BA65671D5.png)

去LVS_Master的终端查看可以看到，已经连接上了WEB_1服务器，运行如下命令

    ipvsadm -ln

![](/img/framework/1FDF3ADC-BB77-493C-A5E7-5576E2522A7E.png)

其他WEB服务器也一样步骤配置。

#### 4.常用命令

显示集群中服务器ip信息：ipvsadm -ln

查看日志：tail -f /var/log/messages

查看请求转发情况：ipvsadm -lcn | grep 虚拟IP


参考

1. [手把手教程： CentOS 6.5  LVS + KeepAlived  搭建 负载均衡 高可用 集群](http://blog.csdn.net/tengyuantuohai/article/details/19639671)
2. [keepalived配置文件详解](http://blog.csdn.net/zhu_tianwei/article/details/43603135)

