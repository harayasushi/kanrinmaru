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
- recipe[cl-etc-common::aliases]
- recipe[cl-etc-common::hostname]
- recipe[cl-etc-common::hosts-access]

Web UI で行う。

apache2 のインストールとリバースプロキシ設定
--------------------------------------------

- recipe[cl-reverse-proxy]

Web UI で run_list に追加。

なお、htpasswd ファイルは Cookbook では作成しない。

.. code-block:: console

 root@sandbox:~# htpasswd -c /etc/apache2/htpasswd.jenkins-master jenkins
 New password: 
 Re-type new password: 
 Adding password for user jenkins
 root@sandbox:~#

 root@sandbox:~# htpasswd -c /etc/apache2/htpasswd.redmine redmine
 New password: 
 Re-type new password: 
 Adding password for user redmine
 root@sandbox:~# 

以下の公開ディレクトリの ID/PW は都度変更すること。

.. code-block:: console

 root@sandbox:~# htpasswd -c /etc/apache2/htpasswd.public-storage public
 New password: 
 Re-type new password: 
 Adding password for user public
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
