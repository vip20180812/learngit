###安装及配置saltstack
* 安装saltrack，推荐使用yum安装
<pre>
yum install https://repo.saltstack.com/yum/redhat/salt-repo-latest-2.el7.noarch.rpm
yum install salt-master   (服务端)
yum install salt-minion   (客户端)
</pre>

* 配置
<pre>
/etc/salt/master   服务端配置文件
/etc/salt/minion   客户端配置文件
客户端主要修改2处：
  1.master: 10.0.0.12  （ip地址为服务端ip地址）
  2.#id:   默认设置就行
</pre>   

* 修改后冲起minion会在/etc/salt下生成minion_id文件，如要修改#ID，
必须删除minion_id文件

* 密钥验证验证交换
<pre>
/etc/salt/pki/master  
[root@node-2 master]# ll
total 8
-r-------- 1 root root 1674 Sep 10 23:40 master.pem
-rw-r--r-- 1 root root  450 Sep 10 23:40 master.pub
drwxr-xr-x 2 root root   58 Sep 10 23:58 minions
drwxr-xr-x 2 root root    6 Sep 10 23:40 minions_autosign
drwxr-xr-x 2 root root    6 Sep 10 23:40 minions_denied
drwxr-xr-x 2 root root    6 Sep 10 23:58 minions_pre
drwxr-xr-x 2 root root    6 Sep 10 23:40 minions_rejected
 
/etc/salt/pki/minion
[root@node-3 minion]# ll
total 12
-rw-r--r-- 1 root root  450 Sep 10 23:58 minion_master.pub
-r-------- 1 root root 1674 Sep 10 23:49 minion.pem
-rw-r--r-- 1 root root  450 Sep 10 23:49 minion.pub
</pre>
* 允许minion加入master
<pre>
[root@node-2 master]# salt-key    查看状态
Accepted Keys:
node-2.localdomain
node-3.localdomain
Denied Keys:
Unaccepted Keys:
Rejected Keys:

salt-key -a node-2.localdomain  允许node-2加入
salt-key -A                     允许所有客户端加入
</pre>

* 远程执行命令
<pre>
 salt '*' test.ping   查看客户端状态  test是模块  ping是动作
 salt 'node-2.localdomain' cmd.run 'w'
</pre>

* 配置管理

state（状态模块）  你要写一个描述文件。格式：YAML  后缀：.sls

YAML:三板斧
   1.缩进   （代表层级关系）
           2个空格，绝对不能使用TAB键，要不然会报错

   2.冒号    （key:value分格）
           :后边有个空格
   
   3.短横向  （列表）
           -：后边有个空格
		   
**自动安装httpd、httpd-devel**
<pre>
[root@node-2 salt]# vim master
file_roots:
  base:
    - /srv/salt

[root@node-2 salt]# systemctl restart salt-master
[root@node-2 salt]# mkdir /srv/salt
[root@node-2 salt]# cd /srv/salt/
[root@node-2 salt]# mkdir web
[root@node-2 salt]# cd web/
[root@node-2 web]# pwd
/srv/salt/web
[root@node-2 web]# vim apache.sls
[root@node-2 web]# cat apache.sls 
apache-install: 
  pkg.installed:
    - names:
      - httpd
      - httpd-devel

apache-service:
  service.running:
    - name: httpd
    - enable: True
	
[root@node-2 ~]# salt '*' state.sls web.apache
</pre>
<pre>
高级状态：
[root@node-2 salt]# vim top.sls
[root@node-2 salt]# cat top.sls 
base:
  'node-2.localdomain':
    - web.apache
  'node-3.localdomain':
    - web.apahce
[root@node-2 salt]# pwd
/srv/salt

[root@node-2 /srv/salt]# salt '*' state.highstate test=Ture  检查
[root@node-2 /srv/salt]# salt '*' state.highstate            执行
</pre>

###Grains
grains是minion第一次启动的时候采集的静态数据，可以用在salt的模块和其他组件中。其实grains在每次的minion启动
（重启）的时候都会采集，即向master汇报一次的。

* 查看minion的全部静态变量
<pre>
[root@node1 ~]# salt '*' grains.items
node2.minion:
    ----------
    SSDs:
    cpu_flags:
        - fpu
        - de
        - pse
        - tsc
        - msr
        - pae
        - mce
        - cx8
        - apic
        - sep
        - mtrr
        - pge
        - mca
        - cmov
        - pse36
        - clflush
        - mmx
        - fxsr
        - sse
        - sse2
        - ht
        - syscall
        - nx
        - lm
        - up
        - rep_good
        - unfair_spinlock
        - pni
        - cx16
        - popcnt
        - hypervisor
        - lahf_lm
        - abm
    cpu_model:
        QEMU Virtual CPU version 1.2.0
    cpuarch:
        x86_64
    domain:
    fqdn:
        localhost
    fqdn_ip4:
        - 127.0.0.1
    fqdn_ip6:
        - ::1
    gpus:
        |_
          ----------
          model:
              GD 5446
          vendor:
              unknown
    host:
        localhost
    hwaddr_interfaces:
        ----------
        eth0:
            52:54:00:05:9e:b4
        lo:
            00:00:00:00:00:00
    id:
        node2.minion
    init:
        upstart
    ip4_interfaces:
        ----------
        eth0:
            - 10.105.61.227
        lo:
            - 127.0.0.1
    ip6_interfaces:
        ----------
        eth0:
        lo:
    ip_interfaces:
        ----------
        eth0:
            - 10.105.61.227
        lo:
            - 127.0.0.1
    ipv4:
        - 10.105.61.227
        - 127.0.0.1
    ipv6:
    kernel:
        Linux
    kernelrelease:
        2.6.32-504.el6.x86_64
    locale_info:
        ----------
        defaultencoding:
            UTF8
        defaultlanguage:
            en_US
        detectedencoding:
            UTF-8
    localhost:
        node2
    lsb_distrib_codename:
        Final
    lsb_distrib_id:
        CentOS
    lsb_distrib_release:
        6.6
    machine_id:
        9293b91f6111b402930f04c20000000f
    master:
        115.29.51.8
    mdadm:
    mem_total:
        996
    nodename:
        node2
    num_cpus:
        1
    num_gpus:
        1
    os:
        CentOS
    os_family:
        RedHat
    osarch:
        x86_64
    oscodename:
        Final
    osfinger:
        CentOS-6
    osfullname:
        CentOS
    osmajorrelease:
        6
    osrelease:
        6.6
    osrelease_info:
        - 6
        - 6
    path:
        /sbin:/usr/sbin:/bin:/usr/bin
    ps:
        ps -efH
    pythonexecutable:
        /usr/bin/python2.6
    pythonpath:
        - /usr/bin
        - /usr/lib64/python26.zip
        - /usr/lib64/python2.6
        - /usr/lib64/python2.6/plat-linux2
        - /usr/lib64/python2.6/lib-tk
        - /usr/lib64/python2.6/lib-old
        - /usr/lib64/python2.6/lib-dynload
        - /usr/lib64/python2.6/site-packages
        - /usr/lib/python2.6/site-packages
        - /usr/lib/python2.6/site-packages/setuptools-0.6c11-py2.6.egg-info
    pythonversion:
        - 2
        - 6
        - 6
        - final
        - 0
    saltpath:
        /usr/lib/python2.6/site-packages/salt
    saltversion:
        2015.5.8
    saltversioninfo:
        - 2015
        - 5
        - 8
        - 0
    selinux:
        ----------
        enabled:
            False
        enforced:
            Disabled
    server_id:
        556532862
    shell:
        /bin/bash
    virtual:
        kvm
    zmqversion:
        3.2.5

