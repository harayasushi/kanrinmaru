sandbox.akb.creationline.com (219.117.239.188)
==============================================

FQDN の設定
-----------

.. code-block:: console

 root@sandbox:~# hostname --fqdn
 sandbox.akb.creationline.com
 root@sandbox:~# 

Chef Client のインストールと Chef Private Server への登録
---------------------------------------------------------

PrimeDrive /咸臨丸@(uemura)/cl-chef-priv.sh を取得して実行する。
Chef Private Server に登録されたことを Web UI で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[fail2ban]
- recipe[logwatch]
- recipe[postfix]
- recipe[ntp]

Web UI で行う。

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
