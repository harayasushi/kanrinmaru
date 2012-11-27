sandbox.akb.creationline.com (219.117.239.188)
==============================================

FQDN の設定
-----------

.. code-block:: console

 root@sandbox:~# hostname --fqdn
 sandbox.akb.creationline.com
 root@sandbox:~# 

設定済。

/etc/hosts.{allow,deny} の設定
------------------------------

.. code-block:: console

 root@sandbox:~# cp -a /etc/hosts.allow /etc/hosts.allow.2012-1101
 root@sandbox:~# vi /etc/hosts.allow
 root@sandbox:~# diff -u /etc/hosts.allow.2012-1101 /etc/hosts.allow
 --- /etc/hosts.allow.2012-1101	2012-11-01 18:05:51.000000000 +0900
 +++ /etc/hosts.allow	2012-11-01 19:07:51.000000000 +0900
 @@ -11,3 +11,9 @@
  # for further information.
  #
  
 +# 2012/11/01 d-higuchi add
 +sshd: localhost
 +sshd: 219.117.239.160/255.255.255.224
 +sshd: 192.168.10.0/255.255.255.0
 +sshd: .tyma.nt.ftth4.ppp.infoweb.ne.jp
 +#
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# cp -a /etc/hosts.deny /etc/hosts.deny.2012-1101
 root@sandbox:~# vi /etc/hosts.deny
 root@sandbox:~# diff -u /etc/hosts.deny.2012-1101 /etc/hosts.deny
 --- /etc/hosts.deny.2012-1101	2012-11-01 18:05:51.000000000 +0900
 +++ /etc/hosts.deny	2012-11-01 19:08:43.000000000 +0900
 @@ -18,3 +18,6 @@
  # versions of Debian this has been the default.
  # ALL: PARANOID
  
 +# 2012/11/01 d-higuchi add
 +sshd: ALL
 +#
 root@sandbox:~# 

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/05)

chef omnibus インストール
-------------------------

.. code-block:: console

 root@sandbox:~# apt-get install curl
 	:
 root@sandbox:~# curl -L http://opscode.com/chef/install.sh | bash
 	:
 Downloading Chef  for debian...
 Installing Chef 
 Selecting previously deselected package chef.
 (Reading database ... 35142 files and directories currently installed.)
 Unpacking chef (from .../tmp.a1jaRhOn/chef__amd64.deb) ...
 Setting up chef (10.16.2-1.debian.6.0.5) ...
 Thank you for installing Chef!
 root@sandbox:~# 

設定ファイルの設置
------------------

.. code-block:: console

 root@sandbox:~# mkdir /etc/chef
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# cat > /etc/chef/client.rb
 log_level		:info
 log_location		STDOUT
 chef_server_url	"https://219.117.239.177/organizations/kanrinmaru"
 validation_key		"/etc/chef/kanrinmaru-validator.pem"
 validation_client_name	"kanrinmaru-validator"
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# cat > /etc/chef/kanrinmaru-validator.pem
 -----BEGIN RSA PRIVATE KEY-----
 MIIEpAIBAAKCAQEArCDgwHiNeXifjnxYwaiM5n7mC47n7v5rqUy9rmt769ndyE7O
 
 	(中略)
 
 wPa3z7UMoyMm0aBH4GBw0P23/E7usCBYr43RlDJU4g1bT/Fy3UX8OQ==
 -----END RSA PRIVATE KEY-----
 root@sandbox:~# 

chef-client を実行
------------------

.. code-block:: console

 root@sandbox:~# chef-client 
 [2012-11-01T19:14:58+09:00] INFO: *** Chef 10.16.2 ***
 [2012-11-01T19:14:59+09:00] INFO: Client key /etc/chef/client.pem is not present - registering
 [2012-11-01T19:15:00+09:00] INFO: Run List is []
 [2012-11-01T19:15:00+09:00] INFO: Run List expands to []
 [2012-11-01T19:15:00+09:00] INFO: Starting Chef Run for sandbox.akb.creationline.com
 [2012-11-01T19:15:00+09:00] INFO: Running start handlers
 [2012-11-01T19:15:00+09:00] INFO: Start handlers complete.
 [2012-11-01T19:15:00+09:00] INFO: Loading cookbooks []
 [2012-11-01T19:15:00+09:00] WARN: Node sandbox.akb.creationline.com has an empty run list.
 [2012-11-01T19:15:00+09:00] INFO: Chef Run complete in 0.219991507 seconds
 [2012-11-01T19:15:00+09:00] INFO: Running report handlers
 [2012-11-01T19:15:00+09:00] INFO: Report handlers complete
 root@sandbox:~# 

