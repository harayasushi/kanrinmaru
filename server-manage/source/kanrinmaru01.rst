kanrinmaru01.akb.creationline.com (219.117.239.177)
===================================================

CentOS 6.3 に Private Chef Server のインストール (Standalone 編)
----------------------------------------------------------------

`Universal Prerequisites`_ と `Standalone Installation`_ を参照すること。

.. _Universal Prerequisites: http://private-chef-docs.opscode.com/installation/prereqs.html
.. _Standalone Installation: http://private-chef-docs.opscode.com/installation/standalone.html

FQDN の設定
-----------

.. code-block:: console

 [root@kanrinmaru01 ~]# hostname --fqdn
 hostname: Unknown host
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# cp -a /etc/hosts /etc/hosts.2012-1015
 [root@kanrinmaru01 ~]# vi /etc/hosts                        
 [root@kanrinmaru01 ~]# diff -u /etc/hosts.2012-1015 /etc/hosts
 --- /etc/hosts.2012-1015	2010-01-12 22:28:22.000000000 +0900
 +++ /etc/hosts	2012-10-15 16:17:09.518586743 +0900
 @@ -1,2 +1,4 @@
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
 +
 +219.117.239.177	kanrinmaru01.akb.creationline.com	kanrinmaru01
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# hostname --fqdn
 kanrinmaru01.akb.creationline.com
 [root@kanrinmaru01 ~]# 

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/05)

NTP の設定
----------

.. code-block:: console

 [root@kanrinmaru01 ~]# ps auxwwwf | grep "[ n]tp"
 ntp       1137  0.0  0.0  30160  1624 ?        Ss   Aug29   0:03 ntpd -u ntp:ntp -p /var/run/ntpd.pid -g
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# chkconfig --list ntpd 
 ntpd           	0:off	1:off	2:on	3:on	4:on	5:on	6:off
 [root@kanrinmaru01 ~]# 

Chef 管理下に移行したら run_list に以下を追加する(2012/11/09)

- recipe[ntp]

mail
----

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -q postfix
 postfix-2.6.6-2.2.el6_1.x86_64
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# telnet localhost 25        
 Trying ::1...
 Connected to localhost.
 Escape character is '^]'.
 220 kanrinmaru01.localdomain ESMTP Postfix
 HELO localhost
 250 kanrinmaru01.localdomain
 MAIL FROM: <cluser>
 250 2.1.0 Ok
 RCPT TO: <d-higuchi@creationline.com>
 250 2.1.5 Ok
 DATA
 354 End data with <CR><LF>.<CR><LF>
 From: cluser
 To: d-higuchi@creationline.com
 Subject: test 1619
      
 test 1619
 .
 250 2.0.0 Ok: queued as BCCC01780285
 RSET
 250 2.0.0 Ok
 QUIT
 221 2.0.0 Bye
 Connection closed by foreign host.
 [root@kanrinmaru01 ~]# 

届いたメール

.. code-block:: console

 Delivered-To: d-higuchi@creationline.com
 Received: by 10.194.19.72 with SMTP id c8csp47987wje;
         Mon, 15 Oct 2012 00:19:44 -0700 (PDT)
 Received: by 10.68.237.231 with SMTP id vf7mr2535465pbc.63.1350285583457;
         Mon, 15 Oct 2012 00:19:43 -0700 (PDT)
 Return-Path: <cluser@kanrinmaru01.localdomain>
 Received: from kanrinmaru01.localdomain (219.117.239.177.static.zoot.jp. [219.117.239.177])
         by mx.google.com with ESMTP id pm3si21276736pbc.341.2012.10.15.00.19.42;
         Mon, 15 Oct 2012 00:19:43 -0700 (PDT)
 Received-SPF: neutral (google.com: 219.117.239.177 is neither permitted nor denied by best guess record for domain of cluser@kanrinmaru01.localdomain) client-ip=219.117.239.177;
 Authentication-Results: mx.google.com; spf=neutral (google.com: 219.117.239.177 is neither permitted nor denied by best guess record for domain of cluser@kanrinmaru01.localdomain) smtp.mail=cluser@kanrinmaru01.localdomain
 Received: from localhost (localhost [IPv6:::1])
 	by kanrinmaru01.localdomain (Postfix) with SMTP id BCCC01780285
 	for <d-higuchi@creationline.com>; Mon, 15 Oct 2012 16:19:17 +0900 (JST)
 From: cluser@kanrinmaru01.localdomain
 To: d-higuchi@creationline.com
 Subject: test 1619
 Message-Id: <20121015071926.BCCC01780285@kanrinmaru01.localdomain>
 Date: Mon, 15 Oct 2012 16:19:17 +0900 (JST)

