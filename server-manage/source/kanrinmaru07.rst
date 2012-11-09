kanrinmaru07.akb.creationline.com (219.117.239.190)
===================================================

FQDN の設定
-----------

.. code-block:: console

 root@kanrinmaru07:~# hostname --fqdn
 kanrinmaru07.akb.creationline.com
 root@kanrinmaru07:~# 

設定済。

/etc/hosts.{allow,deny} の設定
------------------------------

.. code-block:: console

 root@kanrinmaru07:~# cp -a /etc/hosts.allow /etc/hosts.allow.2012-1016
 root@kanrinmaru07:~# vi /etc/hosts.allow
 root@kanrinmaru07:~# diff -u /etc/hosts.allow.2012-1016 /etc/hosts.allow
 --- /etc/hosts.allow.2012-1016	2012-09-27 23:45:36.000000000 +0900
 +++ /etc/hosts.allow	2012-10-17 14:14:58.000000000 +0900
 @@ -11,3 +11,9 @@
  # for further information.
  #
  
 +# 2012/10/16 add d-higuchi
 +sshd: localhost
 +sshd: 219.117.239.160/255.255.255.224
 +sshd: 192.168.10.0/255.255.255.0
 +sshd: .tyma.nt.ftth4.ppp.infoweb.ne.jp
 +#
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# cp -a /etc/hosts.deny /etc/hosts.deny.2012-1016
 root@kanrinmaru07:~# vi /etc/hosts.deny
 root@kanrinmaru07:~# diff -u /etc/hosts.deny.2012-1016 /etc/hosts.deny
 --- /etc/hosts.deny.2012-1016	2012-09-27 23:45:36.000000000 +0900
 +++ /etc/hosts.deny	2012-10-17 14:15:51.000000000 +0900
 @@ -18,3 +18,6 @@
  # versions of Debian this has been the default.
  # ALL: PARANOID
  
 +# 2012/10/16 add d-higuchi
 +sshd: ALL
 +#
 root@kanrinmaru07:~# 

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/05)

chef omnibus インストール
-------------------------

.. code-block:: console

 root@kanrinmaru07:~# curl -L http://opscode.com/chef/install.sh | bash
 	:
 Downloading Chef  for debian...
 Unable to retrieve a valid package!
 Please file a bug report at http://tickets.opscode.com
 Project: Chef
 Component: Packages
 Label: Omnibus
 Version: 
  
 Please detail your operating system type, version and any other relevant details
 URL: http://www.opscode.com/chef/download?v=&p=debian&pv=wheezy/sid&m=x86_64
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# wget 'http://www.opscode.com/chef/download?v=&p=debian&pv=6&m=x86_64'
 	:
 	:
 	:
 root@kanrinmaru07:~# dpkg -i 'download?v=&p=debian&pv=6&m=x86_64'
 Selecting previously unselected package chef.
 (Reading database ... 54078 files and directories currently installed.)
 Unpacking chef (from download?v=&p=debian&pv=6&m=x86_64) ...
 Setting up chef (10.14.4-2.debian.6.0.5) ...
 Thank you for installing Chef!
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# chef-client -v
 Chef: 10.14.4
 root@kanrinmaru07:~# 

設定ファイルの設置
------------------

.. code-block:: console

 root@kanrinmaru07:~# mkdir /etc/chef
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# cat > /etc/chef/client.rb
 log_level		:info
 log_location		STDOUT
 chef_server_url	"https://219.117.239.177/organizations/kanrinmaru"
 validation_key		"/etc/chef/kanrinmaru-validator.pem"
 validation_client_name	"kanrinmaru-validator"
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# cat > /etc/chef/kanrinmaru-validator.pem
 -----BEGIN RSA PRIVATE KEY-----
 MIIEpAIBAAKCAQEArCDgwHiNeXifjnxYwaiM5n7mC47n7v5rqUy9rmt769ndyE7O
 
 	(中略)
 
 wPa3z7UMoyMm0aBH4GBw0P23/E7usCBYr43RlDJU4g1bT/Fy3UX8OQ==
 -----END RSA PRIVATE KEY-----
 root@kanrinmaru07:~# 

chef-client を実行
------------------

