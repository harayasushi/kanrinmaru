kanrinmaru03.akb.creationline.com (219.117.239.179)
===================================================

OS 基本情報
-----------

- CentOS 6.3 (x86_64)
- 最小インストール
- パーティション設定: 自動

FQDN の設定
-----------

.. code-block:: console

 [root@kanrinmaru03 ~]# hostname --fqdn
 kanrinmaru03.akb.creationline.com
 [root@kanrinmaru03 ~]# 

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
- recipe[cl-etc-common::hostname]
- recipe[cl-etc-common::hosts-access]

Web UI で行う。

ファイアウォールの設定
----------------------

run_list に以下を追加する。

- recipe[lokkit::service_ssh]

..
 [EOF]
