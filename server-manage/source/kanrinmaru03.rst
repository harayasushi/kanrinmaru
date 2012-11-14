kanrinmaru03.akb.creationline.com (219.117.239.179)
===================================================

FQDN の設定
-----------

.. code-block:: console

 [root@kanrinmaru03 ~]# hostname --fqdn
 hostname: Unknown host
 [root@kanrinmaru03 ~]#

.. code-block:: console

 [root@kanrinmaru03 ~]# cp -a /etc/hosts /etc/hosts.2012-1016
 [root@kanrinmaru03 ~]# vi /etc/hosts
 [root@kanrinmaru03 ~]# diff -u /etc/hosts.2012-1016 /etc/hosts
 --- /etc/hosts.2012-1016	2010-01-12 22:28:22.000000000 +0900
 +++ /etc/hosts	2012-10-16 10:37:49.736315560 +0900
 @@ -1,2 +1,6 @@
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
 +
 +# 2012/10/16 add d-higuchi
 +219.117.239.179	kanrinmaru03.akb.creationline.com	kanrinmaru03
 +#
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# hostname --fqdn
 kanrinmaru03.akb.creationline.com
 [root@kanrinmaru03 ~]# 

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/05)

/etc/hosts.{allow,deny} の設定
------------------------------

.. code-block:: console

 [root@kanrinmaru03 ~]# cp -a /etc/hosts.allow /etc/hosts.allow.2012-1016
 [root@kanrinmaru03 ~]# vi /etc/hosts.allow
 [root@kanrinmaru03 ~]# diff -u /etc/hosts.allow.2012-1016 /etc/hosts.allow
 --- /etc/hosts.allow.2012-1016	2010-01-12 22:28:22.000000000 +0900
 +++ /etc/hosts.allow	2012-10-16 11:20:17.608315619 +0900
 @@ -8,3 +8,10 @@
  #		for information on rule syntax.
  #		See 'man tcpd' for information on tcp_wrappers
  #
 +
 +# 2012/10/16 add d-higuchi
 +sshd: locahost
 +sshd: 219.117.239.160/255.255.255.224
 +sshd: 192.168.10.0/255.255.255.0
 +sshd: .tyma.nt.ftth4.ppp.infoweb.ne.jp
 +#
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# cp -a /etc/hosts.deny /etc/hosts.deny.2012-1016
 [root@kanrinmaru03 ~]# vi /etc/hosts.deny
 [root@kanrinmaru03 ~]# diff -u /etc/hosts.deny.2012-1016 /etc/hosts.deny
 --- /etc/hosts.deny.2012-1016	2010-01-12 22:28:22.000000000 +0900
 +++ /etc/hosts.deny	2012-10-16 11:20:53.960292952 +0900
 @@ -11,3 +11,7 @@
  #		for information on rule syntax.
  #		See 'man tcpd' for information on tcp_wrappers
  #
 +
 +# 2012/10/16 add d-higuchi
 +sshd: ALL
 +#
 [root@kanrinmaru03 ~]# 

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/05)

chef omnibus インストール
-------------------------

.. code-block:: console

 [root@kanrinmaru03 ~]# curl -L http://opscode.com/chef/install.sh | bash
 	:
 [root@kanrinmaru03 ~]# chef-client -v
 Chef: 10.14.4
 [root@kanrinmaru03 ~]# 

設定ファイルの設置
------------------

.. code-block:: console

 [root@kanrinmaru03 ~]# mkdir /etc/chef
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# cat > /etc/chef/client.rb
 log_level		:info
 log_location		STDOUT
 chef_server_url	"https://219.117.239.177/organizations/kanrinmaru"
 validation_key		"/etc/chef/kanrinmaru-validator.pem"
 validation_client_name	"kanrinmaru-validator"
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# cat > /etc/chef/kanrinmaru-validator.pem
 -----BEGIN RSA PRIVATE KEY-----
 MIIEpAIBAAKCAQEArCDgwHiNeXifjnxYwaiM5n7mC47n7v5rqUy9rmt769ndyE7O
 
 	(中略)
 
 wPa3z7UMoyMm0aBH4GBw0P23/E7usCBYr43RlDJU4g1bT/Fy3UX8OQ==
 -----END RSA PRIVATE KEY-----
 [root@kanrinmaru03 ~]# 

chef-client を実行
------------------