</pre>    
* 显示grains的变量名称
<pre>
[root@node1 ~]# salt '*' grains.ls
node2.minion:
    - SSDs
    - cpu_flags
    - cpu_model
    - cpuarch
    - domain
    - fqdn
    - fqdn_ip4
    - fqdn_ip6
    - gpus
    - host
    - hwaddr_interfaces
    - id
    - init
    - ip4_interfaces
    - ip6_interfaces
    - ip_interfaces
    - ipv4
    - ipv6
    - kernel
    - kernelrelease
    - locale_info
    - localhost
    - lsb_distrib_codename
    - lsb_distrib_id
    - lsb_distrib_release
    - machine_id
    - master
    - mdadm
    - mem_total
    - nodename
    - num_cpus
    - num_gpus
    - os
    - os_family
    - osarch
    - oscodename
    - osfinger
    - osfullname
    - osmajorrelease
    - osrelease
    - osrelease_info
    - path
    - ps
    - pythonexecutable
    - pythonpath
    - pythonversion
    - saltpath
    - saltversion
    - saltversioninfo
    - selinux
    - server_id
    - shell
    - virtual
    - zmqversion
</pre>
* 显示某一个变量
<pre>
[root@node1 ~]# salt '*' grains.item os
node2.minion:
    ----------
    os:
        CentOS
获取的是键值对 os：centos

[root@node1 ~]# salt '*' grains.item num_cpus
node2.minion:
----------
num_cpus:
    1
</pre>

* 直接获取内容
<pre>
[root@node1 ~]# salt '*' grains.get os
node2.minion:
    CentOS
</pre>

* -G的使用
<pre>
-G就是grains  
[root@node1 ~]# salt -G 'os:CentOs' test.ping
node2.minion:
    True

如果os的centos的 执行test.ping
</pre>

* 自定义grains
<pre>
在minion端修改配置文件：

    加入以下内容
    grains:
      roles: nginx
      env: prod
    重启服务
在master端执行：

[root@node1 ~]# salt -G 'env:prod' test.ping
node2.minion:
True
</pre>

* 基于文件的grains
<pre>
在minion端
新建grains文件
touch  /etc/salt/grains
写入以下内容：
centos: node2
重启服务

master端执行：

[root@node1 ~]# salt -G 'centos:node2' test.ping
node2.minion:
True
不重启minion端 刷新grains

1.修改minion配置文件
[root@node2 ~]# cat /etc/salt/grains 
centos: node2
test: node2
2.master端刷新
[root@node1 ~]# salt '*' saltutil.sync_grains 
node2.minion:
3.测试
[root@node1 ~]# salt -G 'test:node2' test.ping
node2.minion:
True
</pre>

* 开发一个grains:自定义
<pre>
[root@node-2]# cd /srv/salt/
[root@node-2 /srv/salt]# mkdir _grains

python脚本放在：/srv/salt/_grains
[root@node-2 /srv/salt/_grains]# vim my_grains.py
[root@node-2 /srv/salt/_grains]# cat my_grains.py 
#!/usr/bin/env python
#_*_ coding: utf-8 _*_

def my_grains():
    #初始化一个grains字典
    grains = {}
    #设置字典中的key-vlaue
    grains['iaas'] = 'openstack'
    grains['edu'] = 'oldboyedu'
    #返回这个字典
    return grains

[root@node-2 ~]# salt '*' saltutil.sync_grains    把_grains推到minion端

_grains在minion存在位置/var/cache/salt/minion/extmods/grains

</pre>

###pillar
<pre>
[root@node-2 ~]# vim /etc/salt/master   去掉844-846的注释
 844 pillar_roots:
 845   base:
 846     - /srv/pillar
 
[root@node-2 ~]# mkdir /srv/pillar
[root@node-2 ~]# systemctl restart salt-master
[root@node-2 ~]# cd /srv/pillar/
[root@node-2 /srv/pillar]# mkdir web
[root@node-2 /srv/pillar]# cd web/
[root@node-2 /srv/pillar/web]# vim apache.sls

[root@node-2 /srv/pillar/web]# vim apache.sls
[root@node-2 /srv/pillar/web]# cat apache.sls 

{% if grains['os'] == 'CentOS' %}
apache: httpd
{% elif granins['os'] == 'Debian' %}
apache: apache2
{% endif %}