リレーされている。

crontabs
--------

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -q crontabs
 crontabs-1.10-33.el6.noarch
 [root@kanrinmaru01 ~]# 

git
---

.. code-block:: console

 [root@kanrinmaru01 ~]# yum install git
 	:
 Installed:
   git.x86_64 0:1.7.1-2.el6_0.1
 
 Dependency Installed:
   perl.x86_64 4:5.10.1-127.el6
   perl-Error.noarch 1:0.17015-4.el6
   perl-Git.noarch 0:1.7.1-2.el6_0.1
   perl-Module-Pluggable.x86_64 1:3.90-127.el6
   perl-Pod-Escapes.x86_64 1:1.04-127.el6
   perl-Pod-Simple.x86_64 1:3.13-127.el6
   perl-libs.x86_64 4:5.10.1-127.el6
   perl-version.x86_64 3:0.77-127.el6
   rsync.x86_64 0:3.0.6-9.el6
 
 Complete!
 [root@kanrinmaru01 ~]# 

freetype, libpng
----------------

.. code-block:: console

 [root@kanrinmaru01 ~]# yum install freetype libpng
 	:
 Installed:
   freetype.x86_64 0:2.3.11-6.el6_2.9       libpng.x86_64 2:1.2.49-1.el6_2
 
 Complete!
 [root@kanrinmaru01 ~]# 

apache qpid
-----------

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -qa | grep qpid
 python-qpid-0.14-8.el6.noarch
 qpid-qmf-0.14-7.el6_2.x86_64
 qpid-cpp-server-0.14-16.el6.x86_64
 qpid-tools-0.14-2.el6_2.noarch
 qpid-cpp-client-0.14-16.el6.x86_64
 python-qpid-qmf-0.14-7.el6_2.x86_64
 qpid-cpp-client-ssl-0.14-16.el6.x86_64
 qpid-cpp-server-ssl-0.14-16.el6.x86_64
 [root@kanrinmaru01 ~]# 

停止ではなく削除しておく。

.. code-block:: console

 [root@kanrinmaru01 ~]# yum erase '*qpid*'
 	:
 Removed:
   python-qpid.noarch 0:0.14-8.el6      python-qpid-qmf.x86_64 0:0.14-7.el6_2   
   qpid-cpp-client.x86_64 0:0.14-16.el6 qpid-cpp-client-ssl.x86_64 0:0.14-16.el6
   qpid-cpp-server.x86_64 0:0.14-16.el6 qpid-cpp-server-ssl.x86_64 0:0.14-16.el6
   qpid-qmf.x86_64 0:0.14-7.el6_2       qpid-tools.noarch 0:0.14-2.el6_2        
 
 Dependency Removed:
   matahari.x86_64 0:0.6.0-14.el6
   matahari-agent-lib.x86_64 0:0.6.0-14.el6
   matahari-broker.x86_64 0:0.6.0-14.el6
   matahari-consoles.x86_64 0:0.6.0-14.el6
   matahari-host.x86_64 0:0.6.0-14.el6
   matahari-network.x86_64 0:0.6.0-14.el6
   matahari-python.x86_64 0:0.6.0-14.el6
   matahari-rpc.x86_64 0:0.6.0-14.el6
   matahari-service.x86_64 0:0.6.0-14.el6
   matahari-sysconfig.x86_64 0:0.6.0-14.el6
 
 Complete!
 [root@kanrinmaru01 ~]# 

インストール
------------

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -Uvh ~cluser/private-chef-demo-1.el6.x86_64.rpm 
 Preparing...                ########################################### [100%]
    1:private-chef           ########################################### [100%]
 Thank you for installing Chef!
 [root@kanrinmaru01 ~]# 

ファイアウォールの開放
----------------------

.. code-block:: console

 [root@kanrinmaru01 ~]# iptables -n -L INPUT 
 Chain INPUT (policy ACCEPT)
 target     prot opt source               destination         
 ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
 ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
 ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
 ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
 REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 
 [root@kanrinmaru01 ~]#

