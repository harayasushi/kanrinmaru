kanrinmaru02.akb.creationline.com (219.117.239.178)
===================================================

OS 基本情報
-----------

- CentOS 6.3 (x86_64)
- 最小インストール
- パーティション設定: 自動
- on-premise enStratus chef-solo installer を実施済であること。

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

 enStratus が /etc/hosts を書き換えることに注意。

Chef Client のインストールと Chef Private Server への登録
---------------------------------------------------------

PrimeDrive /咸臨丸@(uemura)/cl-chef-priv.sh を取得して実行する。
Chef Private Server に登録されたことを Web UI で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[yum::epel]
- recipe[fail2ban]
- recipe[logwatch]
- recipe[postfix]
- recipe[ntp]
- recipe[cl-etc-common::aliases]
- recipe[cl-etc-common::hosts-access]

Web UI で行う。

ファイアウォールの設定
----------------------

run_list に以下を追加するだけでOK。

- recipe[lokkit::service_ssh]
- recipe[lokkit::https_tcp]

..
 [EOF]