Chef Server に登録されたことを web で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[fail2ban]
- recipe[logwatch]
- recipe[postfix]

Web UI で行った。

chef-client を initscripts に登録
---------------------------------

.. code-block:: console

 root@sandbox:~# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.16.2/distro/debian/etc/default/chef-client /etc/default/
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.16.2/distro/debian/etc/init.d/chef-client /etc/init.d/  
 root@sandbox:~# chmod +x /etc/init.d/chef-client 
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# update-rc.d chef-client defaults 99
 update-rc.d: using dependency based boot sequencing
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# ls -l /etc/*.d/*chef-client*
 -rwxr-xr-x 1 root root 4538 Nov  1 19:17 /etc/init.d/chef-client
 lrwxrwxrwx 1 root root   21 Nov  1 19:17 /etc/rc0.d/K01chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Nov  1 19:17 /etc/rc1.d/K01chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Nov  1 19:17 /etc/rc2.d/S17chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Nov  1 19:17 /etc/rc3.d/S17chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Nov  1 19:17 /etc/rc4.d/S17chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Nov  1 19:17 /etc/rc5.d/S17chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 Nov  1 19:17 /etc/rc6.d/K01chef-client -> ../init.d/chef-client
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# mkdir /var/log/chef
 root@sandbox:~#

.. code-block:: console

 root@sandbox:~# /etc/init.d/chef-client start
 Starting chef-client : chef-client.
 root@sandbox:~#

