---
title: vsftpd 安装
date: 2018-12-16 23:08:56
tags: vsftpd
category:
- linux
- 运维
---

**1.** **安装**

**执行 yum -y install vsftpd**

注：

(1)是否使用sudo权限执行请根据您具体环境来决定

(2)rpm -qa | grep vsftpd 可通过这个检查是否已经安装vsftpd

(3)默认配置文件在/etc/vsftpd/vsftpd.conf

**2.** **创建虚拟用户**

(1) 选择在根或者用户目录下创建ftp文件夹：mkdir ftpfile, 如：/ftpfile

(2) 添加匿名用户：useradd ftpuser -d /ftpfile -s /sbin/nologin

(3) 修改ftpfile权限： chown -R ftpuser.ftpuser /ftpfile

(4) 重设ftpuser密码：passwd ftpuser (例如：123456)

**3.** **配置**

(1) cd /etc/vsftpd

(2) sudo vim chroot_list

(3) 把刚才新增的虚拟用户添加到此配置文件中

(4) :wq 保存退出

(5) sudo vim /etc/selinux/config , 修改为 SELINUX=disabled

(6) :wq 保存退出

注：如果一会验证的时候碰到550拒绝访问请执行：

sudo setsebool -P ftp_home_dir 1

然后重启linux服务器，执行reboot命令

(7) sudo vim /etc/vsftpd/vsftpd.conf

请查看 vsftpd 配置文件详解


**4.** **防火墙配置**

(1) sudo vim /etc/sysconfig/iptables

(2) 
-A INPUT -p TCP --dport 61001:62000 -j ACCEPT

-A OUTPUT -p TCP --sport 61001:62000 -j ACCEPT

-A INPUT -p TCP --dport 20 -j ACCEPT

-A OUTPUT -p TCP --sport 20 -j ACCEPT

-A INPUT -p TCP --dport 21 -j ACCEPT

-A OUTPUT -p TCP --sport 21 -j ACCEPT

将以上配置添加到防火墙配置中

(3) :wq 保存退出

(4) sudo service iptables restart 执行命令重启防火墙

**5.** **vsftpd验证**

(1）执行 sudo service vsftpd restart

注：第一次启动Shutting down vsftpd 是 faild 不用理会

因为这是重启命令，保证Starting vsftpd是 OK 即代表 vsftpd 服务成功








