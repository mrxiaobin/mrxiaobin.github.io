---
layout: post
title: RHCSA练习笔记
date: 2017-1-11
categories: blog
tags: [学习笔记,Linux]
description: 
---

### 1. 分区

创建一个500M的分区，格式化为xfs，挂载到/data，要求每次开机均生效。

```shell
[root@server0 ~] fdisk -l  #查看系统中的硬盘分区情况
[root@server0 ~] fdisk /dev/vdb 
...fdisk的交互操作
[root@server0 ~] mkfs.xfs /dev/vdb1 
[root@server0 ~] blkid
/dev/vda1: UUID="9bf6b9f7-92ad-441b-848e-0257cbb883d1" TYPE="xfs" 
/dev/vdb1: UUID="0dc01c9c-268f-4684-949a-8d7fe4064cfc" TYPE="xfs" 
[root@server0 ~] vim /etc/fstab 
#fstab中添加： UUID=0dc01c9c-268f-4684-949a-8d7fe4064cfc /data	xfs	defaults	0 0
[root@server0 ~] mkdir /data
[root@server0 ~] mount -a
[root@server0 ~] df -Th #检查是否挂载成功
```

### 2. lvm

创建由50个PE组成的LV，PE大小为8MB，格式化为ext4，挂载至/mnt/data，要求每次开机均生效。

```shell
[root@server0 ~]# fdisk /dev/vdb
#...fdisk的交互操作,注意此处注意要更改分区的类型为8e(Linux LVM)
[root@server0 ~] pvcreate  /dev/vdb2 #此处可能会报错说分区不存在，执行以下命令后再进行操作即可
[root@server0 ~] partprobe /dev/vdb 
[root@server0 ~] pvcreate  /dev/vdb2
  Physical volume "/dev/vdb2" successfully created
[root@server0 ~] vgcreate -s 8M vg0 /dev/vdb2
  Volume group "vg0" successfully created
[root@server0 ~] lvcreate -n data -l 50 vg0
  Logical volume "data" created
[root@server0 ~] lvdisplay #查看lv是否满足要求
[root@server0 ~] mkfs.ext4 /dev/vg0/data 
[root@server0 ~] blkid
[root@server0 ~] vim /etc/fstab
#fstab中添加：UUID=6b6dc548-41df-4fb6-9a81-0fbd889d09f0 /mnt/data	ext4	defaults	0 0
[root@server0 ~] mkdir /mnt/data
[root@server0 ~] mount -a
[root@server0 ~] df -Th #检查是否挂载成功
```

### 3. lvm拉伸

将2中的文件系统拉伸至800M，不要影响其中的数据

```shell
[root@server0 ~] vgdisplay #先查看卷组中是否还有足够的空间，如果有足够的空间直接扩容lv
[root@server0 ~] vgextend vg0 /dev/vdb3 #如果卷组没有足够空间，要先扩容卷组，再进行下边操作
[root@server0 ~] lvextend -L 800M /dev/vg0/data 
[root@server0 ~] resize2fs /dev/vg0/data 
[root@server0 ~] df -Th
```

### 4. 扩展swap，增加256M，要求每次开机均生效

扩展swap可以使用分区，也可使用文件

- 使用文件

```shell
[root@server0 ~] dd if=/dev/zero of=/root/swapfile bs=1M count=256
256+0 records in
256+0 records out
268435456 bytes (268 MB) copied, 1.48084 s, 181 MB/s
[root@server0 ~] mkswap /root/swapfile 
Setting up swapspace version 1, size = 262140 KiB
no label, UUID=2ddec08c-c448-4bcf-bae1-82521d4c957f
[root@server0 ~] vim /etc/fstab 
#此处只能写文件的绝对路径，不能写UUID: /root/swapfile	swap	swap	defaults	0 0
[root@server0 ~] swapon
```

- 使用分区

用分区的时候，在分区时记得改变标签，默认的83改为82，再mkswap，可以看到UUID，即可用UUID在fstab中挂载

### 5. 计划任务

创建一个计划任务，要求在每月的10-15号每天9：00-17：00每隔10分钟，执行/bin/echo hello，仅允许root和student用户能创建计划任务

```shell
[root@server0 ~] crontab -e
#添加内容： */10 9-17 10-15 * * /bin/echo hello
#每一列分别表示： 分 时 日 月 周 命令
[root@server0 ~] vim /etc/cron.allow
#在里边添加两行，分别为root和student
[root@server0 ~] systemctl restart crond.service 
[root@server0 ~] systemctl enable crond.service
```

### 6. SELinux

配置SELinux，确保处于Enforcing模式，并每次开机均生效

系统默认就这个模式，只需确认下配置文件/etc/selinux/config中的SELINUX=enforcing

### 7. 将/etc目录打包并压缩至/data/etc.tgz

```shell
[root@server0 ~] tar -cvzf /data/etc.tgz /etc/
```

### 8. 时间同步

将本机时间使用NTP，NTP Server：classroom.example.com

```shell
[root@server0 ~] vim /etc/chrony.conf 
#文件中注释掉默认的服务器，添加： server classroom.example iburst
[root@server0 ~] systemctl restart chronyd.service 
[root@server0 ~] systemctl enable chronyd.service
```

### 9. yum源

配置yum仓库，安装源http://content.example.com/rhel7.0/x86_64/dvd

```shell
[root@server0 ~] vim /etc/yum.repos.d/rhel_dvd.repo 
[rhel_dvd]
name = dvd
baseurl = http://content.example.com/rhel7.0/x86_64/dvd
gpgcheck = 0
enabled = 1
```