.. code-block:: console

 root@sandbox:~# tail -f /var/log/chef/client.log
 # Logfile created on 2012-11-01 19:18:42 +0900 by logger.rb/31641
 [2012-11-01T19:19:11+09:00] FATAL: SIGTERM received, stopping
 [2012-11-01T19:19:12+09:00] INFO: Daemonizing..
 [2012-11-01T19:19:12+09:00] INFO: Forked, in 12358. Privileges: 0 0
 [2012-11-01T19:19:30+09:00] INFO: *** Chef 10.16.2 ***
 [2012-11-01T19:19:31+09:00] INFO: Run List is [recipe[chef-client::delete_validation], recipe[fail2ban], recipe[logwatch]]
 [2012-11-01T19:19:31+09:00] INFO: Run List expands to [chef-client::delete_validation, fail2ban, logwatch]
 [2012-11-01T19:19:31+09:00] INFO: Starting Chef Run for sandbox.akb.creationline.com
 [2012-11-01T19:19:31+09:00] INFO: Running start handlers
 [2012-11-01T19:19:31+09:00] INFO: Start handlers complete.
 [2012-11-01T19:19:31+09:00] INFO: Loading cookbooks [chef-client, fail2ban, logwatch, perl]
 	:
 	:
 	:
 [2012-11-01T19:19:33+09:00] INFO: Processing file[/etc/chef/kanrinmaru-validator.pem] action delete (chef-client::delete_validation line 21)
 [2012-11-01T19:19:33+09:00] INFO: file[/etc/chef/kanrinmaru-validator.pem] deleted file at /etc/chef/kanrinmaru-validator.pem
 [2012-11-01T19:19:33+09:00] INFO: Processing package[fail2ban] action upgrade (fail2ban::default line 19)
 [2012-11-01T19:19:35+09:00] INFO: package[fail2ban] upgraded from uninstalled to 0.8.4-3+squeeze1
 [2012-11-01T19:19:35+09:00] INFO: Processing template[/etc/fail2ban/fail2ban.conf] action create (fail2ban::default line 24)
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] backed up to /var/chef/backup/etc/fail2ban/fail2ban.conf.chef-20121101191935
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] updated content
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] owner changed to 0
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] group changed to 0
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] mode changed to 644
 [2012-11-01T19:19:35+09:00] INFO: Processing template[/etc/fail2ban/jail.conf] action create (fail2ban::default line 24)
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/jail.conf] backed up to /var/chef/backup/etc/fail2ban/jail.conf.chef-20121101191935
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/jail.conf] updated content
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/jail.conf] owner changed to 0
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/jail.conf] group changed to 0
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/jail.conf] mode changed to 644
 [2012-11-01T19:19:35+09:00] INFO: template[/etc/fail2ban/jail.conf] not queuing delayed action restart on service[fail2ban] (delayed), as it's already been queued
 [2012-11-01T19:19:35+09:00] INFO: Processing service[fail2ban] action enable (fail2ban::default line 33)
 [2012-11-01T19:19:35+09:00] INFO: Processing service[fail2ban] action start (fail2ban::default line 33)
 	:
 	:
 	:
 [2012-11-01T19:19:38+09:00] INFO: Processing package[logwatch] action install (logwatch::default line 22)
 [2012-11-01T19:19:41+09:00] INFO: Processing template[/etc/logwatch/conf/logwatch.conf] action create (logwatch::default line 25)
 [2012-11-01T19:19:41+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] updated content
 [2012-11-01T19:19:41+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] owner changed to 0
 [2012-11-01T19:19:41+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] group changed to 0
 [2012-11-01T19:19:41+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] mode changed to 644
 [2012-11-01T19:19:41+09:00] INFO: template[/etc/fail2ban/fail2ban.conf] sending restart action to service[fail2ban] (delayed)
 [2012-11-01T19:19:41+09:00] INFO: Processing service[fail2ban] action restart (fail2ban::default line 33)
 [2012-11-01T19:19:43+09:00] INFO: service[fail2ban] restarted
 	:
  	:
 	:
 [2012-11-01T19:22:16+09:00] INFO: Processing package[postfix] action install (postfix::default line 21)
 [2012-11-01T19:22:32+09:00] INFO: Processing service[postfix] action enable (postfix::default line 32)
 [2012-11-01T19:22:32+09:00] INFO: Processing template[/etc/postfix/main.cf] action create (postfix::default line 51)
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/main.cf] backed up to /var/chef/backup/etc/postfix/main.cf.chef-20121101192232
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/main.cf] updated content
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/main.cf] owner changed to 0
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/main.cf] group changed to 0
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/main.cf] mode changed to 644
 [2012-11-01T19:22:32+09:00] INFO: Processing template[/etc/postfix/master.cf] action create (postfix::default line 51)
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/master.cf] backed up to /var/chef/backup/etc/postfix/master.cf.chef-20121101192232
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/master.cf] updated content
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/master.cf] owner changed to 0
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/master.cf] group changed to 0
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/master.cf] mode changed to 644
 [2012-11-01T19:22:32+09:00] INFO: template[/etc/postfix/master.cf] not queuing delayed action restart on service[postfix] (delayed), as it's already been queued
 [2012-11-01T19:22:32+09:00] INFO: Processing service[postfix] action start (postfix::default line 60)
 [2012-11-01T19:22:34+09:00] INFO: template[/etc/postfix/main.cf] sending restart action to service[postfix] (delayed)
 [2012-11-01T19:22:34+09:00] INFO: Processing service[postfix] action restart (postfix::default line 32)
 [2012-11-01T19:22:34+09:00] INFO: service[postfix] restarted
 [2012-11-01T19:22:34+09:00] INFO: Chef Run complete in 19.260800862 seconds
 [2012-11-01T19:22:34+09:00] INFO: Running report handlers
 [2012-11-01T19:22:34+09:00] INFO: Report handlers complete
 root@sandbox:~# 

実行されたことを実際に確認する。

.. code-block:: console

 root@sandbox:~# ls -la /etc/chef/
 total 16
 drwxr-xr-x  2 root root 4096 Nov  1 19:19 .
 drwxr-xr-x 90 root root 4096 Nov  1 19:19 ..
 -rw-------  1 root root 1675 Nov  1 19:15 client.pem
 -rw-r--r--  1 root root  205 Nov  1 19:12 client.rb
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# dpkg -l fail2ban
 Desired=Unknown/Install/Remove/Purge/Hold
 | Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
 |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
 ||/ Name           Version        Description
 +++-==============-==============-============================================
 ii  fail2ban       0.8.4-3+squeez bans IPs that cause multiple authentication 
 root@sandbox:~# 
 root@sandbox:~# ps auxwwwf | grep '[ f]ail2ban'
 root     16653  2.0  0.0  47800  6468 ?        Sl   19:21   0:00 /usr/bin/python /usr/bin/fail2ban-server -b -s /tmp/fail2ban.sock
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# dpkg -l logwatch
 Desired=Unknown/Install/Remove/Purge/Hold
 | Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
 |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
 ||/ Name           Version        Description
 +++-==============-==============-============================================
 ii  logwatch       7.3.6.cvs20090 log analyser with nice output written in Per
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# dpkg -l postfix
 Desired=Unknown/Install/Remove/Purge/Hold
 | Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
 |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
 ||/ Name           Version        Description
 +++-==============-==============-============================================
 ii  postfix        2.7.1-1+squeez High-performance mail transport agent
 root@sandbox:~# 
 root@sandbox:~# ps auxwwwf | grep '[ p]ostfix'
 root     17673  0.0  0.0  37168  2400 ?        Ss   19:22   0:00 /usr/lib/postfix/master
 postfix  17674  0.0  0.0  39232  2308 ?        S    19:22   0:00  \_ pickup -l -t fifo -u
 postfix  17675  0.0  0.0  39284  2356 ?        S    19:22   0:00  \_ qmgr -l -t fifo -u
 root@sandbox:~# 