.. code-block:: console

 [root@kanrinmaru01 ~]# lokkit --service http
 [root@kanrinmaru01 ~]# lokkit --service https
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# iptables -n -L INPUT  
 Chain INPUT (policy ACCEPT)
 target     prot opt source               destination         
 ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
 ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
 ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
 ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
 ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:80 
 ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:443 
 REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 
 [root@kanrinmaru01 ~]# 

後設定
------

.. code-block:: console

 [root@kanrinmaru01 ~]# private-chef-ctl reconfigure 2>&1 | tee reconfigure.log
 	:
 Chef Server Reconfigured!
 [root@kanrinmaru01 ~]# 

Chef クライアント化設定
-----------------------

/etc/hosts.{allow,deny} の設定
------------------------------

.. code-block:: console

 [root@kanrinmaru01 ~]# grep sshd /etc/hosts.*    
 /etc/hosts.allow:sshd: localhost
 /etc/hosts.allow:sshd: 219.117.239.160/255.255.255.224
 /etc/hosts.allow:sshd: 192.168.10.0/255.255.255.0
 /etc/hosts.allow:sshd: .tyma.nt.ftth4.ppp.infoweb.ne.jp
 /etc/hosts.deny:sshd: ALL
 [root@kanrinmaru01 ~]# 

設定済。

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/05)

chef omnibus インストール
-------------------------

.. code-block:: console

 [root@kanrinmaru01 ~]# chef-client -v
 Chef: 10.12.0
 [root@kanrinmaru01 ~]# 

Private Chef Server がインストール済なので必要ない。

設定ファイルの設置
--------------------

.. code-block:: console

 [root@kanrinmaru01 ~]# ls -la /etc/chef/
 total 8
 drwxrwxr-x   2 root opscode 4096 Oct 15 16:27 .
 drwxr-xr-x. 67 root root    4096 Oct 16 09:18 ..
 [root@kanrinmaru01 ~]# 

.. code-block:: console
 
 [root@kanrinmaru01 ~]# cat > /etc/chef/client.rb
 log_level		:info
 log_location		STDOUT
 chef_server_url	"https://219.117.239.177/organizations/kanrinmaru"
 validation_key		"/etc/chef/kanrinmaru-validator.pem"
 validation_client_name	"kanrinmaru-validator"
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# cat > /etc/chef/kanrinmaru-validator.pem
 -----BEGIN RSA PRIVATE KEY-----
 MIIEpAIBAAKCAQEArCDgwHiNeXifjnxYwaiM5n7mC47n7v5rqUy9rmt769ndyE7O
 
 	(中略)
 
 wPa3z7UMoyMm0aBH4GBw0P23/E7usCBYr43RlDJU4g1bT/Fy3UX8OQ==
 -----END RSA PRIVATE KEY-----
 [root@kanrinmaru01 ~]# 

chef-client の実行
------------------

.. code-block:: console

 [root@kanrinmaru01 ~]# chef-client
 [Tue, 16 Oct 2012 12:49:12 +0900] INFO: *** Chef 10.12.0 ***
 [Tue, 16 Oct 2012 12:49:13 +0900] INFO: Client key /etc/chef/client.pem is not present - registering
 [Tue, 16 Oct 2012 12:49:14 +0900] INFO: Run List is []
 [Tue, 16 Oct 2012 12:49:14 +0900] INFO: Run List expands to []
 [Tue, 16 Oct 2012 12:49:14 +0900] INFO: Starting Chef Run for kanrinmaru01.akb.creationline.com
 [Tue, 16 Oct 2012 12:49:14 +0900] INFO: Running start handlers
 [Tue, 16 Oct 2012 12:49:14 +0900] INFO: Start handlers complete.
 [Tue, 16 Oct 2012 12:49:14 +0900] INFO: Loading cookbooks []
 [Tue, 16 Oct 2012 12:49:14 +0900] WARN: Node kanrinmaru01.akb.creationline.com has an empty run list.
 [Tue, 16 Oct 2012 12:49:15 +0900] INFO: Chef Run complete in 0.440747529 seconds
 [Tue, 16 Oct 2012 12:49:15 +0900] INFO: Running report handlers
 [Tue, 16 Oct 2012 12:49:15 +0900] INFO: Report handlers complete
 [root@kanrinmaru01 ~]# 

