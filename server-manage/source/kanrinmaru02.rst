kanrinmaru02.akb.creationline.com (219.117.239.178)
===================================================

FQDN の設定
-----------

.. code-block:: console

 root@kanrinmaru02:~# hostname --fqdn
 hostname: Unknown host
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# cp -a /etc/hosts /etc/hosts.2012-1016
 root@kanrinmaru02:~# vi /etc/hosts
 root@kanrinmaru02:~# diff -u /etc/hosts.2012-1016 /etc/hosts
 --- /etc/hosts.2012-1016	2012-09-10 17:21:11.931006843 +0900
 +++ /etc/hosts	2012-10-16 12:09:22.990906498 +0900
 @@ -1,3 +1,7 @@
  127.0.0.1 localhost build km dispatcher console
  127.0.0.1 localhost build km dispatcher console
  127.0.0.1 localhost build km dispatcher console
 +
 +# 2012/10/16 add d-higuchi
 +219.117.239.178	kanrinmaru02.akb.creationline.com	kanrinmaru02
 +#
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# hostname --fqdn                        
 kanrinmaru02.akb.creationline.com
 root@kanrinmaru02:~# 

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/05)

/etc/hosts.{allow,deny} の設定
------------------------------

.. code-block:: console

 root@kanrinmaru02:~# cp -a /etc/hosts.allow /etc/hosts.allow.2012-1016
 root@kanrinmaru02:~# vi /etc/hosts.allow
 root@kanrinmaru02:~# diff -u /etc/hosts.allow.2012-1016 /etc/hosts.allow
 --- /etc/hosts.allow.2012-1016	2010-01-12 22:28:22.000000000 +0900
 +++ /etc/hosts.allow	2012-10-16 12:16:25.597034060 +0900
 @@ -8,3 +8,10 @@
  #		for information on rule syntax.
  #		See 'man tcpd' for information on tcp_wrappers
  #
 +
 +# 2012/10/16 add d-higuchi
 +sshd: localhost
 +sshd: 219.117.239.160/255.255.255.224
 +sshd: 192.168.10.0/255.255.255.0
 +sshd: .tyma.nt.ftth4.ppp.infoweb.ne.jp
 +#
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# cp -a /etc/hosts.deny /etc/hosts.deny.2012-1016
 root@kanrinmaru02:~# vi /etc/hosts.deny
 root@kanrinmaru02:~# diff -u /etc/hosts.deny.2012-1016 /etc/hosts.deny
 --- /etc/hosts.deny.2012-1016	2010-01-12 22:28:22.000000000 +0900
 +++ /etc/hosts.deny	2012-10-16 12:17:07.196921128 +0900
 @@ -11,3 +11,7 @@
  #		for information on rule syntax.
  #		See 'man tcpd' for information on tcp_wrappers
  #
 +
 +# 2012/10/16 add d-higuchi
 +sshd: ALL
 +#
 root@kanrinmaru02:~# 

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/05)

chef omnibus インストール
-------------------------

.. code-block:: console

 root@kanrinmaru02:~# chef-client -v
 Chef: 10.12.0
 root@kanrinmaru02:~# 

少し古いがインストール済。

設定ファイルの設置
------------------

.. code-block:: console

 root@kanrinmaru02:~# mkdir /etc/chef        
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# cat > /etc/chef/client.rb
 log_level		:info
 log_location		STDOUT
 chef_server_url	"https://219.117.239.177/organizations/kanrinmaru"
 validation_key		"/etc/chef/kanrinmaru-validator.pem"
 validation_client_name	"kanrinmaru-validator"
 root@kanrinmaru02:~#

.. code-block:: console

 root@kanrinmaru02:~# cat > /etc/chef/kanrinmaru-validator.pem
 -----BEGIN RSA PRIVATE KEY-----
 MIIEpAIBAAKCAQEArCDgwHiNeXifjnxYwaiM5n7mC47n7v5rqUy9rmt769ndyE7O
 
 	(中略)
 
 wPa3z7UMoyMm0aBH4GBw0P23/E7usCBYr43RlDJU4g1bT/Fy3UX8OQ==
 -----END RSA PRIVATE KEY-----
 root@kanrinmaru02:~#

chef-client を実行
------------------

.. code-block:: console

 root@kanrinmaru02:~# chef-client 
 [Tue, 16 Oct 2012 12:24:10 +0900] INFO: *** Chef 10.12.0 ***
 [Tue, 16 Oct 2012 12:24:11 +0900] INFO: Client key /etc/chef/client.pem is not present - registering
 [Tue, 16 Oct 2012 12:24:12 +0900] INFO: Run List is []
 [Tue, 16 Oct 2012 12:24:12 +0900] INFO: Run List expands to []
 [Tue, 16 Oct 2012 12:24:12 +0900] INFO: Starting Chef Run for kanrinmaru02.akb.creationline.com
 [Tue, 16 Oct 2012 12:24:12 +0900] INFO: Running start handlers
 [Tue, 16 Oct 2012 12:24:12 +0900] INFO: Start handlers complete.
 [Tue, 16 Oct 2012 12:24:12 +0900] INFO: Loading cookbooks []
 [Tue, 16 Oct 2012 12:24:12 +0900] WARN: Node kanrinmaru02.akb.creationline.com has an empty run list.
 [Tue, 16 Oct 2012 12:24:13 +0900] INFO: Chef Run complete in 0.555470242 seconds
 [Tue, 16 Oct 2012 12:24:13 +0900] INFO: Running report handlers
 [Tue, 16 Oct 2012 12:24:13 +0900] INFO: Report handlers complete
 root@kanrinmaru02:~# 