LVM の設定
----------

.. code-block:: console

 root@sandbox:~# fdisk /dev/sda
 
 The device presents a logical sector size that is smaller than
 the physical sector size. Aligning to a physical sector (or optimal
 I/O) size boundary is recommended, or performance may be impacted.
 
 WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
          switch off the mode (command 'c') and change display units to
          sectors (command 'u').
 
 Command (m for help): p
 
 Disk /dev/sda: 1000.2 GB, 1000204886016 bytes
 255 heads, 63 sectors/track, 121601 cylinders
 Units = cylinders of 16065 * 512 = 8225280 bytes
 Sector size (logical/physical): 512 bytes / 4096 bytes
 I/O size (minimum/optimal): 4096 bytes / 4096 bytes
 Disk identifier: 0x0008ccbd
 
    Device Boot      Start         End      Blocks   Id  System
 /dev/sda1               1          32      248832   83  Linux
 Partition 1 does not end on cylinder boundary.
 /dev/sda2              32        1247     9765888   83  Linux
 
 Command (m for help): n
 Command action
    e   extended
    p   primary partition (1-4)
 p
 Partition number (1-4): 3
 First cylinder (1247-121601, default 1247): 
 Using default value 1247
 Last cylinder, +cylinders or +size{K,M,G} (1247-121601, default 121601): 
 Using default value 121601
 
 Command (m for help): p
 
 Disk /dev/sda: 1000.2 GB, 1000204886016 bytes
 255 heads, 63 sectors/track, 121601 cylinders
 Units = cylinders of 16065 * 512 = 8225280 bytes
 Sector size (logical/physical): 512 bytes / 4096 bytes
 I/O size (minimum/optimal): 4096 bytes / 4096 bytes
 Disk identifier: 0x0008ccbd
 
   Device Boot      Start         End      Blocks   Id  System
 /dev/sda1               1          32      248832   83  Linux
 Partition 1 does not end on cylinder boundary.
 /dev/sda2              32        1247     9765888   83  Linux
 /dev/sda3            1247      121601   966744288+  83  Linux
 
 Command (m for help): w
 The partition table has been altered!
  
 Calling ioctl() to re-read partition table.
 
 WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
 The kernel still uses the old table. The new table will be used at
 the next reboot or after you run partprobe(8) or kpartx(8)
 Syncing disks.
 root@sandbox:~#

.. code-block:: console

 root@sandbox:~# ls -l /dev/sda*
 brw-rw---- 1 root disk 8, 0 Nov  2 14:06 /dev/sda
 brw-rw---- 1 root disk 8, 1 Nov  1 18:59 /dev/sda1
 brw-rw---- 1 root disk 8, 2 Nov  1 18:59 /dev/sda2
 root@sandbox:~# 

見えないので一旦 reboot。

.. code-block:: console

 root@sandbox:~# ls -l /dev/sda*
 brw-rw---- 1 root disk 8, 0 Nov  2 14:07 /dev/sda
 brw-rw---- 1 root disk 8, 1 Nov  2 14:07 /dev/sda1
 brw-rw---- 1 root disk 8, 2 Nov  2 14:07 /dev/sda2
 brw-rw---- 1 root disk 8, 3 Nov  2 14:07 /dev/sda3
 root@sandbox:~# 

見えるようになった。