### 10. 创建it组，指定gid=1200

```shell
[root@server0 ~] groupadd -g 1200 it
```

### 11. 创建用户

创建用户user1，user2，user3，密码均为redhat，将以上三个用户加入it组，指定user1的UID=1200，user2不允许交互式登录，user3用户的密码在30天后过期。

```shell
[root@server0 ~] useradd -u 1200 user1 
[root@server0 ~] useradd -s /sbin/nologin user2
[root@server0 ~] useradd user3 
[root@server0 ~] chage -M 30 user3
[root@server0 ~] usermod -G it user1
[root@server0 ~] usermod -G it user2
[root@server0 ~] usermod -G it user3
[root@server0 ~] passwd user1 #依次设置三个密码
...
```

### 12. sgid

设置/data目录的拥有组为it，要求任何人在该目录中创建的文件拥有组自动属于it组

```shell
[root@server0 ~] chgrp it /data/
[root@server0 ~] chmod g+s /data/
```

### 13. acls权限

不要改变/mnt/data目录的拥有人和拥有组，要求user1对该目录有完整的控制权限，user3对该目录无任何权限

```shell
[root@server0 ~] setfacl -m u:user1:rwx /data/
[root@server0 ~] setfacl -m u:user3:--- /data/
[root@server0 ~] getfacl /data/
```

### 14. 破解root密码，修改为redhat2015

在grub菜单界面按e，编辑启动参数，在linux16的最后加上rd.break，根据提示保存并启动进入单用户模式

![破解密码](http://7xt805.com1.z0.glb.clouddn.com/mima.png)

注意，修改完密码不要忘了最后的 touch /.autorelabel，否则会因为SELinux的原因开不了机

### 15. yum升级

升级内核至3.10.0-123.1.2.e17.x86_64，新内核在http://content.example.com/rhel7.0/x86_64/errata，确保新内核作为默认启动项

```shell
#先添加yum源
[root@server0 ~] vim /etc/yum.repos.d/errata.repo 
[errata]
name = errata
baseurl = http://content.example.com/rhel7.0/x86_64/errata
gpgcheck = 0
enabled = 1
[root@server0 ~] 
[root@server0 ~] yum update kernel
```

### 16. grep命令、管道

下载http://classroom.example.com/pub/vsftpd.conf至/root目录，过滤出该文件中的所有非注释行和非空行，并且以YES结尾的内容，导出至/root/vsftpd.bak中，不要改变文件的顺序

```shell
[root@server0 ~] wget http://classroom.example.com/pub/vsftpd.conf
[root@server0 ~] grep -v ^# vsftpd.conf | grep YES$ > vsftpd.bak
```

### 17. find命令

查找/home目录下拥有人和拥有组均为user1的文件备份至/root/backups目录下，并保留权限

```shell
[root@server0 ~] find /home/ -user user1 -group user1 -exec cp -a {} /root/backups \;
```

### 18. ldap客户端

设置本机使用外部的认证，要求如下

1. 将本机加入到LDAP认证中，并使用TLS加密，LDAP信息如下:
- LDAP Server：classroom.example.com
- BASE DN：dc=classroom，dc=com
- TLS密钥可以从http://classroom.example.com/pub/example-ca.crt下载
2. 当使用LDAP认证以后，将可以使用ldapuserX登录，X表示你的desktop number，当使用该用户登录时没有该用户的宿主目录，除非配置后续的autofs

```shell
[root@server0 ~] yum -y install sssd
[root@server0 ~] mkdir /etc/openldap/cacerts && cd /etc/openldap/cacerts
[root@server0 cacerts] wget http://classroom.example.com/pub/example-ca.crt
[root@server0 cacerts] authconfig-tui #文本图形界面配置
```
![authconfig-tui](http://7xt805.com1.z0.glb.clouddn.com/ldap.png)
![authconfig-tui](http://7xt805.com1.z0.glb.clouddn.com/ldap1.png)

配置完成后ssd服务会启动并默认会开机启动，接着可以切换用户进行测试

```shell
[root@server0 ~] su - student
Last login: Wed Jan 11 16:24:23 CST 2017 from 172.25.0.250 on pts/0
[student@server0 ~]$ su - ldapuser1
Password: 
Last login: Wed Jan 11 17:50:42 CST 2017 on pts/0
su: warning: cannot change directory to /home/guests/ldapuser1: No such file or directory
mkdir: cannot create directory '/home/guests': Permission denied
-bash-4.2$ 

```

### 19. autofs

配置autofs，当用户登录时自动挂载宿主目录，目录已经在classroom.example.com上通过NFS共享

```shell
[root@server0 ~] yum -y install autofs
[root@server0 ~] vim /etc/auto.master
#文件中添加一行： /home/guests    /etc/auto.ldap
[root@server0 ~] cp -p /etc/auto.misc /etc/auto.ldap
[root@server0 ~] vim /etc/auto.ldap 
[root@server0 ~] systemctl restart autofs
[root@server0 ~] systemctl enable autofs
login: Wed Jan 11 17:52:24 CST 2017 on pts/0
[student@server0 ~]$ 
[student@server0 ~]$ su - ldapuser1
Password: 
Last login: Wed Jan 11 17:59:00 CST 2017 on pts/0
[ldapuser1@server0 ~]$ pwd
/home/guests/ldapuser1
[ldapuser1@server0 ~]$ 
```