.. code-block:: console

 root@kanrinmaru07:~# chef-client
 [2012-10-16T14:44:42+09:00] INFO: *** Chef 10.14.4 ***
 [2012-10-16T14:44:45+09:00] INFO: Client key /etc/chef/client.pem is not present - registering
 [2012-10-16T14:44:46+09:00] INFO: Run List is []
 [2012-10-16T14:44:46+09:00] INFO: Run List expands to []
 [2012-10-16T14:44:46+09:00] INFO: Starting Chef Run for kanrinmaru07.akb.creationline.com
 [2012-10-16T14:44:46+09:00] INFO: Running start handlers
 [2012-10-16T14:44:46+09:00] INFO: Start handlers complete.
 [2012-10-16T14:44:46+09:00] INFO: Loading cookbooks []
 [2012-10-16T14:44:46+09:00] WARN: Node kanrinmaru07.akb.creationline.com has an empty run list.
 [2012-10-16T14:44:46+09:00] INFO: Chef Run complete in 0.242645172 seconds
 [2012-10-16T14:44:46+09:00] INFO: Running report handlers
 [2012-10-16T14:44:46+09:00] INFO: Report handlers complete
 root@kanrinmaru07:~# 

Chef Server に登録されたことを web で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[fail2ban]
- recipe[logwatch]

Debian なので fail2ban インストール用の recipe[yum::epel] は不要。

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node show kanrinmaru07.akb.creationline.com
 Node Name:   kanrinmaru07.akb.creationline.com
 Environment: _default
 FQDN:        kanrinmaru07.akb.creationline.com
 IP:          219.117.239.190
 Run List:    
 Roles:       
 Recipes:     
 Platform:    debian wheezy/sid
 cf@ubuntu:~/chef-repo$ 

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node run_list add kanrinmaru07.akb.creationline.com 'recipe[chef-client::delete_validation],recipe[fail2ban],recipe[logwatch]'
 run_list: 
     recipe[chef-client::delete_validation]
     recipe[fail2ban]
     recipe[logwatch]
 cf@ubuntu:~/chef-repo$

.. code-block:: console

 cf@ubuntu:~/chef-repo$ knife node show kanrinmaru07.akb.creationline.com
 Node Name:   kanrinmaru07.akb.creationline.com
 Environment: _default
 FQDN:        kanrinmaru07.akb.creationline.com
 IP:          219.117.239.190
 Run List:    recipe[chef-client::delete_validation], recipe[fail2ban], recipe[logwatch]
 Roles:       
 Recipes:     
 Platform:    debian wheezy/sid
 cf@ubuntu:~/chef-repo$ 

chef-client を initscripts に登録
---------------------------------