.. code-block:: console

 [root@kanrinmaru03 ~]# chef-client
 [2012-10-16T10:43:48+09:00] INFO: *** Chef 10.14.4 ***
 [2012-10-16T10:43:48+09:00] INFO: Client key /etc/chef/client.pem is not present - registering
 [2012-10-16T10:43:49+09:00] INFO: Run List is []
 [2012-10-16T10:43:49+09:00] INFO: Run List expands to []
 [2012-10-16T10:43:49+09:00] INFO: Starting Chef Run for kanrinmaru03.akb.creationline.com
 [2012-10-16T10:43:49+09:00] INFO: Running start handlers
 [2012-10-16T10:43:49+09:00] INFO: Start handlers complete.
 [2012-10-16T10:43:49+09:00] INFO: Loading cookbooks []
 [2012-10-16T10:43:49+09:00] WARN: Node kanrinmaru03.akb.creationline.com has an empty run list.
 [2012-10-16T10:43:49+09:00] INFO: Chef Run complete in 0.188759489 seconds
 [2012-10-16T10:43:49+09:00] INFO: Running report handlers
 [2012-10-16T10:43:49+09:00] INFO: Report handlers complete
 [root@kanrinmaru03 ~]# 

Chef Server に登録されたことを web で確認する。

chef-client を initscripts に登録
---------------------------------

.. code-block:: console

 [root@kanrinmaru03 ~]# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.14.4/distro/redhat/etc/init.d/chef-client /etc/init.d/
 [root@kanrinmaru03 ~]# chmod +x /etc/init.d/chef-client 
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# chkconfig --add chef-client
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# chkconfig --list chef-client
 chef-client    	0:off	1:off	2:off	3:off	4:off	5:off	6:off
 [root@kanrinmaru03 ~]# chkconfig chef-client on
 [root@kanrinmaru03 ~]# chkconfig --list chef-client
 chef-client    	0:off	1:off	2:on	3:on	4:on	5:on	6:off
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# mkdir /var/log/chef
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# /etc/init.d/chef-client start
 Starting chef-client:                                      [  OK  ]
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# ps auxwwwwf | grep '[ c]hef-client'
 root      7703  0.6  0.2 211688 33204 ?        Sl   10:45   0:00 /opt/chef/embedded/bin/ruby /usr/bin/chef-client -d -c /etc/chef/client.rb -L /var/log/chef/client.log -P /var/run/chef/client.pid -i 1800 -s 20
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# tail -f /var/log/chef/client.log 
 [2012-10-16T10:45:47+09:00] INFO: Run List is []
 [2012-10-16T10:45:47+09:00] INFO: Run List expands to []
 [2012-10-16T10:45:47+09:00] INFO: Starting Chef Run for kanrinmaru03.akb.creationline.com
 [2012-10-16T10:45:47+09:00] INFO: Running start handlers
 [2012-10-16T10:45:47+09:00] INFO: Start handlers complete.
 [2012-10-16T10:45:47+09:00] INFO: Loading cookbooks []
 [2012-10-16T10:45:47+09:00] WARN: Node kanrinmaru03.akb.creationline.com has an empty run list.
 [2012-10-16T10:45:47+09:00] INFO: Chef Run complete in 0.196015874 seconds
 [2012-10-16T10:45:47+09:00] INFO: Running report handlers
 [2012-10-16T10:45:47+09:00] INFO: Report handlers complete

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[yum::epel]
- recipe[fail2ban]
- recipe[logwatch]

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node show kanrinmaru03.akb.creationline.com
 Node Name:   kanrinmaru03.akb.creationline.com
 Environment: _default
 FQDN:        kanrinmaru03.akb.creationline.com
 IP:          219.117.239.179
 Run List:    
 Roles:       
 Recipes:     
 Platform:    centos 6.3
 cf@ubuntu:~/chef-repo$ 

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node run_list add kanrinmaru03.akb.creationline.com 'recipe[chef-client::delete_validation],recipe[yum::epel],recipe[fail2ban],recipe[logwatch]'
 run_list: 
     recipe[chef-client::delete_validation]
     recipe[yum::epel]
     recipe[fail2ban]
     recipe[logwatch]
 cf@ubuntu:~/chef-repo$ 

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node show kanrinmaru03.akb.creationline.com
 Node Name:   kanrinmaru03.akb.creationline.com
 Environment: _default
 FQDN:        kanrinmaru03.akb.creationline.com
 IP:          219.117.239.179
 Run List:    recipe[chef-client::delete_validation], recipe[yum::epel], recipe[fail2ban], recipe[logwatch]
 Roles:       
 Recipes:     
 Platform:    centos 6.3
 cf@ubuntu:~/chef-repo$ 