Chef Server に登録されたことを web で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[yum::epel]
- recipe[fail2ban]
- recipe[logwatch]

のレシピを追加する。

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node show kanrinmaru01.akb.creationline.com
 Node Name:   kanrinmaru01.akb.creationline.com
 Environment: _default
 FQDN:        kanrinmaru01.akb.creationline.com
 IP:          219.117.239.177
 Run List:    
 Roles:       
 Recipes:     
 Platform:    centos 6.3
 cf@ubuntu:~/chef-repo$ 

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node run_list add kanrinmaru01.akb.creationline.com 'recipe[chef-client::delete_validation],recipe[yum::epel],recipe[fail2ban],recipe[logwatch]'
 run_list: 
     recipe[chef-client::delete_validation]
     recipe[yum::epel]
     recipe[fail2ban]
     recipe[logwatch]
 cf@ubuntu:~/chef-repo$ 

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node show kanrinmaru01.akb.creationline.com
 Node Name:   kanrinmaru01.akb.creationline.com
 Environment: _default
 FQDN:        kanrinmaru01.akb.creationline.com
 IP:          219.117.239.177
 Run List:    recipe[chef-client::delete_validation], recipe[yum::epel], recipe[fail2ban], recipe[logwatch]
 Roles:       
 Recipes:     
 Platform:    centos 6.3
 cf@ubuntu:~/chef-repo$ 

chef-client を initscripts に登録し、実行
-----------------------------------------

.. code-block:: console

 [root@kanrinmaru01 ~]# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/distro/redhat/etc/init.d/chef-client /etc/init.d/
 [root@kanrinmaru01 ~]# chmod +x /etc/init.d/chef-client 
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# chkconfig --add chef-client
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# chkconfig --list chef-client
 chef-client    	0:off	1:off	2:off	3:off	4:off	5:off	6:off
 [root@kanrinmaru01 ~]# chkconfig chef-client on
 [root@kanrinmaru01 ~]# chkconfig --list chef-client
 chef-client    	0:off	1:off	2:on	3:on	4:on	5:on	6:off
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# mkdir /var/log/chef
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# /etc/init.d/chef-client start
 Starting chef-client:                                      [  OK  ]
 [root@kanrinmaru01 ~]#