.. code-block:: console
 
 root@sandbox:~# pvcreate /dev/sda3
   Physical volume "/dev/sda3" successfully created
 root@sandbox:~# 
 root@sandbox:~# vgcreate vg_sandbox /dev/sda3
   Volume group "vg_sandbox" successfully created
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# pvdisplay 
   --- Physical volume ---
   PV Name               /dev/sda3
   VG Name               vg_sandbox
   PV Size               921.96 GiB / not usable 2.22 MiB
   Allocatable           yes 
   PE Size               4.00 MiB
   Total PE              236021
   Free PE               236021
   Allocated PE          0
   PV UUID               tP2efi-ZgDl-P0lc-27FY-vv80-1C7N-2WqsMr
    
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# vgdisplay 
   --- Volume group ---
   VG Name               vg_sandbox
   System ID             
   Format                lvm2
   Metadata Areas        1
   Metadata Sequence No  1
   VG Access             read/write
   VG Status             resizable
   MAX LV                0
   Cur LV                0
   Open LV               0
   Max PV                0
   Cur PV                1
   Act PV                1
   VG Size               921.96 GiB
   PE Size               4.00 MiB
   Total PE              236021
   Alloc PE / Size       0 / 0   
   Free  PE / Size       236021 / 921.96 GiB
   VG UUID               8CxGAI-yUSi-iSwi-dblb-adVb-9auV-kIcOXu
    
 root@sandbox:~#

virt-manager で

編集 > Host Details > ストレージ > 左下の水色の十字アイコン

名前: vg_sandbox
Type: logical: LVM Volume Group

Target Path: /dev/vg_sandbox
Source Path: /dev/sda3

右下の New Volume で、KVM ゲストから利用できる LV を作成できる。

apache2 のインストールと初期設定
--------------------------------

- recipe[apache2]
- recipe[apache2::mod_proxy]
- recipe[apache2::mod_proxy_http]

Web UI で run_list に追加。

.. code-block:: console

 root@sandbox:~# /etc/init.d/chef-client restart
 [ ok ] Restarting chef-client: chef-client.
 root@sandbox:~#

jenkins-master に接続するための apache2 の設定
----------------------------------------------

Jenkins + bitbucket.org で Sphinx で作られた Web サイトを自動公開する
http://d.hatena.ne.jp/tk0miya/20111212/p2

.. code-block:: console

 root@sandbox:~# cat > /etc/apache2/sites-available/jenkins-master
 # 2012/11/02 d-higuchi
 
 ProxyRequests           Off
 ProxyPass               /jenkins        http://192.168.122.11:8080/jenkins
 ProxyPassReverse        /jenkins        http://192.168.122.11:8080/jenkins
 
 <Location /jenkins>
         order deny,allow
         deny from all
         allow from localhost
         allow from 219.117.239.160/255.255.255.224
         allow from .tyma.nt.ftth4.ppp.infoweb.ne.jp
         AuthUserFile    /etc/apache2/htpasswd.jenkins-master
         AuthName        jenkins-master
         AuthType        Basic
         Require         valid-user
 </Location>
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# ls -l /etc/apache2/sites-*/jenkins-master
 -rw-r--r-- 1 root root 407 Nov  2 15:46 /etc/apache2/sites-available/jenkins-master
 root@sandbox:~# a2ensite jenkins-master
 Enabling site jenkins-master.
 To activate the new configuration, you need to run:
   service apache2 reload
 root@sandbox:~# ls -l /etc/apache2/sites-*/jenkins-master
 -rw-r--r-- 1 root root 407 Nov  2 15:46 /etc/apache2/sites-available/jenkins-master
 lrwxrwxrwx 1 root root  33 Nov  2 15:46 /etc/apache2/sites-enabled/jenkins-master -> ../sites-available/jenkins-master
 root@sandbox:~#

.. code-block:: console

 root@sandbox:~# /etc/init.d/apache2 restart
 [ ok ] Restarting web server: apache2 ... waiting .
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# htpasswd -c /etc/apache2/htpasswd.jenkins-master jenkins
 New password: 
 Re-type new password: 
 Adding password for user jenkins
 root@sandbox:~#

jenkins-master の sphinx ディレクトリに接続するための apache2 の設定
--------------------------------------------------------------------

.. code-block:: console

 root@sandbox:~# cat > /etc/apache2/sites-available/jenkins-master-sphinx 
 # 2012/11/05 d-higuchi
 
 ProxyRequests		Off
 ProxyPass		/sphinx		http://192.168.122.11/sphinx
 ProxyPassReverse	/sphinx		http://192.168.122.11/sphinx
 
 <Location /sphinx>
 	order deny,allow
 	deny from all
 	allow from localhost
 	# CL AKB
 	allow from 219.117.239.160/27
 	allow from 192.168.2.0/24
 	# d-higuchi
 	allow from .tyma.nt.ftth4.ppp.infoweb.ne.jp
 	# j-hotta
 	allow from 221.249.136.50/29
 	# y-uemura
 	allow from 124.35.220.7
 	AuthUserFile	/etc/apache2/htpasswd.jenkins-master
 	AuthName	jenkins-master
 	AuthType	Basic
 	Require		valid-user
 </Location>
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# a2ensite jenkins-master-sphinx
 Enabling site jenkins-master-sphinx.
 To activate the new configuration, you need to run:
   service apache2 reload
 root@sandbox:~# /etc/init.d/apache2 restart
 [ ok ] Restarting web server: apache2 ... waiting .
 root@sandbox:~# 