[root@node-2 /srv/pillar]# vim top.sls
[root@node-2 /srv/pillar]# cat top.sls 
base:
  'node-2.localdomain':
    - web.apache

[root@node-2 /srv/pillar]# salt '*' saltutil.refresh_pillar      刷新

[root@node-2 /srv/pillar]# salt '*' pillar.items apache
node-3.localdomain:
    ----------
    apache:
node-2.localdomain:
    ----------
    apache:
        httpd

</pre>

###深入学习saltstack远程执行

salt '*' cmd.run 'w'

命令： salt
目标： '*'
模块： cmd.run  自带150+模块  自己写模块
返回：执行后结果返回，reternners

目标:targeting
     两种：一种和minion ID 有关的  使用通配符，列表-L ',',正则表达式-E (|)
<pre>
    [root@node-2 ~]# salt 'node-3.localdomain' test.ping
              node-3.localdomain:
              True

1. 通配符：
    [root@node-2 ~]# salt 'node-*' test.ping
    node-2.localdomain:
       True
    node-3.localdomain:
       True
    
2. 列表-L ','：
    [root@node-2 ~]# salt -L 'node-2.localdomain,node-3.localdomain' test.ping
     node-2.localdomain:
     True
     node-3.localdomain:
     True

3. 正则表达式-E :
    [root@node-2 ~]# salt -E '(node-2|node3)*' test.ping
     node-2.localdomain:
     True
     node-3.localdomain:
     True
</pre>

     **所有匹配目标的方式，都可以用到top file里面来制定目标**
<pre>
一种和minion	ID 无关的，

[root@node-2 ~]# salt -S 10.0.0.13 test.ping
node-3.localdomain:
    True
[root@node-2 ~]# salt -S 10.0.0.0/24 test.ping
node-2.localdomain:
    True
node-3.localdomain:
    True
</pre>
* 混合匹配
<pre>
[root@node-2 ~]# salt -C 'S@10.0.0.0/24' test.ping
node-3.localdomain:
    True
node-2.localdomain:
    True

混合匹配参数：
G	Grains glob	G@os:Ubuntu	Yes
E	PCRE Minion ID	E@web\d+\.(dev|qa|prod)\.loc	No
P	Grains PCRE	P@os:(RedHat|Fedora|CentOS)	Yes
L	List of minions	L@minion1.example.com,minion3.domain.com or bl*.domain.com	No
I	Pillar glob	I@pdata:foobar	Yes
J	Pillar PCRE	J@pdata:^(foo|bar)$	Yes
S	Subnet/IP address	S@192.168.1.0/24 or S@192.168.1.100	No
R	Range cluster	R@%foo.bar

组：
vim /etc/salt/master

nodegroups:
  group1: 'L@foo.domain.com,bar.domain.com,baz.domain.com or bl*.domain.com'
  group2: 'G@os:Debian and foo.domain.com'
  group3: 'G@os:Debian and N@group1'
  group4:
    - 'G@foo:bar'
    - 'or'
    - 'G@foo:baz'
</pre>

###模块