.. code-block:: console

 [root@kanrinmaru01 ~]# tail -f /var/log/chef/client.log 
 [Tue, 16 Oct 2012 12:56:33 +0900] INFO: Daemonizing..
 [Tue, 16 Oct 2012 12:56:33 +0900] INFO: Forked, in 20389. Privileges: 0 0
 [Tue, 16 Oct 2012 12:56:33 +0900] INFO: *** Chef 10.12.0 ***
 [Tue, 16 Oct 2012 12:56:34 +0900] INFO: Run List is [recipe[chef-client::delete_validation], recipe[yum::epel], recipe[fail2ban], recipe[logwatch]]
 [Tue, 16 Oct 2012 12:56:34 +0900] INFO: Run List expands to [chef-client::delete_validation, yum::epel, fail2ban, logwatch]
 [Tue, 16 Oct 2012 12:56:34 +0900] INFO: Starting Chef Run for kanrinmaru01.akb.creationline.com
 [Tue, 16 Oct 2012 12:56:34 +0900] INFO: Running start handlers
 [Tue, 16 Oct 2012 12:56:34 +0900] INFO: Start handlers complete.
 [Tue, 16 Oct 2012 12:56:34 +0900] INFO: Loading cookbooks [chef-client, fail2ban, logwatch, perl, yum]
 	:
 	:
 	:
 [Tue, 16 Oct 2012 12:56:37 +0900] INFO: Processing file[/etc/chef/kanrinmaru-validator.pem] action delete (chef-client::delete_validation line 21)
 [Tue, 16 Oct 2012 12:56:37 +0900] INFO: file[/etc/chef/kanrinmaru-validator.pem] deleted file at /etc/chef/kanrinmaru-validator.pem
 [Tue, 16 Oct 2012 12:56:37 +0900] INFO: Processing remote_file[/var/chef/cache/epel-release-6-7.noarch.rpm] action create (yum::epel line 38)
 [Tue, 16 Oct 2012 12:56:37 +0900] INFO: remote_file[/var/chef/cache/epel-release-6-7.noarch.rpm] updated
 [Tue, 16 Oct 2012 12:56:37 +0900] INFO: remote_file[/var/chef/cache/epel-release-6-7.noarch.rpm] sending install action to rpm_package[epel-release] (immediate)
 [Tue, 16 Oct 2012 12:56:37 +0900] INFO: Processing rpm_package[epel-release] action install (yum::epel line 45)
 [Tue, 16 Oct 2012 12:56:38 +0900] INFO: rpm_package[epel-release] installed version 6-7
 [Tue, 16 Oct 2012 12:56:38 +0900] INFO: Processing rpm_package[epel-release] action nothing (yum::epel line 45)
 [Tue, 16 Oct 2012 12:56:38 +0900] INFO: Processing file[epel-release-cleanup] action delete (yum::epel line 51)
 [Tue, 16 Oct 2012 12:56:38 +0900] INFO: file[epel-release-cleanup] backed up to /var/chef/backup/var/chef/cache/epel-release-6-7.noarch.rpm.chef-20121016125638
 [Tue, 16 Oct 2012 12:56:38 +0900] INFO: file[epel-release-cleanup] deleted file at /var/chef/cache/epel-release-6-7.noarch.rpm
 [Tue, 16 Oct 2012 12:56:38 +0900] INFO: Processing package[fail2ban] action upgrade (fail2ban::default line 19)
 [Tue, 16 Oct 2012 12:56:44 +0900] INFO: package[fail2ban] installing fail2ban-0.8.4-28.el6 from epel repository
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: package[fail2ban] upgraded from 0.8.2-3.el6.rf to 0.8.4-28.el6
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: Processing template[/etc/fail2ban/fail2ban.conf] action create (fail2ban::default line 24)
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: template[/etc/fail2ban/fail2ban.conf] backed up to /var/chef/backup/etc/fail2ban/fail2ban.conf.chef-20121016125652
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: template[/etc/fail2ban/fail2ban.conf] mode changed to 644
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: template[/etc/fail2ban/fail2ban.conf] updated content
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: Processing template[/etc/fail2ban/jail.conf] action create (fail2ban::default line 24)
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: template[/etc/fail2ban/jail.conf] backed up to /var/chef/backup/etc/fail2ban/jail.conf.chef-20121016125652
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: template[/etc/fail2ban/jail.conf] mode changed to 644
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: template[/etc/fail2ban/jail.conf] updated content
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: template[/etc/fail2ban/jail.conf] not queuing delayed action restart on service[fail2ban] (delayed), as it's already been queued
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: Processing service[fail2ban] action enable (fail2ban::default line 33)
 [Tue, 16 Oct 2012 12:56:52 +0900] INFO: Processing service[fail2ban] action start (fail2ban::default line 33)
 	:
 	:
 	:
 [Tue, 16 Oct 2012 12:57:09 +0900] INFO: Processing package[logwatch] action install (logwatch::default line 22)
 [Tue, 16 Oct 2012 12:57:09 +0900] INFO: Processing template[/etc/logwatch/conf/logwatch.conf] action create (logwatch::default line 25)
 [Tue, 16 Oct 2012 12:57:09 +0900] INFO: template[/etc/logwatch/conf/logwatch.conf] backed up to /var/chef/backup/etc/logwatch/conf/logwatch.conf.chef-20121016125709
 [Tue, 16 Oct 2012 12:57:09 +0900] INFO: template[/etc/logwatch/conf/logwatch.conf] mode changed to 644
 [Tue, 16 Oct 2012 12:57:09 +0900] INFO: template[/etc/logwatch/conf/logwatch.conf] updated content
 [Tue, 16 Oct 2012 12:57:09 +0900] INFO: template[/etc/fail2ban/fail2ban.conf] sending restart action to service[fail2ban] (delayed)
 [Tue, 16 Oct 2012 12:57:09 +0900] INFO: Processing service[fail2ban] action restart (fail2ban::default line 33)
 [Tue, 16 Oct 2012 12:57:12 +0900] INFO: service[fail2ban] restarted
 [Tue, 16 Oct 2012 12:57:12 +0900] INFO: Chef Run complete in 38.014457962 seconds
 [Tue, 16 Oct 2012 12:57:12 +0900] INFO: Running report handlers
 [Tue, 16 Oct 2012 12:57:12 +0900] INFO: Report handlers complete