chef-client の実行は 1800±20秒なので、しばらく待つ。

.. code-block:: console

 [root@kanrinmaru03 ~]# tail -f /var/log/chef/client.log 
 [2012-10-16T11:15:50+09:00] INFO: *** Chef 10.14.4 ***
 [2012-10-16T11:15:50+09:00] INFO: Run List is [recipe[chef-client::delete_validation], recipe[yum::epel], recipe[fail2ban], recipe[logwatch]]
 [2012-10-16T11:15:50+09:00] INFO: Run List expands to [chef-client::delete_validation, yum::epel, fail2ban, logwatch]
 [2012-10-16T11:15:50+09:00] INFO: Starting Chef Run for kanrinmaru03.akb.creationline.com
 [2012-10-16T11:15:50+09:00] INFO: Running start handlers
 [2012-10-16T11:15:50+09:00] INFO: Start handlers complete.
 [2012-10-16T11:15:51+09:00] INFO: Loading cookbooks [chef-client, fail2ban, logwatch, perl, yum]
 	:
 	:
 	:
 [2012-10-16T11:15:54+09:00] INFO: Processing file[/etc/chef/kanrinmaru-validator.pem] action delete (chef-client::delete_validation line 21)
 [2012-10-16T11:15:54+09:00] INFO: file[/etc/chef/kanrinmaru-validator.pem] deleted file at /etc/chef/kanrinmaru-validator.pem
 [2012-10-16T11:15:54+09:00] INFO: Processing remote_file[/var/chef/cache/epel-release-6-7.noarch.rpm] action create (yum::epel line 38)
 [2012-10-16T11:15:54+09:00] INFO: remote_file[/var/chef/cache/epel-release-6-7.noarch.rpm] updated
 [2012-10-16T11:15:54+09:00] INFO: remote_file[/var/chef/cache/epel-release-6-7.noarch.rpm] sending install action to rpm_package[epel-release] (immediate)
 [2012-10-16T11:15:54+09:00] INFO: Processing rpm_package[epel-release] action install (yum::epel line 45)
 [2012-10-16T11:15:55+09:00] INFO: Processing rpm_package[epel-release] action nothing (yum::epel line 45)
 [2012-10-16T11:15:55+09:00] INFO: Processing file[epel-release-cleanup] action delete (yum::epel line 51)
 [2012-10-16T11:15:55+09:00] INFO: file[epel-release-cleanup] backed up to /var/chef/backup/var/chef/cache/epel-release-6-7.noarch.rpm.chef-20121016111555
 [2012-10-16T11:15:55+09:00] INFO: file[epel-release-cleanup] deleted file at /var/chef/cache/epel-release-6-7.noarch.rpm
 [2012-10-16T11:15:55+09:00] INFO: Processing package[fail2ban] action upgrade (fail2ban::default line 19)
 [2012-10-16T11:16:01+09:00] INFO: package[fail2ban] installing fail2ban-0.8.4-28.el6 from epel repository
 [2012-10-16T11:16:18+09:00] INFO: package[fail2ban] upgraded from uninstalled to 0.8.4-28.el6
 [2012-10-16T11:16:18+09:00] INFO: Processing template[/etc/fail2ban/fail2ban.conf] action create (fail2ban::default line 24)
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] backed up to /var/chef/backup/etc/fail2ban/fail2ban.conf.chef-20121016111618
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] updated content
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] owner changed to 0
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] group changed to 0
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] mode changed to 644
 [2012-10-16T11:16:18+09:00] INFO: Processing template[/etc/fail2ban/jail.conf] action create (fail2ban::default line 24)
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/jail.conf] backed up to /var/chef/backup/etc/fail2ban/jail.conf.chef-20121016111618
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/jail.conf] updated content
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/jail.conf] owner changed to 0
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/jail.conf] group changed to 0
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/jail.conf] mode changed to 644
 [2012-10-16T11:16:18+09:00] INFO: template[/etc/fail2ban/jail.conf] not queuing delayed action restart on service[fail2ban] (delayed), as it's already been queued
 [2012-10-16T11:16:18+09:00] INFO: Processing service[fail2ban] action enable (fail2ban::default line 33)
 [2012-10-16T11:16:18+09:00] INFO: service[fail2ban] enabled
 [2012-10-16T11:16:18+09:00] INFO: Processing service[fail2ban] action start (fail2ban::default line 33)
 [2012-10-16T11:16:19+09:00] INFO: service[fail2ban] started
 	:
 	:
 	:
 [2012-10-16T11:16:35+09:00] INFO: Processing package[logwatch] action install (logwatch::default line 22)
 [2012-10-16T11:16:35+09:00] INFO: package[logwatch] installing logwatch-7.3.6-49.el6 from base repository
 [2012-10-16T11:16:43+09:00] INFO: Processing template[/etc/logwatch/conf/logwatch.conf] action create (logwatch::default line 25)
 [2012-10-16T11:16:43+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] backed up to /var/chef/backup/etc/logwatch/conf/logwatch.conf.chef-20121016111643
 [2012-10-16T11:16:43+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] updated content
 [2012-10-16T11:16:43+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] owner changed to 0
 [2012-10-16T11:16:43+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] group changed to 0
 [2012-10-16T11:16:43+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] mode changed to 644
 [2012-10-16T11:16:43+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] sending restart action to service[fail2ban] (delayed)
 [2012-10-16T11:16:43+09:00] INFO: Processing service[fail2ban] action restart (fail2ban::default line 33)
 [2012-10-16T11:16:46+09:00] INFO: service[fail2ban] restarted
 [2012-10-16T11:16:46+09:00] INFO: Chef Run complete in 55.699202938 seconds
 [2012-10-16T11:16:46+09:00] INFO: Running report handlers
 [2012-10-16T11:16:46+09:00] INFO: Report handlers complete
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# ps auxwwwf | grep '[ f]ail2ban'
 root      8133  0.0  0.0 334668  6840 ?        Sl   11:16   0:00 /usr/bin/python /usr/bin/fail2ban-server -b -s /tmp/fail2ban.sock -x
 [root@kanrinmaru03 ~]# 