Chef Server に登録されたことを web で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[yum::epel]
- recipe[fail2ban]
- recipe[logwatch]

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node show kanrinmaru02.akb.creationline.com
 Node Name:   kanrinmaru02.akb.creationline.com
 Environment: _default
 FQDN:        kanrinmaru02.akb.creationline.com
 IP:          219.117.239.178
 Run List:    
 Roles:       
 Recipes:     
 Platform:    centos 6.3
 cf@ubuntu:~/chef-repo$ 

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node run_list add kanrinmaru02.akb.creationline.com 'recipe[chef-client::delete_validation],recipe[yum::epel],recipe[fail2ban],recipe[logwatch]'
 run_list: 
     recipe[chef-client::delete_validation]
     recipe[yum::epel]
     recipe[fail2ban]
     recipe[logwatch]
 cf@ubuntu:~/chef-repo$ 

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node show kanrinmaru02.akb.creationline.com
 Node Name:   kanrinmaru02.akb.creationline.com
 Environment: _default
 FQDN:        kanrinmaru02.akb.creationline.com
 IP:          219.117.239.178
 Run List:    recipe[chef-client::delete_validation], recipe[yum::epel], recipe[fail2ban], recipe[logwatch]
 Roles:       
 Recipes:     
 Platform:    centos 6.3
 cf@ubuntu:~/chef-repo$ 

chef-client を initscripts に登録
---------------------------------