実行されたことを実際に確認する。

.. code-block:: console

 [root@kanrinmaru01 ~]# ls -la /etc/chef/
 total 16
 drwxrwxr-x   2 root opscode 4096 Oct 16 12:56 .
 drwxr-xr-x. 69 root root    4096 Oct 16 12:56 ..
 -rw-------   1 root root    1679 Oct 16 12:49 client.pem
 -rw-r--r--   1 root root     205 Oct 16 12:47 client.rb
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -q epel-release fail2ban
 epel-release-6-7.noarch
 fail2ban-0.8.4-28.el6.noarch
 [root@kanrinmaru01 ~]# 
 [root@kanrinmaru01 ~]# ps auxwwwf | grep '[ f]ail2ban'
 root     25502  0.0  0.0 176568  5312 ?        S    Oct15   0:00 /usr/bin/python /usr/bin/fail2ban-server -b -s /var/run/fail2ban/fail2ban.sock
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -q logwatch
 logwatch-7.3.6-49.el6.noarch
 [root@kanrinmaru01 ~]# 

nginx のアクセス制限
--------------------

nginx はホスト名で allow/deny が書けないことに注意。

.. code-block:: console

 [root@kanrinmaru01 ~]# cp -a /var/opt/opscode/nginx/etc/nginx.conf /var/opt/opscode/nginx/etc/nginx.conf.2012-1017
 [root@kanrinmaru01 ~]# vi /var/opt/opscode/nginx/etc/nginx.conf
 [root@kanrinmaru01 ~]# diff -u /var/opt/opscode/nginx/etc/nginx.conf.2012-1017 /var/opt/opscode/nginx/etc/nginx.conf
 --- /var/opt/opscode/nginx/etc/nginx.conf.2012-1017	2012-10-15 16:28:49.293812507 +0900
 +++ /var/opt/opscode/nginx/etc/nginx.conf	2012-10-17 12:39:56.054586930 +0900
 @@ -53,12 +53,26 @@
      listen 80;
      server_name kanrinmaru01.akb.creationline.com;
      access_log /var/log/opscode/nginx/rewrite-port-80.log;
 +# 2012/10/17 add d-higuchi
 +    allow 127.0.0.1;
 +    allow 219.117.239.160/27;
 +    allow 192.168.0.0/16;
 +    allow 115.177.128.0/17;
 +    deny  all;
 +#
      rewrite ^(.*) https://$server_name$1 permanent;
    }
  
    server { 
      listen 443; 
      server_name kanrinmaru01.akb.creationline.com; 
 +# 2012/10/17 add d-higuchi
 +    allow 127.0.0.1;
 +    allow 219.117.239.160/27;
 +    allow 192.168.0.0/16;
 +    allow 115.177.128.0/17;
 +    deny  all;
 +#
      access_log /var/log/opscode/nginx/access.log opscode;
      ssl on; 
      ssl_certificate /var/opt/opscode/nginx/ca/kanrinmaru01.akb.creationline.com.crt;
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# private-chef-ctl nginx hup
 [root@kanrinmaru01 ~]# 

メールドメインの設定
--------------------

run_list に以下を追加するだけでOK

- recipe[postfix]

リバースプロキシの設定
----------------------

apache のインストール。
細かい調整が必要なので、Opscode Cookbooks からはインストールしない。

.. code-block:: console

 [root@kanrinmaru01 ~]# yum install httpd mod_ssl
        :
        :
        :
 Installed:
   httpd.x86_64 0:2.2.15-15.el6.centos.1 mod_ssl.x86_64 1:2.2.15-15.el6.centos.1

 Dependency Installed:
   apr.x86_64 0:1.3.9-5.el6_2
   apr-util.x86_64 0:1.3.9-3.el6_0.1
   apr-util-ldap.x86_64 0:1.3.9-3.el6_0.1
   httpd-tools.x86_64 0:2.2.15-15.el6.centos.1

 Complete!
 [root@kanrinmaru01 ~]#

80/tcp を止める。

.. code-block:: console

 [root@kanrinmaru01 ~]# cp -a /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.2012-1114
 [root@kanrinmaru01 ~]# vi /etc/httpd/conf/httpd.conf
 [root@kanrinmaru01 ~]# diff -u /etc/httpd/conf/httpd.conf.2012-1114 /etc/httpd/conf/httpd.conf
 --- /etc/httpd/conf/httpd.conf.2012-1114        2012-02-07 23:47:02.000000000 +0900
 +++ /etc/httpd/conf/httpd.conf  2012-11-14 16:16:37.265409044 +0900
 @@ -133,7 +133,9 @@
  # prevent Apache from glomming onto all bound IP addresses (0.0.0.0)
  #
  #Listen 12.34.56.78:80
 -Listen 80
 +# 2012/11/14 d-higuchi stop
 +#Listen 80
 +#

  #
  # Dynamic Shared Object (DSO) Support
 [root@kanrinmaru01 ~]#

リバースプロキシを 8443/tcp に設定する。

.. code-block:: console

 [root@kanrinmaru01 ~]# cp -a /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/ssl.conf.2012-1114
 [root@kanrinmaru01 ~]# vi /etc/httpd/conf.d/ssl.conf
 [root@kanrinmaru01 ~]# diff -u /etc/httpd/conf.d/ssl.conf.2012-1114 /etc/httpd/conf.d/ssl.conf
 --- /etc/httpd/conf.d/ssl.conf.2012-1114        2012-02-07 23:47:02.000000000 +0900
 +++ /etc/httpd/conf.d/ssl.conf  2012-11-14 16:24:31.606419320 +0900
 @@ -15,7 +15,10 @@
  # When we also provide SSL we have to listen to the
  # the HTTPS port in addition.
  #
 -Listen 443
 +# 2012/11/14 d-higuchi 443 -> 8443
 +#Listen 443
 +Listen 8443
 +#

  ##
  ##  SSL Global Context
 @@ -71,7 +74,10 @@
  ## SSL Virtual Host Context
  ##
 
 -<VirtualHost _default_:443>
 +# 2012/11/14 d-higuchi 443 -> 8443
 +#<VirtualHost _default_:443>
 +<VirtualHost _default_:8443>
 +#

  # General setup for the virtual host, inherited from global configuration
  #DocumentRoot "/var/www/html"
 @@ -218,5 +224,50 @@
  CustomLog logs/ssl_request_log \
            "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"

 +# 2012/11/14 d-higuchi - Private Chef Reverse Proxy
 +# reverse proxy only
 +ProxyRequests off
 +
 +# SSL proxy
 +SSLProxyEngine on
 +
 +# use mod_rewrite
 +RewriteEngine on
 +RewriteOptions Inherit
 +
 +# API access
 +RewriteCond %{HTTP:X-Ops-Timestamp} .
 +RewriteRule (^/.*$) /chefapi$1 [PT,L]
 +
 +<Location /chefapi>
 +       ProxyPass                       https://127.0.0.1
 +       ProxyPassReverse                https://127.0.0.1
 +       ProxyPassReverseCookieDomain    219.117.239.177 127.0.0.1
 +</Location>
 +
 +<Location /chefui>
 +       order deny,allow
 +       deny from all
 +       allow from 219.117.239.160/255.255.255.224
 +       allow from 192.168.1.0/255.255.255.0
 +       allow from 192.168.2.0/255.255.255.0
 +       allow from 192.168.10.0/255.255.255.0
 +       allow from 115.177.128.0/255.255.128.0  # d-higuchi
 +       allow from 221.249.136.50               # j-hotta
 +
 +       AuthUserFile    /etc/httpd/conf/htpasswd.chefui
 +       AuthName        realm
 +       AuthType        Basic
 +       Require         valid-user
 +
 +       ProxyPass                       https://127.0.1.1
 +       ProxyPassReverse                https://127.0.1.1
 +       ProxyPassReverseCookieDomain    219.117.239.177 127.0.1.1
 +</Location>
 +#
 +
  </VirtualHost>
 
 [root@kanrinmaru01 ~]#

パスワードを設定する。PrimeDrive を参照のこと。

.. code-block:: console

 [root@kanrinmaru01 ~]# htpasswd -c /etc/httpd/conf/htpasswd.chefui chefui
 New password:
 Re-type new password:
 Adding password for user chefui
 [root@kanrinmaru01 ~]#

apache を起動する。

.. code-block:: console

 [root@kanrinmaru01 ~]# /etc/init.d/httpd start
 Starting httpd:                                            [  OK  ]
 [root@kanrinmaru01 ~]#

Private Chef Server の nginx のアクセス制限を強める。

.. code-block:: console

 [root@kanrinmaru01 ~]# cp -a /var/opt/opscode/nginx/etc/nginx.conf /var/opt/opscode/nginx/etc/nginx.conf.2012-1114
 [root@kanrinmaru01 ~]# vi /var/opt/opscode/nginx/etc/nginx.conf
 [root@kanrinmaru01 ~]# diff -u /var/opt/opscode/nginx/etc/nginx.conf.2012-1017 /var/opt/opscode/nginx/etc/nginx.conf
 --- /var/opt/opscode/nginx/etc/nginx.conf.2012-1017     2012-10-15 16:28:49.293812507 +0900
 +++ /var/opt/opscode/nginx/etc/nginx.conf       2012-11-14 16:36:30.440394225 +0900
 @@ -52,6 +52,12 @@
    server {
      listen 80;
      server_name kanrinmaru01.akb.creationline.com;
 +
 +# 2012/11/14 add d-higuchi
 +    allow 127.0.0.1;
 +    deny  all;
 +#
 +
      access_log /var/log/opscode/nginx/rewrite-port-80.log;
      rewrite ^(.*) https://$server_name$1 permanent;
    }
 @@ -59,6 +65,12 @@
    server {
      listen 443;
      server_name kanrinmaru01.akb.creationline.com;
 +
 +# 2012/11/14 add d-higuchi
 +    allow 127.0.0.1;
 +    deny  all;
 +#
 +
      access_log /var/log/opscode/nginx/access.log opscode;
      ssl on;
      ssl_certificate /var/opt/opscode/nginx/ca/kanrinmaru01.akb.creationline.com.crt;
 [root@kanrinmaru01 ~]#

Private Chef Server の nginx を再起動する。

.. code-block:: console

 [root@kanrinmaru01 ~]# private-chef-ctl nginx restart
 ok: run: nginx: (pid 9394) 0s
 [root@kanrinmaru01 ~]#

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/14)

Chef クライアントの設定
-----------------------

デフォルトでは 443/tcp にアクセスするので、もうアクセスできなくなっている。

.. code-block:: console

 [root@kanrinmaru01 ~]# chef-client
 [Wed, 14 Nov 2012 16:39:46 +0900] INFO: *** Chef 10.12.0 ***
 [Wed, 14 Nov 2012 16:39:47 +0900] FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
 [Wed, 14 Nov 2012 16:39:47 +0900] FATAL: Net::HTTPServerException: 403 "Forbidden"
 [root@kanrinmaru01 ~]#

/etc/chef/client.rb の chef_server_url のポートを変更する。

.. code-block:: console

 [root@kanrinmaru01 ~]# cp -a /etc/chef/client.rb /etc/chef/client.rb.orig
 [root@kanrinmaru01 ~]# vi /etc/chef/client.rb
 [root@kanrinmaru01 ~]# diff -u /etc/chef/client.rb.orig /etc/chef/client.rb
 --- /etc/chef/client.rb.orig    2012-10-16 12:47:39.117587049 +0900
 +++ /etc/chef/client.rb 2012-11-14 16:41:30.432394200 +0900
 @@ -1,5 +1,5 @@
  log_level              :info
  log_location           STDOUT
 -chef_server_url                "https://219.117.239.177/organizations/kanrinmaru"
 +chef_server_url                "https://219.117.239.177:8443/organizations/kanrinmaru"
  validation_key         "/etc/chef/kanrinmaru-validator.pem"
  validation_client_name "kanrinmaru-validator"
 [root@kanrinmaru01 ~]#

アクセスできるようになった。

.. code-block:: console

 [root@kanrinmaru01 ~]# chef-client
 [Wed, 14 Nov 2012 16:41:56 +0900] INFO: *** Chef 10.12.0 ***
 [Wed, 14 Nov 2012 16:41:57 +0900] INFO: Run List is [recipe[chef-client::delete_validation], recipe[yum::epel], recipe[fail2ban], recipe[logwatch], recipe[postfix], recipe[ntp]]
        :
        :
        :
 [root@kanrinmaru01 ~]#

ファイアウォールの設定
----------------------

run_list に以下を追加するだけでOK。

- recipe[lokkit::service_ssh]
- recipe[lokkit::https_tcp]
- recipe[lokkit::8443_tcp]

..
 [EOF]