.. note::

 cookbook 管理が望ましい(TODO: 2012/11/05)

NTP の設定
----------

run_list に以下を追加するだけでOK

- recipe[ntp]

Chef クライアントの設定
-----------------------

デフォルトでは 443/tcp にアクセスするので変更する。

.. code-block:: console

 root@sandbox:~# chef-client
 [2012-11-14T16:47:59+09:00] INFO: *** Chef 10.16.2 ***
        :
        :
        :
 [2012-11-14T16:47:59+09:00] FATAL: Stacktrace dumped to /var/chef/cache/chef-stacktrace.out
 [2012-11-14T16:47:59+09:00] FATAL: Net::HTTPServerException: 403 "Forbidden"
 root@sandbox:~#

/etc/chef/client.rb の chef_server_url のポートを変更する。

.. code-block:: console

 root@sandbox:~# cp -a /etc/chef/client.rb /etc/chef/client.rb.orig
 root@sandbox:~# vi /etc/chef/client.rb
 root@sandbox:~# diff -u /etc/chef/client.rb.orig /etc/chef/client.rb
 --- /etc/chef/client.rb.orig    2012-11-01 19:12:45.000000000 +0900
 +++ /etc/chef/client.rb 2012-11-14 16:48:37.000000000 +0900
 @@ -1,5 +1,5 @@
  log_level              :info
  log_location           STDOUT
 -chef_server_url                "https://219.117.239.177/organizations/kanrinmaru"
 +chef_server_url                "https://219.117.239.177:8443/organizations/kanrinmaru"
  validation_key         "/etc/chef/kanrinmaru-validator.pem"
  validation_client_name "kanrinmaru-validator"
 root@sandbox:~#

アクセスできるようになった。

.. code-block:: console

 root@sandbox:~# chef-client
 [2012-11-14T16:48:53+09:00] INFO: *** Chef 10.16.2 ***
 [2012-11-14T16:48:53+09:00] INFO: Run List is [recipe[chef-client::delete_validation], recipe[fail2ban], recipe[logwatch], recipe[postfix], recipe[apache2], recipe[apache2::mod_proxy], recipe[apache2::mod_proxy_http], recipe[ntp]]
        :
        :
        :
 root@sandbox:~#

redmine に接続するための apache2 の設定
---------------------------------------

.. code-block:: console

 root@sandbox:~# cat > /etc/apache2/sites-available/redmine
 # 2012/11/19 d-higuchi

 ProxyRequests		Off
 ProxyPass		/redmine	http://192.168.122.21/redmine
 ProxyPassReverse	/redmine	http://192.168.122.21/redmine

 <Location /redmine>
	order deny,allow
	deny from all
	allow from localhost
	# CL AKB
	allow from 219.117.239.160/27
	allow from 192.168.2.0/24
	# d-higuchi
	allow from .tyma.nt.ftth4.ppp.infoweb.ne.jp
	# j-hotta
	allow from 221.249.136.50/29
	# y-uemura
	allow from 124.35.220.7
	AuthUserFile	/etc/apache2/htpasswd.redmine
	AuthName	redmine
	AuthType	Basic
	Require		valid-user
 </Location>
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# ls -l /etc/apache2/sites-*/redmine
 -rw-r--r-- 1 root root 538 Nov 19 16:16 /etc/apache2/sites-available/redmine
 root@sandbox:~# 

 root@sandbox:~# a2ensite redmine
 Enabling site redmine.
 To activate the new configuration, you need to run:
   service apache2 reload
 root@sandbox:~# 

 root@sandbox:~# ls -l /etc/apache2/sites-*/redmine
 -rw-r--r-- 1 root root 538 Nov 19 16:16 /etc/apache2/sites-available/redmine
 lrwxrwxrwx 1 root root  26 Nov 19 16:17 /etc/apache2/sites-enabled/redmine -> ../sites-available/redmine
 root@sandbox:~#

.. code-block:: console

 root@sandbox:~# /etc/init.d/apache2 restart
 [ ok ] Restarting web server: apache2 ... waiting .
 root@sandbox:~# 