.. code-block:: console

 [root@kanrinmaru03 ~]# rpm -q logwatch
 logwatch-7.3.6-49.el6.noarch
 [root@kanrinmaru03 ~]# 

メールドメインの設定
--------------------

run_list に以下を追加するだけでOK

- recipe[postfix]

NTP の設定
----------

run_list に以下を追加するだけでOK

- recipe[ntp]

Chef クライアントの設定
-----------------------

デフォルトでは 443/tcp にアクセスするので変更する。

.. code-block:: console

 [root@kanrinmaru03 ~]# chef-client
 [2012-11-14T16:46:15+09:00] INFO: *** Chef 10.14.4 ***
        :
        :
        :
 [2012-11-14T16:46:15+09:00] FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
 [2012-11-14T16:46:15+09:00] FATAL: Net::HTTPServerException: 403 "Forbidden"
 [root@kanrinmaru03 ~]#

/etc/chef/client.rb の chef_server_url のポートを変更する。

.. code-block:: console

 [root@kanrinmaru03 ~]# cp -a /etc/chef/client.rb /etc/chef/client.rb.orig
 [root@kanrinmaru03 ~]# vi /etc/chef/client.rb
 [root@kanrinmaru03 ~]# diff -u /etc/chef/client.rb.orig /etc/chef/client.rb
 --- /etc/chef/client.rb.orig    2012-10-16 10:39:56.883315811 +0900
 +++ /etc/chef/client.rb 2012-11-14 16:46:55.362315376 +0900
 @@ -1,5 +1,5 @@
  log_level              :info
  log_location           STDOUT
 -chef_server_url                "https://219.117.239.177/organizations/kanrinmaru"
 +chef_server_url                "https://219.117.239.177:8443/organizations/kanrinmaru"
  validation_key         "/etc/chef/kanrinmaru-validator.pem"
  validation_client_name "kanrinmaru-validator"
 [root@kanrinmaru03 ~]#

アクセスできるようになった。

.. code-block:: console

 [root@kanrinmaru03 ~]# chef-client
 [2012-11-14T16:47:10+09:00] INFO: *** Chef 10.14.4 ***
 [2012-11-14T16:47:10+09:00] INFO: Run List is [recipe[chef-client::delete_validation], recipe[yum::epel], recipe[fail2ban], recipe[logwatch], recipe[postfix], recipe[ntp]]
        :
        :
        :
 [root@kanrinmaru03 ~]#

..
 [EOF]