.. code-block:: console

 root@kanrinmaru02:~# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.12.0/distro/redhat/etc/init.d/chef-client /etc/init.d/
 root@kanrinmaru02:~# chmod +x /etc/init.d/chef-client 
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# chkconfig --add chef-client
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# chkconfig --list chef-client
 chef-client    	0:off	1:off	2:off	3:off	4:off	5:off	6:off
 root@kanrinmaru02:~# chkconfig chef-client on
 root@kanrinmaru02:~# chkconfig --list chef-client
 chef-client    	0:off	1:off	2:on	3:on	4:on	5:on	6:off
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# mkdir /var/log/chef
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# /etc/init.d/chef-client start
 Starting chef-client:                                      [  OK  ]
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# tail -f /var/log/chef/client.log 
 # Logfile created on 2012-10-16 12:28:51 +0900 by logger.rb/25413
 [Tue, 16 Oct 2012 12:28:51 +0900] INFO: Daemonizing..
 [Tue, 16 Oct 2012 12:28:51 +0900] INFO: Forked, in 26697. Privileges: 0 0
 [Tue, 16 Oct 2012 12:29:05 +0900] INFO: *** Chef 10.12.0 ***
 [Tue, 16 Oct 2012 12:29:06 +0900] INFO: Run List is [recipe[chef-client::delete_validation], recipe[yum::epel], recipe[fail2ban], recipe[logwatch]]
 [Tue, 16 Oct 2012 12:29:06 +0900] INFO: Run List expands to [chef-client::delete_validation, yum::epel, fail2ban, logwatch]
 [Tue, 16 Oct 2012 12:29:06 +0900] INFO: Starting Chef Run for kanrinmaru02.akb.creationline.com
 [Tue, 16 Oct 2012 12:29:06 +0900] INFO: Running start handlers
 [Tue, 16 Oct 2012 12:29:06 +0900] INFO: Start handlers complete.
 [Tue, 16 Oct 2012 12:29:07 +0900] INFO: Loading cookbooks [chef-client, fail2ban, logwatch, perl, yum]
 	:
 	:
 	:
 [Tue, 16 Oct 2012 12:29:09 +0900] INFO: Processing file[/etc/chef/kanrinmaru-validator.pem] action delete (chef-client::delete_validation line 21)
 [Tue, 16 Oct 2012 12:29:09 +0900] INFO: file[/etc/chef/kanrinmaru-validator.pem] deleted file at /etc/chef/kanrinmaru-validator.pem
 [Tue, 16 Oct 2012 12:29:09 +0900] INFO: Processing remote_file[/var/chef/cache/epel-release-6-7.noarch.rpm] action create (yum::epel line 38)
 [Tue, 16 Oct 2012 12:29:10 +0900] INFO: Processing rpm_package[epel-release] action nothing (yum::epel line 45)
 [Tue, 16 Oct 2012 12:29:10 +0900] INFO: Processing file[epel-release-cleanup] action delete (yum::epel line 51)
 [Tue, 16 Oct 2012 12:29:10 +0900] INFO: Processing package[fail2ban] action upgrade (fail2ban::default line 19)
 [Tue, 16 Oct 2012 12:29:22 +0900] INFO: package[fail2ban] installing fail2ban-0.8.4-28.el6 from epel repository
 [Tue, 16 Oct 2012 12:29:30 +0900] INFO: package[fail2ban] upgraded from uninstalled to 0.8.4-28.el6
 [Tue, 16 Oct 2012 12:29:30 +0900] INFO: Processing template[/etc/fail2ban/fail2ban.conf] action create (fail2ban::default line 24)
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: template[/etc/fail2ban/fail2ban.conf] backed up to /var/chef/backup/etc/fail2ban/fail2ban.conf.chef-20121016122931
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: template[/etc/fail2ban/fail2ban.conf] mode changed to 644
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: template[/etc/fail2ban/fail2ban.conf] updated content
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: Processing template[/etc/fail2ban/jail.conf] action create (fail2ban::default line 24)
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: template[/etc/fail2ban/jail.conf] backed up to /var/chef/backup/etc/fail2ban/jail.conf.chef-20121016122931
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: template[/etc/fail2ban/jail.conf] mode changed to 644
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: template[/etc/fail2ban/jail.conf] updated content
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: template[/etc/fail2ban/jail.conf] not queuing delayed action restart on service[fail2ban] (delayed), as it's already been queued
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: Processing service[fail2ban] action enable (fail2ban::default line 33)
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: service[fail2ban] enabled
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: Processing service[fail2ban] action start (fail2ban::default line 33)
 [Tue, 16 Oct 2012 12:29:31 +0900] INFO: service[fail2ban] started
 	:
 	:
 	:
 [Tue, 16 Oct 2012 12:29:49 +0900] INFO: Processing package[logwatch] action install (logwatch::default line 22)
 [Tue, 16 Oct 2012 12:29:50 +0900] INFO: package[logwatch] installing logwatch-7.3.6-49.el6 from base repository
 [Tue, 16 Oct 2012 12:29:57 +0900] INFO: package[logwatch] installed version 7.3.6-49.el6
 [Tue, 16 Oct 2012 12:29:57 +0900] INFO: Processing template[/etc/logwatch/conf/logwatch.conf] action create (logwatch::default line 25)
 [Tue, 16 Oct 2012 12:29:57 +0900] INFO: template[/etc/logwatch/conf/logwatch.conf] backed up to /var/chef/backup/etc/logwatch/conf/logwatch.conf.chef-20121016122957
 [Tue, 16 Oct 2012 12:29:57 +0900] INFO: template[/etc/logwatch/conf/logwatch.conf] mode changed to 644
 [Tue, 16 Oct 2012 12:29:57 +0900] INFO: template[/etc/logwatch/conf/logwatch.conf] updated content
 [Tue, 16 Oct 2012 12:29:57 +0900] INFO: template[/etc/fail2ban/fail2ban.conf] sending restart action to service[fail2ban] (delayed)
 [Tue, 16 Oct 2012 12:29:57 +0900] INFO: Processing service[fail2ban] action restart (fail2ban::default line 33)
 [Tue, 16 Oct 2012 12:30:00 +0900] INFO: service[fail2ban] restarted
 [Tue, 16 Oct 2012 12:30:00 +0900] INFO: Chef Run complete in 53.616599797 seconds
 [Tue, 16 Oct 2012 12:30:00 +0900] INFO: Running report handlers
 [Tue, 16 Oct 2012 12:30:00 +0900] INFO: Report handlers complete
 root@kanrinmaru02:~# 

実行されたことを実際に確認する。

.. code-block:: console

 root@kanrinmaru02:~# ls -la /etc/chef/
 total 16
 drwxr-xr-x.  2 root root 4096 Oct 16 12:29 .
 drwxr-xr-x. 75 root root 4096 Oct 16 12:29 ..
 -rw-------.  1 root root 1679 Oct 16 12:24 client.pem
 -rw-r--r--.  1 root root  205 Oct 16 12:23 client.rb
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# rpm -q epel-release fail2ban
 epel-release-6-7.noarch
 fail2ban-0.8.4-28.el6.noarch
 root@kanrinmaru02:~# 
 root@kanrinmaru02:~# ps auxwwwf | grep '[ f]ail2ban'
 root     27123  0.1  0.0 335344  7588 ?        Sl   12:29   0:00 /usr/bin/python /usr/bin/fail2ban-server -b -s /tmp/fail2ban.sock -x
 root@kanrinmaru02:~# 

.. code-block:: console

 root@kanrinmaru02:~# rpm -q logwatch
 logwatch-7.3.6-49.el6.noarch
 root@kanrinmaru02:~# 

メールドメインの設定
--------------------

run_list に以下を追加するだけでOK

- recipe[postfix]

NTP の設定
----------

run_list に以下を追加するだけでOK

- recipe[ntp]

..
 [EOF]