.. code-block:: console

 root@sandbox:~# htpasswd -c /etc/apache2/htpasswd.redmine redmine
 New password: 
 Re-type new password: 
 Adding password for user redmine
 root@sandbox:~# 

jenkins-master の rabbit ディレクトリに接続するための apache2 の設定
--------------------------------------------------------------------

.. code-block:: console

 root@sandbox:~# vi /etc/apache2/sites-available/jenkins-master-rabbit
 # 2012/11/21 d-higuchi

 ProxyRequests           Off
 ProxyPass               /rabbit         http://192.168.122.11/rabbit
 ProxyPassReverse        /rabbit         http://192.168.122.11/rabbit

 <Location /rabbit>
        order deny,allow
        deny from all
        allow from localhost
        # CL AKB
        allow from 219.117.239.160/27
        allow from 192.168.2.0/24
        # d-higuchi
        allow from .tyma.nt.ftth4.ppp.infoweb.ne.jp
        allow from .tyma.nt.ftth4.ppp.infoweb.ne.jp
        # j-hotta
        allow from 221.249.136.50/29
        # y-uemura
        allow from 124.35.220.7
        AuthUserFile    /etc/apache2/htpasswd.jenkins-master
        AuthName        jenkins-master
        AuthType        Basic
        Require         valid-user
 </Location>
 root@sandbox:~#

.. code-block:: console

 root@sandbox:~# a2ensite jenkins-master-rabbit
 Enabling site jenkins-master-rabbit.
 To activate the new configuration, you need to run:
   service apache2 reload
 root@sandbox:~# /etc/init.d/apache2 reload
 [ ok ] Reloading web server config: apache2.
 root@sandbox:~#

LVM バックアップのテスト
------------------------

バックアップ HDD にパーティション作成。

.. code-block:: console

 root@sandbox:~# fdisk /dev/sdb
	:
 root@sandbox:~# fdisk -l /dev/sdb
 
 Disk /dev/sdb: 2000.4 GB, 2000398934016 bytes
 81 heads, 63 sectors/track, 765633 cylinders, total 3907029168 sectors
 Units = sectors of 1 * 512 = 512 bytes
 Sector size (logical/physical): 512 bytes / 4096 bytes
 I/O size (minimum/optimal): 4096 bytes / 4096 bytes
 Disk identifier: 0x3814e741
 
    Device Boot      Start         End      Blocks   Id  System
 /dev/sdb1            2048  3907029167  1953513560   83  Linux
 root@sandbox:~# 

バックアップパーティションにファイルシステム作成。

.. code-block:: console

 root@sandbox:~# mkfs.ext4 -m0 /dev/sdb1 
	:
 root@sandbox:~#
 
バックアップファイルシステムを mount。

.. code-block:: console

 root@sandbox:~# mkdir /backup
 root@sandbox:~# 
 
 root@sandbox:~# cp -a /etc/fstab /etc/fstab.2012-1127
 root@sandbox:~# vi /etc/fstab
 root@sandbox:~# diff -u /etc/fstab.2012-1127 /etc/fstab
 --- /etc/fstab.2012-1127	2012-11-01 18:02:10.000000000 +0900
 +++ /etc/fstab	2012-11-27 13:18:31.000000000 +0900
 @@ -11,3 +11,7 @@
  # /boot was on /dev/sda1 during installation
  UUID=fd36e6b3-e3b6-4698-9d97-af60bd25ba33 /boot           ext3    defaults        0       2
  /dev/scd0       /media/cdrom0   udf,iso9660 user,noauto     0       0
 +
 +# 2012/11/27 d-higuchi add
 +/dev/sdb1 /backup ext4 defaults,relatime 0 0
 +#
 root@sandbox:~# 

 root@sandbox:~# mount /backup/
 root@sandbox:~# 

 root@sandbox:~# df -h /backup 
 Filesystem      Size  Used Avail Use% Mounted on
 /dev/sdb1       1.9T   28G  1.8T   2% /backup
 root@sandbox:~# 

スナップショットの作成。

.. code-block:: console

 root@sandbox:~# lvcreate -s -L 10G -n jenkins-master_snapshot /dev/vg_sandbox/jenkins-master
 File descriptor 3 (/usr/share/bash-completion/completions) leaked on lvcreate invocation. Parent PID 7397: -su
   Logical volume "jenkins-master_snapshot" created
 root@sandbox:~# 

パーティションのマッピング。

