---
layout: post
title: RHE练习笔记
date: 2017-1-11
categories: blog
tags: [学习笔记,Linux]
description: 
---

### 0. 实验环境

- 主机foundation0 172
- 教室环境classroom(提供DNS/DHCP等各种服务) classroom.example.com 172.25.254.254
- 虚拟机server0: server0.example.com 172.25.0.11
- 虚拟机desktop0:classroom.example.com 172.25.0.10

操作均在两台虚拟机上进行

### 1. 静态IP配置

按照题目要求配置静态的ipv4地址

server上：

```shell
[root@server0 ~] nmcli connection modify "System eth0" ipv4.method manual ipv4.addresses "172.25.0.11/24 172.25.0.254" ipv4.dns 172.25.254.254
[root@server0 ~] nmcli connection up "System eth0"
```

desktop上按照相同方法配置，然后互相ping一下，测试是否正确

### 2. 配置静态的ipv6地址

按照题目给的参数配置静态的ipv6地址

```shell
[root@server0 ~]nmcli connection modify "System eth0" ipv6.method manual ipv6.addresses "2003:ac18::6/64"
```

在desktop上也配置好ipv6地址，然后用ping6命令测试是否能ping通地址

### 3. 双网卡配置

在server上按照要求配置一个链接，使用虚拟机管理器为Server虚拟机添加两块网卡，类型为virtio，桥接模式

- 此链接使用接口eth1和eth2
- 此链路在一个接口失效时仍能工作
- 此链路在server上使用地址172.25.0.20/24
- 此链路在系统重启后依然保持正常状态

```shell
[root@server0 ~]nmcli connection add type team con-name team0 ifname team0 config '{"runner":{"name":"activebackup"}}'
[root@server0 ~]nmcli connection modify team0 ipv4.method manual ipv4.addresses "172.25.0.20/24"
[root@server0 ~]nmcli connection add type team-slave con-name eth1 ifname eth1 master team0 
[root@server0 ~]nmcli connection add type team-slave con-name eth2 ifname eth2 master team0 
[root@server0 ~]nmcli connection up eth1 
[root@server0 ~]nmcli connection up eth2
[root@server0 ~]nmcli connection up team0 
[root@server0 ~]ifconfig eth0
```

配置成功后可在主机上ping team0的地址，然后将其中一块网卡断开连接，看看是否还能继续ping通

### 4. 防火墙

1. 在server上配置端口转发，要求在172.25.0/24的网络系统中，访问本地端口5423将转发到80端口，此配置永久有效
2. 仅允许example.com中的计算机访问本机SSH

用firewall-config图形配置工具配置即可，注意要选择permanent

### 5. iscsi服务端

在server上配置iscsi服务，磁盘名为iqn.2017-10.com.example.server0，并符合以下要求

1. 端口服务为3260
2. 使用iscsi_store作为后端卷，大小为4G
3. 此服务只能被desktop0.example.com访问

首先，在系统中分一块4G大小的分区出来，注意分区格式要更改为fe(LANstep)

```shell
yum -y install targetcli #安装交互式配置工具
```