[远程执行模块](https://docs.saltstack.com/en/latest/ref/modules/all/index.html#all-salt-modules)

**系统模块所在位置：
[root@node-2 ~]# cd /usr/lib/python2.7/site-packages/salt/modules/**


常用的模块

* network
<pre>
[root@node-2 ~]# salt '*' network.get_hostname
node-2.localdomain:
    node-2.localdomain
node-3.localdomain:
    node-3.localdomain

</pre>
* service
<pre>
[root@node-2 ~]# salt '*' service.status sshd
node-2.localdomain:
    True
node-3.localdomain:
    True
[root@node-2 ~]# salt '*' service.status zabbix
node-2.localdomain:
    False
node-3.localdomain:
    False
[root@node-2 ~]# salt '*' service.status zabbix-agent
node-2.localdomain:
    True
node-3.localdomain:
    True

</pre>
* cp
<pre>
[root@node-2 ~]# salt-cp '*' /etc/hosts /tmp/hehe
node-2.localdomain:
    ----------
    /tmp/hehe:
        True
node-3.localdomain:
    ----------
    /tmp/hehe:
        True
[root@node-2 ~]# cat /tmp/hehe 
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

</pre>
* state
<pre>
[root@node-2 ~]# salt '*' state.show_top
node-2.localdomain:
    ----------
    base:
        - web.apache
node-3.localdomain:
    ----------
    base:
        - web.apache

[root@node-2 /srv/salt]# salt '*' state.highstate test=Ture  检查
[root@node-2 /srv/salt]# salt '*' state.highstate            执行
</pre>

###返回程序  返回到mysql中
[salt返回mysql](https://docs.saltstack.com/en/latest/ref/returners/all/salt.returners.mysql.html)

* 使用salt安装MySQL-python
[root@node-2 ~]# salt '*' state.single pkg.installed name=MySQL-python

1.创建数据库及表：
[root@node-2 ~]# mysql
<pre>
CREATE DATABASE  `salt`
  DEFAULT CHARACTER SET utf8
  DEFAULT COLLATE utf8_general_ci;

USE `salt`;

--
-- Table structure for table `jids`
--

DROP TABLE IF EXISTS `jids`;
CREATE TABLE `jids` (
  `jid` varchar(255) NOT NULL,
  `load` mediumtext NOT NULL,
  UNIQUE KEY `jid` (`jid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
CREATE INDEX jid ON jids(jid) USING BTREE;

--
-- Table structure for table `salt_returns`
--

DROP TABLE IF EXISTS `salt_returns`;
CREATE TABLE `salt_returns` (
  `fun` varchar(50) NOT NULL,
  `jid` varchar(255) NOT NULL,
  `return` mediumtext NOT NULL,
  `id` varchar(255) NOT NULL,
  `success` varchar(10) NOT NULL,
  `full_ret` mediumtext NOT NULL,
  `alter_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  KEY `id` (`id`),
  KEY `jid` (`jid`),
  KEY `fun` (`fun`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

--
-- Table structure for table `salt_events`
--

DROP TABLE IF EXISTS `salt_events`;
CREATE TABLE `salt_events` (
`id` BIGINT NOT NULL AUTO_INCREMENT,
`tag` varchar(255) NOT NULL,
`data` mediumtext NOT NULL,
`alter_time` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
`master_id` varchar(255) NOT NULL,
PRIMARY KEY (`id`),
KEY `tag` (`tag`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

--授权
MariaDB [salt]> grant all on salt.* to salt@'%' identified by 'salt@pw';

</pre>

创建完成后检查数据库
<pre>
[root@node-2 ~]# mysql -h 10.0.0.12 -usalt -p
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| salt               |
| test               |
+--------------------+
3 rows in set (0.00 sec)
MariaDB [salt]> show tables;
+----------------+
| Tables_in_salt |
+----------------+
| jids           |
| salt_events    |
| salt_returns   |
+----------------+

</pre>

* minion端配置：
<pre>
[root@node-3 srv]# vim /etc/salt/minion
mysql.host: '10.0.0.12'
mysql.user: 'salt'
mysql.pass: 'salt@pw'
mysql.db: 'salt'
mysql.port: 3306

[root@node-3 srv]# systemctl restart salt-minion

</pre>

* 执行命令 
<pre>
[root@node-2 ~]# salt '*' test.ping --return mysql

MariaDB [salt]> select * from salt_returns;
+-----------+----------------------+--------+--------------------+---------+------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+
| fun       | jid                  | return | id                 | success | full_ret                                                                                                                                       | alter_time          |
+-----------+----------------------+--------+--------------------+---------+------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+
| test.ping | 20180915014651314865 | true   | node-2.localdomain | 1       | {"fun_args": [], "jid": "20180915014651314865", "return": true, "retcode": 0, "success": true, "fun": "test.ping", "id": "node-2.localdomain"} | 2018-09-15 01:46:51 |
| test.ping | 20180915014651314865 | true   | node-3.localdomain | 1       | {"fun_args": [], "jid": "20180915014651314865", "return": true, "retcode": 0, "success": true, "fun": "test.ping", "id": "node-3.localdomain"} | 2018-09-15 01:46:51 |
+-----------+----------------------+--------+--------------------+---------+------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+

</pre>

###如何编写一个模块

* 放在哪
[root@node-2 ~]# cd /srv/salt/
[root@node-2 /srv/salt]# mkdir _modules

* 命名  文件名就是模块名。
* 编写模块
[root@node-2 /srv/salt]# cd _modules/ 
[root@node-2 /srv/salt/_modules]# vim my_disk.py
[root@node-2 /srv/salt/_modules]# cat my_disk.py 
def list():
  cmd = 'df -h'
  ret = __salt__['cmd.run'](cmd)
  return ret
[root@node-2 /srv/salt/_modules]# salt '*' saltutil.sync_modules
node-2.localdomain:
    - modules.my_disk
node-3.localdomain:
    - modules.my_disk

[root@node-2 ~]# salt '*' saltutil.sync_modules   刷新模块


[root@node-2 ~]# salt '*' my_disk.list
node-3.localdomain:
    Filesystem               Size  Used Avail Use% Mounted on
    /dev/mapper/centos-root   16G  6.2G  9.6G  40% /
    devtmpfs                 904M     0  904M   0% /dev
    tmpfs                    916M   44K  916M   1% /dev/shm
    tmpfs                    916M  9.5M  906M   2% /run
    tmpfs                    916M     0  916M   0% /sys/fs/cgroup
    /dev/sda1                197M  120M   77M  61% /boot
    tmpfs                    184M     0  184M   0% /run/user/0
node-2.localdomain:
    Filesystem               Size  Used Avail Use% Mounted on
    /dev/mapper/centos-root   16G  7.3G  8.6G  47% /
    devtmpfs                 904M     0  904M   0% /dev
    tmpfs                    916M   28K  916M   1% /dev/shm
    tmpfs                    916M  9.4M  906M   2% /run
    tmpfs                    916M     0  916M   0% /sys/fs/cgroup
    /dev/sda1                197M  120M   77M  61% /boot
    tmpfs                    184M     0  184M   0% /run/user/0


###LAMP架构：
1. 安装软件包    pkg
2. 修改配置文件  file
3. 启动服务      service

####pkg
* pkg.installd  安装
* pkg.latest    确保最新版本
* pkg.remove    卸载
* pkg.putge     卸载并删除配置文件

同时安装多个包：
common_packages:
  pkg.downloaded:
    - pkgs:
      - unzip
      - dos2unix
      - salt-minion: 2015.8.5-1.el6

**lamp.sls:内容如下**
<pre>
lamp-pkg:
  pkg.installed:
    - pkgs:
      - httpd
      - php
      - mariadb-server
      - mariadb
      - php-mysql
      - php-cli
      - php-mbstring
     


apache-config:
  file.managed:
    - name: /etc/httpd/conf/httpd.conf
    - source: salt://lamp/file/httpd.conf
    - user: root
    - group: root
    - mode: 644

php-config:
  file.managed:
    - name: /etc/php.ini
    - source: salt://lamp/files/php.ini
    - user: root
    - group: root
    - mode: 644

mysql-config:
  file.managed:
    - name: /etc/my.cnf
    - source: salt://lamp/files/my.cnf
    - user: root
    - group: root
    - mode: 644

apache-service:
  service.running:
    - name: httpd
    - enable: True
    - reload: True

mysql-service:
  service.running:
    -name: mariadb
    - enable: True
    - reload: True

</pre>
 
* salt:// 表示当前环境的跟目录  例如：
 
 file_roots:
base:
  - /srv/salt

 那么 source: salt://lamp/files/my.cnf  表示
 /srv/salt/lamp/files/my.cnf 



* 配置步骤：
<pre>
[root@node-2 /srv/salt/lamp]# cd /srv/salt/
[root@node-2 /srv/salt/lamp]# mkdir lamp
[root@node-2 /srv/salt/lamp]# cd lamp/
[root@node-2 /srv/salt/lamp]# vim lamp.sls
[root@node-2 /srv/salt/lamp]# mkdir files
[root@node-2 /srv/salt/lamp]# cd files/
[root@node-2 /srv/salt/lamp/files]# cp /etc/httpd/conf/httpd.conf .
[root@node-2 /srv/salt/lamp/files]# cp /etc/php.ini .
[root@node-2 /srv/salt/lamp/files]# cp /etc/my.cnf .

[root@node-2 /srv/salt/lamp]# ll
total 4
drwxr-xr-x 2 root root  53 Sep 16 22:38 files
-rw-r--r-- 1 root root 590 Sep 16 22:40 lamp.sls

[root@node-2 /srv/salt/lamp/files]# ll
total 80
-rw-r--r-- 1 root root 11753 Sep 16 22:38 httpd.conf
-rw-r--r-- 1 root root   570 Sep 16 22:38 my.cnf
-rw-r--r-- 1 root root 64945 Sep 16 22:38 php.ini

[root@node-2 /srv/salt/lamp]# salt '*' test.ping
node-2.localdomain:
    True
node-3.localdomain:
    True

[root@node-2 /srv/salt/lamp]# salt 'node-3.localdomain' state.sls lamp.lamp
node-3.localdomain:
.........
.........
Summary for node-3.localdomain
------------
Succeeded: 6
Failed:    0
------------
Total states run:     6
Total run time:   1.559 s


[root@node-3 srv]# netstat -lntp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      883/zabbix_agentd   
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      17938/mysqld        
tcp        0      0 0.0.0.0:50000           0.0.0.0:*               LISTEN      44542/db2sysc       
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      869/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1070/master         
tcp6       0      0 :::10050                :::*                    LISTEN      883/zabbix_agentd   
tcp6       0      0 :::80                   :::*                    LISTEN      868/httpd
</pre>

* 如果报错：请检查.sls文件是否正确

<pre>
node-3.localdomain:
    Data failed to compile:
----------
    ID pkg.installed in SLS lamp.lamp is not a dictionary

解决办法：
[root@node-2 /srv/salt/lamp]# ls
files  lamp.sls
[root@node-2 /srv/salt/lamp]# vim lamp.sls 

</pre>

####状态间关系：

* 我依赖谁  require
<pre>
apache-service:
  service.running:
    - name: httpd
    - enable: True
    - reload: True
    - require:
      - pkg: lamp-pkg
      - file: apache-config


</pre>
* 我监控谁  watch
<pre>
apache-service:
  service.running:
    - name: httpd
    - enable: True
    - reload: True
    - require:
      - pkg: lamp-pkg
    - watch:
      - file: apache-config
1.如果apache-config这个id的状态发生变化就reload
2.如果不加reload: True，那么久restart

实例：
[root@node-2 ~]# netstat -lntp | grep 80
tcp6       0      0 :::80                   :::*                    LISTEN      940/httpd 
修改配置文件：[root@node-2 ~]# vim /srv/salt/lamp/files/httpd.conf     把80端口修改为88端口，看apache是否reload


          ID: apache-config
    Function: file.managed
        Name: /etc/httpd/conf/httpd.conf
      Result: True
     Comment: File /etc/httpd/conf/httpd.conf updated
     Started: 03:23:03.749879
    Duration: 81.541 ms
     Changes:   
              ----------
              diff:
                  --- 
                  +++ 
                  @@ -39,7 +39,7 @@
                   # prevent Apache from glomming onto all bound IP addresses.
                   #
                   #Listen 12.34.56.78:80
                  -Listen 80
                  +Listen 81
                   
                   #
                   # Dynamic Shared Object (DSO) Support

----------
          ID: apache-service
    Function: service.running
        Name: httpd
      Result: True
     Comment: Service reloaded
     Started: 03:29:04.551386
    Duration: 187.398 ms
     Changes:   
              ----------
              httpd:
                  True

[root@node-3 srv]# netstat -lntp | grep 88
tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      883/zabbix_agentd   
tcp6       0      0 :::10050                :::*                    LISTEN      883/zabbix_agentd   
tcp6       0      0 :::88                   :::*                    LISTEN      868/httpd 

[root@node-2 /srv/salt/lamp]# salt 'node-3.localdomain' state.sls lamp.init


</pre>
* 我引用谁   include
<pre>
[root@node-2 /srv/salt/lamp]# ll
total 16
-rw-r--r-- 1 root root 376 Sep 17 02:42 apache.sls
drwxr-xr-x 2 root root  53 Sep 16 22:38 files
-rw-r--r-- 1 root root  53 Sep 17 02:45 lamp.sls
-rw-r--r-- 1 root root 316 Sep 17 02:44 mysql.sls
-rw-r--r-- 1 root root 254 Sep 17 02:43 php.sls

[root@node-2 /srv/salt/lamp]# cat lamp.sls 
include:
  - lamp.apache
  - lamp.php
  - lamp.mysql


[root@node-2 /srv/salt/lamp]# cat apache.sls 

apache-pkg:
  pkg.installed:
    - pkgs:
      - httpd

apache-config:
  file.managed:
    - name: /etc/httpd/conf/httpd.conf
    - source: salt://lamp/files/httpd.conf
    - user: root
    - group: root
    - mode: 644

apache-service:
  service.running:
    - name: httpd
    - enable: True
    - reload: True
    - require:
      - pkg: apache-pkg
      - file: apache-config

[root@node-2 /srv/salt/lamp]# cat php.sls 

php-pkg:
  pkg.installed:
    - pkgs:
      - php
      - php-mysql
      - php-cli
      - php-mbstring

php-config:
  file.managed:
    - name: /etc/php.ini
    - source: salt://lamp/files/php.ini
    - user: root
    - group: root
    - mode: 644

[root@node-2 /srv/salt/lamp]# cat mysql.sls 
mysql-pkg:
  pkg.installed:
    - pkgs:
      - mariadb-server
      - mariadb

mysql-config:
  file.managed:
    - name: /etc/my.cnf
    - source: salt://lamp/files/my.cnf
    - user: root
    - group: root
    - mode: 644

mysql-service:
  service.running:
    - name: mariadb
    - enable: True
    - reload: True

</pre>

如何编写sls技巧：

1.按状态分类 
2.按服务分类  可以被其他sls include

###jinja2:
http://docs.jinkan.org/docs/jinja2/

yaml-jinja
两种分隔符： {% ... %} 和 {{ ... }}

####三步走：
* 告诉file模块，你要使用jinja    
- - template: jinja
<pre>
[root@node-2 /srv/salt/lamp]# cat apache.sls 

apache-pkg:
  pkg.installed:
    - pkgs:
      - httpd

apache-config:
  file.managed:
    - name: /etc/httpd/conf/httpd.conf
    - source: salt://lamp/files/httpd.conf
    - user: root
    - group: root
    - mode: 644
    - template: jinja
</pre>
* 你要列出变量参数列表  
-    - defaults:
-      - POER: 88
<pre>
[root@node-2 /srv/salt/lamp]# cat apache.sls 

apache-pkg:
  pkg.installed:
    - pkgs:
      - httpd

apache-config:
  file.managed:
    - name: /etc/httpd/conf/httpd.conf
    - source: salt://lamp/files/httpd.conf
    - user: root
    - group: root
    - mode: 644
    - template: jinja
    - defaults:
      PORT: 88
</pre>

* 模板引用  {{ PORT }}
<pre>
[root@node-2 /srv/salt/lamp]# vim /srv/salt/lamp/files/httpd.conf
#Listen 12.34.56.78:80
Listen {{ PORT }}
[root@node-2 /srv/salt/lamp]# salt 'node-3.localdomain' state.sls lamp.init
node-3.localdomain:
----------
          ID: apache-pkg
    Function: pkg.installed
      Result: True
     Comment: All specified packages are already installed
     Started: 03:03:03.437468
    Duration: 1196.154 ms
     Changes:
              diff:
                  --- 
                  +++ 
                  @@ -39,7 +39,7 @@
                   # prevent Apache from glomming onto all bound IP addresses.
                   #
                   #Listen 12.34.56.78:80
                  -Listen 88
                  +Listen 89
                   
                   #
                   # Dynamic Shared Object (DSO) Support


    
</pre>

模板里支持：salt  grains  pillar 进行赋值

* grains     {{ grains['fqdn_ip4'] [0] }}:{{ PORT }}
<pre>
[root@node-2 /srv/salt/lamp]# vim /srv/salt/lamp/files/httpd.conf
#Listen 12.34.56.78:80
Listen {{ grains['fqdn_ip4'] [0] }}:{{ PORT }}
          ID: apache-config
    Function: file.managed
        Name: /etc/httpd/conf/httpd.conf
      Result: True
     Comment: File /etc/httpd/conf/httpd.conf updated
     Started: 03:11:51.539578
    Duration: 104.738 ms
     Changes:   
              ----------
              diff:
                  --- 
                  +++ 
                  @@ -39,7 +39,7 @@
                   # prevent Apache from glomming onto all bound IP addresses.
                   #
                   #Listen 12.34.56.78:80
                  -Listen [u'10.0.0.13']:89
                  +Listen 10.0.0.13:89
</pre>

* salt   {{ salt['network.hw_addr']('ens33') }}
<pre>
[root@node-2 /srv/salt/lamp]# vim /srv/salt/lamp/files/httpd.conf 
#Listen 12.34.56.78:80
Listen {{ grains['fqdn_ip4'] [0] }}:{{ PORT }}
# MAC is {{ salt['network.hw_addr']('ens33') }}
#

[root@node-2 /srv/salt/lamp]# salt 'node-3.localdomain' state.sls lamp.init
----------
          ID: apache-config
    Function: file.managed
        Name: /etc/httpd/conf/httpd.conf
      Result: True
     Comment: File /etc/httpd/conf/httpd.conf updated
     Started: 03:18:29.562320
    Duration: 184.872 ms
     Changes:   
              ----------
              diff:
                  --- 
                  +++ 
                  @@ -40,7 +40,7 @@
                   #
                   #Listen 12.34.56.78:80
                   Listen 10.0.0.13:89
                  -# MAC is Interface "ens*" not in available interfaces: "ens34", "lo", "ens38", "ens33"
                  +# MAC is 00:0c:29:7c:9b:8d
                   #
</pre>

* pillar     {{ pillar['apache'] }}
<pre> 
[root@node-2 /srv/salt/lamp]# vim /srv/salt/lamp/files/httpd.conf
[root@node-2 /srv/salt/lamp]# salt 'node-2.localdomain' state.sls lamp.init

#Listen 12.34.56.78:80
Listen {{ grains['fqdn_ip4'] [0] }}:{{ PORT }}
# MAC is {{ salt['network.hw_addr']('ens33') }}
# pillar is  {{ pillar['apache'] }}



          ID: apache-config
    Function: file.managed
        Name: /etc/httpd/conf/httpd.conf
      Result: True
     Comment: File /etc/httpd/conf/httpd.conf updated
     Started: 03:29:40.459072
    Duration: 202.077 ms
     Changes:   
              ----------
              diff:
                  --- 
                  +++ 
                  @@ -39,9 +39,13 @@
                   # prevent Apache from glomming onto all bound IP addresses.
                   #
                   #Listen 12.34.56.78:80
                  -Listen 80
                  -
                  -#
                  +Listen 220.250.64.225:89
                  +# MAC is 00:0c:29:27:37:d8
                  +# pillar is  httpd

</pre>

* 写在模板文件里
{{ grains['fqdn_ip4'] [0] }}:{{ PORT }}
{{ salt['network.hw_addr']('ens33') }}
{{ pillar['apache'] }}
* 写在sls里边的Defaults，变量列表中    
    
-    -defaults:
      PORT: 89


官方优秀模块参考： https://github.com/saltstack-formulas


###base

[root@node-2 ~]# vim /etc/salt/master

file_roots:
  base:
    - /srv/salt/base
    - /srv/salt/prod

pillar_roots:
  base:
    - /srv/pillar/base
  prod:
    - /srv/pillar/prod

[root@node-2 ~]# mkdir -p /srv/salt/base
[root@node-2 ~]# mkdir -p /srv/salt/prod
[root@node-2 ~]# mkdir -p /srv/pillar/base
[root@node-2 ~]# mkdir -p /srv/pillar/prod

[root@node-2 /srv/salt/base/init]# cd /srv/salt/base
[root@node-2 /srv/salt/base/init]# mkdir init
[root@node-2 /srv/salt/base/init]# cd init/
[root@node-2 /srv/salt/base/init]# vim dns.sls
[root@node-2 /srv/salt/base/init]# cat dns.sls 

/etc/resolv.conf:
  file.managed:
    - source: salt://init/files/resolv.conf
    - user: root
    - gourp: root
    - mode: 644

[root@node-2 /srv/salt/base/init]# mkdir files
[root@node-2 /srv/salt/base/init]# cp /etc/resolv.conf files/

[root@node-2 /srv/salt/base/init]# tree 
.
├── dns.sls
└── files
    └── resolv.conf

[root@node-2 /srv/salt/base/init]# cp /etc/zabbix/zabbix_agentd.conf files/ 
[root@node-2 /srv/salt/base/init]# vim files/zabbix_agentd.conf
[root@node-2 /srv/salt/base/init]# cat files/zabbix_agentd.conf | grep Server | grep -v '#'
Server={{ Zabbix_Server }}

###prod：（以编译安装haproxy为例）
<pre>
[root@node-2 /srv/salt/prod]# tree
.
├── haproxy
│   ├── files
│   │   ├── haproxy-1.6.14.tar.gz
│   │   └── haproxy.init
│   └── install.sls
├── keepalived
├── memcached
├── nginx
├── php
└── pkg
    └── make.sls

</pre>

* 基础依赖包安装
<pre>
[root@node-2 /srv/salt/base]# cd /srv/salt/prod/
[root@node-2 /srv/salt/prod]# ls
[root@node-2 /srv/salt/prod]# 
[root@node-2 /srv/salt/prod]# mkdir haproxy
[root@node-2 /srv/salt/prod]# mkdir keepalived
[root@node-2 /srv/salt/prod]# mkdir nginx
[root@node-2 /srv/salt/prod]# mkdir php
[root@node-2 /srv/salt/prod]# mkdir memcached
[root@node-2 /srv/salt/prod]# mkdir pkg
[root@node-2 /srv/salt/prod]# ls
haproxy  keepalived  memcached  nginx  php  pkg
[root@node-2 /srv/salt/prod/pkg]# cd pkg
[root@node-2 /srv/salt/prod/pkg]# vim make.sls
[root@node-2 /srv/salt/prod/pkg]# cat make.sls 
make-pkg:
  pkg.installed:
    - pkgs:
      - gcc
      - gcc-c++
      - glibc
      - make
      - openssl
      - openssl-devel
      - pcre
      - pcre-devel
</pre>
* 编译安装haproxy（安装一遍为salt做准备）
<pre>
[root@node-2 /srv/salt/prod/haproxy]# mkdir files
[root@node-2 /srv/salt/prod/haproxy]# cd files/
[root@node-2 /srv/salt/prod/haproxy/files]# rz  ##上传源码
[root@node-2 /srv/salt/prod/haproxy/files]#tar zxf haproxy-1.6.14.tar.gz
[root@node-2 /srv/salt/prod/haproxy/files/haproxy-1.6.14]# make TARGET=linux2628 PREFIX=/usr/local/haproxy-1.6.14
[root@node-2 /srv/salt/prod/haproxy/files/haproxy-1.6.14]# make install PREFIX=/usr/local/haproxy-1.6.14
[root@node-2 /srv/salt/prod/haproxy/files/haproxy-1.6.14]# cd /usr/local/
[root@node-2 /usr/local]# ln -s /usr/local/haproxy-1.6.14/ /usr/local/haproxy


</pre>
* salt 编译安装haproxy
<pre>
[root@node-2 /usr/local/haproxy]# cd /srv/salt/prod/haproxy/files/haproxy-1.6.14/examples/
[root@node-2 /srv/salt/prod/haproxy/files/haproxy-1.6.14/examples]# vim haproxy.init
BIN=/usr/local/haproxy/sbin/$BASENAME
[root@node-2 /srv/salt/prod/haproxy/files/haproxy-1.6.14/examples]#cp haproxy.init /srv/salt/prod/haproxy/files/
[root@node-2 /srv/salt/prod/haproxy]# vim install.sls
[root@node-2 /srv/salt/prod/haproxy]# cat install.sls 
include:
  - pkg.make

haproxy-install:
  file.managed:
    - name: /usr/local/src/haproxy-1.6.14.tar.gz
    - source: salt://haproxy/files/haproxy-1.6.14.tar.gz
    - mode: 755
    - user: root
    - group: root
  cmd.run:
    - name: cd /usr/local/src && tar zxf haproxy-1.6.14.tar.gz && cd  haproxy-1.6.14 && make TARGET=linux2628 PREFIX=/usr/local/haproxy-1.6.14 && make install PREFIX=/usr/local/haproxy-1.6.14 && ln -s /usr/local/haproxy-1.6.14/ /usr/local/haproxy
    - unless: test -L /usr/local/haproxy
    - require:
      - pkg: make-pkg
      - file: haproxy-install
[root@node-2 /srv/salt/prod/haproxy]# salt '*' state.sls haproxy.install
node-2.localdomain:
    Data failed to compile:
----------
    State 'make-pkg' in SLS 'pkg.make' is not formed as a list
node-3.localdomain:
    Data failed to compile:
----------
    State 'make-pkg' in SLS 'pkg.make' is not formed as a list
ERROR: Minions returned with non-zero exit code

[root@node-2 /srv/salt/prod/haproxy]# salt '*' state.sls haproxy.install env=prod

</pre>

* 详解 haproxy的install.sls
<pre>
[root@node-2 /srv/salt/prod/haproxy]# cat install.sls 
#包括pkg.mkae
include:
  - pkg.make

haproxy-install:
  file.managed:   ##安装文件管理
    - name: /usr/local/src/haproxy-1.6.14.tar.gz
    - source: salt://haproxy/files/haproxy-1.6.14.tar.gz
    - mode: 755
    - user: root
    - group: root
  cmd.run:      ##编译安装
    - name: cd /usr/local/src && tar zxf haproxy-1.6.14.tar.gz && cd  haproxy-1.6.14 && make TARGET=linux2628 PREFIX=/usr/local/haproxy-1.6.14 && make install PREFIX=/usr/local/haproxy-1.6.14 && ln -s /usr/local/haproxy-1.6.14/ /usr/local/haproxy
    - unless: test -L /usr/local/haproxy 
    - require:    #依赖
      - pkg: make-pkg
      - file: haproxy-install
# unless后边的命令返回为True，不执行cmd.run

#管理配置文件
haproxy-init:
  file.managed:
    - name: /etc/init.d/haproxy
    - source: salt://modules/haproxy/files/haproxy.init
    - mode: 755
    - user: root
    - group: root
    - require_in:
      - file: haproxy-install
  cmd.run:
    - name: chkconfig --add haproxy
    - unless: chkconfig --list | grep haproxy
##内核调优
net.ipv4.ip_nonlocal_bind:  
  sysctl.present:
    - value: 1

</pre>

###业务引用（修改路径，把模块全部放到modules里）
<pre>
[root@node-2 /srv/salt/prod]# tree
.
├── haproxy
│   ├── files
│   │   ├── haproxy-1.6.14.tar.gz
│   │   └── haproxy.init
│   └── install.sls
├── keepalived
├── memcached
├── nginx
├── php
└── pkg
    └── make.sls

[root@node-2 /srv/salt/prod]# mkdir modules
[root@node-2 /srv/salt/prod]# mv * modules/
[root@node-2 /srv/salt/prod]# mkdir cluster

[root@node-2 /srv/salt/prod]# tree
.
├── cluster
└── modules
    ├── haproxy
    │   ├── files
    │   │   ├── haproxy-1.6.14.tar.gz
    │   │   └── haproxy.init
    │   └── install.sls
    ├── keepalived
    ├── memcached
    ├── nginx
    ├── php
    └── pkg
        └── make.sls
#修改路径
[root@node-2 /srv/salt/prod]# cat modules/haproxy/install.sls 
include:
  - pkg.make

haproxy-install:
  file.managed:
    - name: /usr/local/src/haproxy-1.6.14.tar.gz
    - source: salt://haproxy/files/haproxy-1.6.14.tar.gz
.......
.......
net.ipv4.ip_nonlocal_bind:
  sysctl.present:
    - value: 1
[root@node-2 /srv/salt/prod]# vim modules/haproxy/install.sls
[root@node-2 /srv/salt/prod]# cat modules/haproxy/install.sls 
include:
  - modules.pkg.make

haproxy-install:
  file.managed:
    - name: /usr/local/src/haproxy-1.6.14.tar.gz
    - source: salt://modules/haproxy/files/haproxy-1.6.14.tar.gz
.......
.......
    - source: salt://modules/haproxy/files/haproxy.init
net.ipv4.ip_nonlocal_bind:
  sysctl.present:
    - value: 1

[root@node-2 /srv/salt/prod]# salt '*' state.sls modules.haproxy.install env=prod
</pre>
###业务
<pre>
[root@node-2 /srv/salt/prod/]#cd cluster
[root@node-2 /srv/salt/prod/cluster]# mkdir files
[root@node-2 /srv/salt/prod/cluster]# cd files
[root@node-2 /srv/salt/prod/cluster]# vim haproxy-outside.cfg
[root@node-2 /srv/salt/prod/cluster/files]# cat haproxy-outside.cfg 

global
maxconn 100000
chroot /usr/local/haproxy
uid 99  
gid 99 
daemon
nbproc 1 
pidfile /usr/local/haproxy/logs/haproxy.pid 
log 127.0.0.1 local3 info

defaults
option http-keep-alive
maxconn 100000
mode http
timeout connect 5000ms
timeout client  50000ms
timeout server 50000ms

listen stats
mode http
bind 0.0.0.0:8888
stats enable
stats uri     /haproxy-status 
stats auth    haproxy:saltstack

frontend frontend_www_example_com
bind 192.168.56.21:80
mode http
option httplog
log global
    default_backend backend_www_example_com

backend backend_www_example_com
option forwardfor header X-REAL-IP
option httpchk HEAD / HTTP/1.0
balance source
server web-node1  10.0.0.12:8080 check inter 2000 rise 30 fall 15
server web-node2  10.0.0.13:8080 check inter 2000 rise 30 fall 15
#穿件sls文件
[root@node-2 /srv/salt/prod/cluster/files]# cd ..

[root@node-2 /srv/salt/prod/cluster]# vim haproxy-outside.sls
[root@node-2 /srv/salt/prod/cluster]# cat haproxy-outside.sls 

include:
  - modules.haproxy.install

haproxy-service:
  file.managed:
    - name: /etc/haproxy/haproxy.cfg
    - source: salt://cluster/files/haproxy-outside.cfg
    - user: root
    - group: root
    - mode: 644
  service.running:
    - name: haproxy
    - enable: True
    - reload: True
    - require:
      - cmd: haproxy-install
    - watch:
      - file: haproxy-service

[root@node-2 /srv/salt/prod/cluster]# vim /srv/salt/base/top.sls 
[root@node-2 /srv/salt/prod/cluster]# cat /srv/salt/base/top.sls 
base:
  '*':
    - init.init

prod:
  '*':
    - cluster.haproxy-outside

[root@node-2 ~]# salt 'node-2*' state.highstate test=True 
</pre>