.. code-block:: console

 root@sandbox:~# kpartx -av /dev/vg_sandbox/jenkins-master_snapshot 
 add map vg_sandbox-jenkins--master_snapshot1 (254:5): 0 497664 linear /dev/vg_sandbox/jenkins-master_snapshot 2048
 add map vg_sandbox-jenkins--master_snapshot2 (254:6): 0 19978240 linear /dev/vg_sandbox/jenkins-master_snapshot 499712
 root@sandbox:~# 

マウント。

.. code-block:: console

 root@sandbox:~# mount -o ro /dev/mapper/vg_sandbox-jenkins--master_snapshot2 /mnt
 root@sandbox:~# 

バックアップ。

.. code-block:: console

 root@sandbox:~# rsync -av --delete /mnt/ /backup/jenkins-master
	:
	:
	:
 sent 4698592884 bytes  received 2370443 bytes  23802345.96 bytes/sec
 total size is 4689811894  speedup is 1.00
 root@sandbox:~# 

アンマウント + アンマッピング + スナップショット削除。

.. code-block:: console

 root@sandbox:~# umount /mnt 
 root@sandbox:~# 

 root@sandbox:~# kpartx -d /dev/vg_sandbox/jenkins-master_snapshot
 root@sandbox:~# 

 root@sandbox:~# lvremove -vf /dev/vg_sandbox/jenkins-master_snapshot 
 File descriptor 3 (/usr/share/bash-completion/completions) leaked on lvremove invocation. Parent PID 7397: -su
    Using logical volume(s) on command line
    Archiving volume group "vg_sandbox" metadata (seqno 10).
    Removing snapshot jenkins-master_snapshot
    Found volume group "vg_sandbox"
    Found volume group "vg_sandbox"
    Loading vg_sandbox-jenkins--master table (254:0)
    Loading vg_sandbox-jenkins--master_snapshot table (254:2)
  /sbin/dmeventd: stat failed: No such file or directory
    vg_sandbox/snapshot0 already not monitored.
    Suspending vg_sandbox-jenkins--master (254:0) with device flush
    Suspending vg_sandbox-jenkins--master_snapshot (254:2) with device flush
    Suspending vg_sandbox-jenkins--master-real (254:3) with device flush
    Suspending vg_sandbox-jenkins--master_snapshot-cow (254:4) with device flush
    Found volume group "vg_sandbox"
    Resuming vg_sandbox-jenkins--master_snapshot-cow (254:4)
    Resuming vg_sandbox-jenkins--master-real (254:3)
    Resuming vg_sandbox-jenkins--master_snapshot (254:2)
    Removing vg_sandbox-jenkins--master_snapshot-cow (254:4)
    Found volume group "vg_sandbox"
    Resuming vg_sandbox-jenkins--master (254:0)
    Removing vg_sandbox-jenkins--master-real (254:3)
    Found volume group "vg_sandbox"
    Removing vg_sandbox-jenkins--master_snapshot (254:2)
    Releasing logical volume "jenkins-master_snapshot"
    Creating volume group backup "/etc/lvm/backup/vg_sandbox" (seqno 12).
  Logical volume "jenkins-master_snapshot" successfully removed
 root@sandbox:~# 

LVM バックアップの実設定
------------------------

.. code-block:: console

 root@sandbox:~# vi /etc/cron.daily/lvm-backup
 #!/bin/sh

 # 2012/11/27 d-higuchi

 TARGET="jenkins-master redmine"

 for i in $TARGET;do
	echo
	echo "----- BEGIN ${i} -----"
	echo

	# create snapshot
	lvcreate -s -L 10G -n ${i}_snapshot /dev/vg_sandbox/${i} || exit 1
	# mapping
	kpartx -av /dev/vg_sandbox/${i}_snapshot || exit 1
	# make mount point
	mkdir -p /snapshot/${i}
	# mount
	j=`echo "${i}" | sed -e 's/-/--/'`
	mount -o ro /dev/mapper/vg_sandbox-${j}_snapshot2 /snapshot/${i} || exit 1
	# backup
	rsync -av --delete /snapshot/${i}/ /backup/${i}
	# umount
	umount /snapshot/${i}
	# unmapping
	kpartx -d /dev/vg_sandbox/${i}_snapshot
	# remove snapshot
	lvremove -f /dev/vg_sandbox/${i}_snapshot

	echo
	echo "----- END ${i} -----"
	echo
 done

 exit 0

 # [EOF]
 root@sandbox:~# 

..
 [EOF]