参数如图所示
![targetcli](http://7xt805.com1.z0.glb.clouddn.com/targetcli.PNG?imageView2/2/w/600/interlace/0/q/100)
![targetcli](http://7xt805.com1.z0.glb.clouddn.com/targetcli1.PNG?imageView2/2/w/600/interlace/0/q/100)

```shell
[root@server0 ~] systemctl enable target.service 
[root@server0 ~] systemctl restart target
[root@server0 ~] firewall-cmd --permanent --add-port=3260/tcp
success
```

### 6. iscsi客户端

在desktop上配置使其能连接到server上提供的iqn.2017-10.com.example.server0，并符合以下要求：

1. iscsi设备在系统启动时自动加载
2. 在发现的iscsi设备上创建一个3.5G的分区，格式化为ext4，挂载至/mnt/storage上要求每次开机均生效

```shell
[root@desktop0 ~]vim /etc/iscsi/initiatorname.iscsi #修改iqn
#修改成在server中设置的acl：InitiatorName=iqn.2017-10.com.example.desktop0

[root@desktop0 ~]# systemctl restart iscsi
[root@desktop0 ~]iscsiadm -m discovery -t st -p 172.25.0.11
172.25.0.11:3260,1 iqn.2017-10.com.example.server0
[root@desktop0 ~]iscsiadm -m node -l
此时，用fdisk -l命令查看，可以看到系统多出来了一块硬盘sda，接下来就可以对该硬盘按照正常的方式进行分区、格式化、挂载了
```

### 7. nfs服务配置

在server上配置NFS服务要求：

1. 以只读方式共享目录/public，同时只能被example.com域中的用户访问
2. 以读写的方式共享/protected，能被example.com域中的用户访问
3. 访问/protected需要通过kerberos安全加密，密钥可以通过以下URL下载：（xxx）
4. 目录/protected应该包含名为project，拥有人为ldapuser0的子目录
5. 用户ldapuser0能以读写方式访问/protected/project

先配置ldap+kerberos认证

```shell
[root@server0 ~] nslookup classroom.example.com
[root@server0 ~] ntpdate -u classroom.example.com
[root@server0 ~] yum -y install sssd krb5-workstation.x86_64
[root@server0 ~] mkdir /etc/openldap/cacerts
[root@server0 ~] cd /etc/openldap/cacerts/
[root@server0 cacerts] wget http://classroom.example.com/pub/example-ca.crt
[root@server0 cacerts]# wget -O /etc/krb5.keytab http://classroom.example.com/pub/keytabs/server0.keytab
[root@server0 ~] authconfig-tui 
#文本图形界面勾选使用LDAP以及kerberos认证，填写相应的配置
```

配置完成之后以普通用户切换到ldapuser0，输入密码kerberos，再使用klist命令查看权限，接下来配置nfs相关

```shell
[root@server0 ~] vim /etc/exports
[root@server0 ~] cat /etc/exports
/public	*.example.com(ro)
/protected *.example.com(rw,sync,sec=krb5p)
 
[root@server0 ~] exportfs -avr
exporting *.example.com:/protected
exporting *.example.com:/public
[root@server0 ~] systemctl restart rpcbind.service 
root@server0 ~] systemctl restart nfs-secure-server.service 
[root@server0 ~] systemctl restart nfs-server.service 
[root@server0 ~] systemctl enable nfs-secure-server.service 
[root@server0 ~] systemctl enable nfs-server.service 
[root@server0 ~] firewall-cmd --permanent --add-service=nfs
[root@server0 ~] firewall-cmd --permanent --add-service=mountd 
[root@server0 ~] firewall-cmd --permanent --add-service=rpc-bind 
[root@server0 ~] firewall-cmd --reload

[root@server0 ~] mkdir /protected/project
[root@server0 ~] chown ldapuser0 /protected/project
```

### 8. nfs 客户端配置

1. /public挂载在/mnt/nfsmount
2. /protected挂载在/mnt/nfssecure，并使用安全方式，密钥可以通过以下URL下载：（xxx）
3. 用户ldapuser0能够在/mnt/nfssecure/project上创建文件
4. 这些文件系统在系统启动时自动挂载

先配置ldap+kerberos认证,同上

```shell
[root@desktop0 ~] showmount -e 172.25.0.11 #查看server端nfs共享的文件夹
Export list for 172.25.0.11:
/protected *.example.com
/public    *.example.com
[root@desktop0 ~] systemctl start nfs-secure
[root@desktop0 ~] systemctl enable nfs-secure
[root@desktop0 ~] vim /etc/fstab 
[root@desktop0 ~] tail -n 2 /etc/fstab 
172.25.0.11:/public     /mnt/nfsmount   nfs     defaults        0 0
172.25.0.11:/protected  /mnt/nfssecure  nfs     defaults,rw,sec=krb5p   0 0
[root@desktop0 ~] mkdir /mnt/nfsmount
[root@desktop0 ~] mkdir /mnt/nfssecure
[root@desktop0 ~] mount -a
[root@desktop0 ~] df

```

挂载成功之后可验证权限是否正确，用普通用户切换到ldapuser0获取权限后，可以在/mnt/protected/project上创建文件即正确

### 9. 脚本1

创建脚本myscript.sh要求：

1. 当用户执行 ./myscripts.sh redhat时，显示 "Fedora"
2. 当用户执行 ./myscripts.sh Fedora时，显示 "redhat"
3. 否则显示 "Error, Pls input ./myscripts.sh redhat/Fedora"

```shell

```

### 10. 脚本2

创建脚本batchuser.sh,该脚本能实现为系统创建本地用户，并且这些用户的用户名来自一个文件，要求：

1. 此脚本提供一个参数，此参数就是包含用户列表的文件
2. 如果没有提供参数，提示"Error, Pls input ./batchusers.sh filename"，退出并返回相应的值
3. 如果提供的文件不存在，则提示"file not found",退出并返回相应的
4. 创建的用户登录shell为/bin/false
5. 可在以下地址下载用户名列表做测试：xxx

```shell

```

### 11. samba

配置多用户samba挂载，在server上共享samba目录/common，要求如下：

1. 共享名为rhce
2. 共享目录只能被example.com域中计算机访问
3. 该共享目录必须能被浏览
4. 用户user1必须能以只读方式访问该共享，密码为redhat
5. 用户user3，it组必须能以读写方式访问该共享，密码为redhat
6. 此共享必须永久方式挂载在desktop系统的/mnt/dev目录，将使用user1用户作为认证用户
7. 任何用户都可以通过用户user3来临时获得读写权限

```shell
[root@server0 ~] yum -y install samba samba-client
[root@server0 ~] systemctl start smb nmb
[root@server0 ~] systemctl enable smb nmb
[root@server0 ~] firewall-cmd --permanent --add-service=samba
[root@server0 ~] firewall-cmd --reload
[root@server0 ~] mkdir /common
[root@server0 ~] chmod 777 /common/
[root@server0 ~] semanage fcontext -a -t samba_share_t '/common(./*)?'
[root@server0 ~] restorecon -vvRF /common/
[root@server0 ~] vim /etc/samba/smb.conf 
[root@server0 ~] vim /etc/samba/smb.conf 
[root@server0 ~] tail -n 7 /etc/samba/smb.conf 
[rhce]
path = /common
browseable = yes
writable = no
write list = user3, @it
hosts allow = 172.25.
[root@server0 ~] testparm #测试配置文件语法是否正确

[root@server0 ~] smbpasswd -a user1
[root@server0 ~] smbpasswd -a user3
[root@server0 ~] pdbedit -L
[root@server0 ~] systemctl restart smb nmb
```

desktop端

```shell
[root@desktop0 ~] yum -y install samba-client.x86_64 cifs-utils.x86_64 
[root@desktop0 ~] smbclient //172.25.0.11/rhce -U user1 #测试是否能连接成功以及读写权限
[root@desktop0 ~] vim /etc/fstab 
#在fstab中添加以下内容： //172.25.0.11/rhce	/mnt/dev	cifs	defaults,rw,sec=ntlmssp,multiuser,username=user1,password=redhat 0 0
[root@desktop0 ~] mount -a #挂载后再/mnt/dev目录下测试user1的读写权限
[student@desktop0 dev]$ cifscreds add -u user3 server0 #临时获得user3的权限
```

### 12. 自定义命令

在server上创建自定义命令为qstat，此命令将执行 /bin/ps -Ao pid,user,%cpu,%mem,comm,此命令对系统中所有用户有效

在/etc/bashrc中添加： alias qstat='/bin/ps -Ao pid,user,%cpu,%mem,comm' ，再执行 source /etc/bashrc即可

### 13. Apache配置1

### 14. Apache配置2

### 15. Apache配置3

### 16. Apache配置4

### 17. Apache配置5

### 18. smtp-nullclien

### 19. mariaDB数据库配置