.. code-block:: console

 root@kanrinmaru07:~# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.14.4/distro/debian/etc/default/chef-client /etc/default/
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.14.4/distro/debian/etc/init.d/chef-client /etc/init.d/
 root@kanrinmaru07:~# chmod +x /etc/init.d/chef-client 
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# update-rc.d chef-client defaults 99
 update-rc.d: using dependency based boot sequencing
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# ls -l /etc/*.d/*chef-client*
 -rwxr-xr-x 1 root root 4538 Oct 16 14:47 /etc/init.d/chef-client
 lrwxrwxrwx 1 root root   21 Oct 16 14:50 /etc/rc0.d/K01chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Oct 16 14:50 /etc/rc1.d/K01chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Oct 16 14:50 /etc/rc2.d/S01chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Oct 16 14:50 /etc/rc3.d/S01chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Oct 16 14:50 /etc/rc4.d/S01chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Oct 16 14:50 /etc/rc5.d/S01chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Oct 16 14:50 /etc/rc6.d/K01chef-client -> ../init.d/chef-client
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# mkdir /var/log/chef
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# /etc/init.d/chef-client start
 Starting chef-client : chef-client.
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# tail -f /var/log/chef/client.log 
 # Logfile created on 2012-10-16 14:53:44 +0900 by logger.rb/31641
 [2012-10-16T14:53:44+09:00] INFO: Daemonizing..
 [2012-10-16T14:53:44+09:00] INFO: Forked, in 4769. Privileges: 0 0
 [2012-10-16T14:54:00+09:00] INFO: *** Chef 10.14.4 ***
 [2012-10-16T14:54:01+09:00] INFO: Run List is [recipe[chef-client::delete_validation], recipe[fail2ban], recipe[logwatch]]
 [2012-10-16T14:54:01+09:00] INFO: Run List expands to [chef-client::delete_validation, fail2ban, logwatch]
 [2012-10-16T14:54:01+09:00] INFO: Starting Chef Run for kanrinmaru07.akb.creationline.com
 [2012-10-16T14:54:01+09:00] INFO: Running start handlers
 [2012-10-16T14:54:01+09:00] INFO: Start handlers complete.
 [2012-10-16T14:54:01+09:00] INFO: Loading cookbooks [chef-client, fail2ban, logwatch, perl]
 	:
 	:
 	:
 [2012-10-16T14:54:03+09:00] INFO: Processing file[/etc/chef/kanrinmaru-validator.pem] action delete (chef-client::delete_validation line 21)
 [2012-10-16T14:54:03+09:00] INFO: file[/etc/chef/kanrinmaru-validator.pem] deleted file at /etc/chef/kanrinmaru-validator.pem
 [2012-10-16T14:54:03+09:00] INFO: Processing package[fail2ban] action upgrade (fail2ban::default line 19)
 [2012-10-16T14:54:03+09:00] INFO: Processing template[/etc/fail2ban/fail2ban.conf] action create (fail2ban::default line 24)
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] backed up to /var/chef/backup/etc/fail2ban/fail2ban.conf.chef-20121016145403
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] updated content
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] owner changed to 0
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] group changed to 0
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] mode changed to 644
 [2012-10-16T14:54:03+09:00] INFO: Processing template[/etc/fail2ban/jail.conf] action create (fail2ban::default line 24)
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/jail.conf] backed up to /var/chef/backup/etc/fail2ban/jail.conf.chef-20121016145403
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/jail.conf] updated content
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/jail.conf] owner changed to 0
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/jail.conf] group changed to 0
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/jail.conf] mode changed to 644
 [2012-10-16T14:54:03+09:00] INFO: template[/etc/fail2ban/jail.conf] not queuing delayed action restart on service[fail2ban] (delayed), as it's already been queued
 [2012-10-16T14:54:03+09:00] INFO: Processing service[fail2ban] action enable (fail2ban::default line 33)
 [2012-10-16T14:54:03+09:00] INFO: Processing service[fail2ban] action start (fail2ban::default line 33)
 	:
 	:
 	:
 [2012-10-16T14:54:46+09:00] INFO: Processing package[logwatch] action install (logwatch::default line 22)
 [2012-10-16T14:54:52+09:00] INFO: Processing template[/etc/logwatch/conf/logwatch.conf] action create (logwatch::default line 25)
 [2012-10-16T14:54:52+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] updated content
 [2012-10-16T14:54:52+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] owner changed to 0
 [2012-10-16T14:54:52+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] group changed to 0
 [2012-10-16T14:54:52+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] mode changed to 644
 [2012-10-16T14:54:52+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] sending restart action to service[fail2ban] (delayed)
 [2012-10-16T14:54:52+09:00] INFO: Processing service[fail2ban] action restart (fail2ban::default line 33)
 [2012-10-16T14:54:54+09:00] INFO: service[fail2ban] restarted
 [2012-10-16T14:54:54+09:00] INFO: Chef Run complete in 53.625964533 seconds
 [2012-10-16T14:54:54+09:00] INFO: Running report handlers
 [2012-10-16T14:54:54+09:00] INFO: Report handlers complete
 root@kanrinmaru07:~# 

実行されたことを実際に確認する。

.. code-block:: console

 root@kanrinmaru07:~# ls -la /etc/chef/
 total 16
 drwxr-xr-x  2 root root 4096 Oct 16 14:54 .
 drwxr-xr-x 90 root root 4096 Oct 16 14:54 ..
 -rw-------  1 root root 1675 Oct 16 14:44 client.pem
 -rw-r--r--  1 root root  205 Oct 17  2012 client.rb
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# dpkg -l fail2ban
 Desired=Unknown/Install/Remove/Purge/Hold
 | Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
 |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
 ||/ Name           Version      Architecture Description
 +++-==============-============-============-=================================
 ii  fail2ban       0.8.6-3      all          ban hosts that cause multiple aut
 root@kanrinmaru07:~# ps auxwwwf | grep '[ f]ail2ban'
 root      5251  4.3  0.0  54416  7916 ?        Sl   14:55   0:00 /usr/bin/python /usr/bin/fail2ban-server -b -s /tmp/fail2ban.sock
 root@kanrinmaru07:~# 

.. code-block:: console

 root@kanrinmaru07:~# dpkg -l logwatch
 Desired=Unknown/Install/Remove/Purge/Hold
 | Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
 |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
 ||/ Name           Version      Architecture Description
 +++-==============-============-============-=================================
 ii  logwatch       7.4.0+svn201 all          log analyser with nice output wri
 root@kanrinmaru07:~# 

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
