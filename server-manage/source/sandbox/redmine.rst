redmine.akb-sb.creationline (192.168.122.21)
============================================

FQDN の設定
-----------

.. code-block:: console

 root@redmine:~# hostname --fqdn
 redmine.akb-sb.creationline.com
 root@redmine:~# 

設定済。

chef omnibus インストール
-------------------------

.. code-block:: console

 root@redmine:~# curl -L http://opscode.com/chef/install.sh | bash
	:
 Downloading Chef  for ubuntu...
 Installing Chef 
 Selecting previously unselected package chef.
 (データベースを読み込んでいます ... 現在 75084 個のファイルとディレクトリがインストールされています。)
 (.../tmp.LW7XCW32/chef__amd64.deb から) chef を展開しています...
 chef (10.16.2-1.ubuntu.11.04) を設定しています ...
 Thank you for installing Chef!
 
 root@redmine:~# 

設定ファイルの設置
------------------

.. code-block:: console

 root@redmine:~# mkdir /etc/chef
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# cat > /etc/chef/client.rb
 log_level              :info
 log_location           STDOUT
 chef_server_url        "https://219.117.239.177:8443/organizations/kanrinmaru"
 validation_key         "/etc/chef/kanrinmaru-validator.pem"
 validation_client_name "kanrinmaru-validator"
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# cat > /etc/chef/kanrinmaru-validator.pem
 -----BEGIN RSA PRIVATE KEY-----
 MIIEpAIBAAKCAQEArCDgwHiNeXifjnxYwaiM5n7mC47n7v5rqUy9rmt769ndyE7O
 
 	(中略)
 
 wPa3z7UMoyMm0aBH4GBw0P23/E7usCBYr43RlDJU4g1bT/Fy3UX8OQ==
 -----END RSA PRIVATE KEY-----
 root@redmine:~# 

chef-client を実行
------------------

.. code-block:: console

 root@redmine:~# chef-client 
 [2012-11-19T15:19:25+09:00] INFO: *** Chef 10.16.2 ***
 [2012-11-19T15:19:25+09:00] INFO: Client key /etc/chef/client.pem is not present - registering
 [2012-11-19T15:19:26+09:00] INFO: Run List is []
 [2012-11-19T15:19:26+09:00] INFO: Run List expands to []
 [2012-11-19T15:19:26+09:00] INFO: Starting Chef Run for redmine.akb-sb.creationline.com
 [2012-11-19T15:19:26+09:00] INFO: Running start handlers
 [2012-11-19T15:19:26+09:00] INFO: Start handlers complete.
 [2012-11-19T15:19:26+09:00] INFO: Loading cookbooks []
 [2012-11-19T15:19:26+09:00] WARN: Node redmine.akb-sb.creationline.com has an empty run list.
 [2012-11-19T15:19:27+09:00] INFO: Chef Run complete in 0.261995852 seconds
 [2012-11-19T15:19:27+09:00] INFO: Running report handlers
 [2012-11-19T15:19:27+09:00] INFO: Report handlers complete
 root@redmine:~# 

Chef Server に登録されたことを web で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[ntp]
- recipe[postfix]
- recipe[logwatch]

Web UI で行う。

chef-client を initscripts に登録
---------------------------------

.. code-block:: console

 root@redmine:~# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.16.2/distro/debian/etc/default/chef-client /etc/default/
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.16.2/distro/debian/etc/init.d/chef-client /etc/init.d/
 root@redmine:~# chmod +x /etc/init.d/chef-client 
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# update-rc.d chef-client defaults 99
  Adding system startup for /etc/init.d/chef-client ...
    /etc/rc0.d/K99chef-client -> ../init.d/chef-client
    /etc/rc1.d/K99chef-client -> ../init.d/chef-client
    /etc/rc6.d/K99chef-client -> ../init.d/chef-client
    /etc/rc2.d/S99chef-client -> ../init.d/chef-client
    /etc/rc3.d/S99chef-client -> ../init.d/chef-client
    /etc/rc4.d/S99chef-client -> ../init.d/chef-client
    /etc/rc5.d/S99chef-client -> ../init.d/chef-client
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# ls -l /etc/*.d/*chef-client*
 -rwxr-xr-x 1 root root 4538 11月 19 15:20 /etc/init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月 19 15:20 /etc/rc0.d/K99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月 19 15:20 /etc/rc1.d/K99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月 19 15:20 /etc/rc2.d/S99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月 19 15:20 /etc/rc3.d/S99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月 19 15:20 /etc/rc4.d/S99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月 19 15:20 /etc/rc5.d/S99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月 19 15:20 /etc/rc6.d/K99chef-client -> ../init.d/chef-client
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# mkdir /var/log/chef
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# /etc/init.d/chef-client start
  * Starting chef-client  chef-client                                     [ OK ] 
 root@redmine:~#

.. code-block:: console

 root@redmine:~# tail -f /var/log/chef/client.log 
 # Logfile created on 2012-11-19 15:21:00 +0900 by logger.rb/31641
 [2012-11-19T15:21:00+09:00] INFO: Daemonizing..
 [2012-11-19T15:21:00+09:00] INFO: Forked, in 1489. Privileges: 0 0
 [2012-11-19T15:21:16+09:00] INFO: *** Chef 10.16.2 ***
 [2012-11-19T15:21:17+09:00] INFO: Run List is [recipe[chef-client::delete_validation], recipe[ntp], recipe[logwatch], recipe[postfix]]
 [2012-11-19T15:21:17+09:00] INFO: Run List expands to [chef-client::delete_validation, ntp, logwatch, postfix]
  	:
 	:
 	:
 [2012-11-19T15:21:58+09:00] INFO: Chef Run complete in 41.45904606 seconds
 [2012-11-19T15:21:58+09:00] INFO: Running report handlers
 [2012-11-19T15:21:58+09:00] INFO: Report handlers complete
 root@redmine:~# 

実行されたことを実際に確認する。

.. code-block:: console

 root@redmine:~# ls -la /etc/chef/
 合計 16
 drwxr-xr-x  2 root root 4096 11月 19 15:21 .
 drwxr-xr-x 89 root root 4096 11月 19 15:21 ..
 -rw-------  1 root root 1679 11月 19 15:19 client.pem
 -rw-r--r--  1 root root  210 11月 19 15:18 client.rb
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# dpkg -l ntp
 Desired=Unknown/Install/Remove/Purge/Hold
 | Status=Not/Inst/Cfg-files/Unpacked/Failed-cfg/Half-inst/trig-aWait/Trig-pend
 |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
 ||/ 名前         バージョ   説明
 +++-==============-==============-============================================
 ii  ntp            1:4.2.6.p3+dfs Network Time Protocol daemon and utility pro
 root@redmine:~# ps auxwwwf | grep '[ n]tp'
 ntp      11834  0.0  0.0  37708  2112 ?        Ss   15:21   0:00 /usr/sbin/ntpd -p /var/run/ntpd.pid -g -u 106:113
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# dpkg -l postfix
 Desired=Unknown/Install/Remove/Purge/Hold
 | Status=Not/Inst/Cfg-files/Unpacked/Failed-cfg/Half-inst/trig-aWait/Trig-pend
 |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
 ||/ 名前         バージョ   説明
 +++-==============-==============-============================================
 ii  postfix        2.9.3-2~12.04. High-performance mail transport agent
 root@redmine:~# ps auxwwwf | grep '[ p]ostfix'
 root     11929  0.0  0.0  25108  1532 ?        Ss   15:21   0:00 /usr/lib/postfix/master
 postfix  11930  0.0  0.0  27172  1460 ?        S    15:21   0:00  \_ pickup -l -t fifo -u
 postfix  11931  0.0  0.0  27224  1500 ?        S    15:21   0:00  \_ qmgr -l -t fifo -u
 root@redmine:~# 

.. code-block:: console

 root@redmine:~# dpkg -l logwatch
 Desired=Unknown/Install/Remove/Purge/Hold
 | Status=Not/Inst/Cfg-files/Unpacked/Failed-cfg/Half-inst/trig-aWait/Trig-pend
 |/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
 ||/ 名前         バージョ   説明
 +++-==============-==============-============================================
 ii  logwatch       7.4.0+svn20111 log analyser with nice output written in Per
 root@redmine:~# 

aliases の設定
--------------

.. code-block:: console

 root@redmine:~# cp -a /etc/aliases /etc/aliases.2012-1119
 root@redmine:~# vi /etc/aliases
 root@redmine:~# diff -u /etc/aliases.2012-1119 /etc/aliases
 --- /etc/aliases.2012-1119	2012-11-19 15:21:53.312177824 +0900
 +++ /etc/aliases	2012-11-19 15:23:36.318338684 +0900
 @@ -1,2 +1,6 @@
  # See man 5 aliases for format
  postmaster:    root
 +
 +# 2012/11/19 d-higuchi add
 +root: solution@creationline.com
 +#
 root@redmine:~# newaliases 
 root@redmine:~# 

redmine のインストール
----------------------

バックエンドをローカルの MySQL、フロントエンドをローカルの apache2 とした redmine をインストールする。

https://github.com/cl-lab-k/cl-redmine

- recipe[cl-redmine]
- recipe[cl-redmine::plugin_views_revisions]
- recipe[cl-redmine::xls_export]
- recipe[cl-redmine::markdown_extra_formatter]
- recipe[cl-redmine::restructuredtext_formatter]
- recipe[cl-redmine::require_closing_note]
- recipe[cl-redmine::lightbox]
- recipe[cl-redmine::send_reminders]

redmine 設定メモ
----------------

Debconf の事前設定(preceeding)では以下が行われる。

- redmine/instances/default のデータベースを dbconfig-common で設定しますか？
	- [はい]
- redmine/instances/default が使うデータベースの種類
	- [mysql]
- データベースの管理権限を持つユーザのパスワード
	- /etc/mysql/grants.sql, /var/cache/local/preseeding/mysql-server.seed に記載あり
- redmine/instances/default 用の MySQL アプリケーションパスワード
	- [空白でランダムパスワードを生成する]

メール通知のためのconfiguration.ymlの設定
http://redmine.jp/faq/general/mail_notification/

redmine のユーザ登録
--------------------

まず、右上の「登録する」リンクからユーザ登録画面に移動する。
必要事項を埋めてボタンを押して登録する。
ただし、すぐに使えるようにはならず、管理者の承認が必要。

右上の「ログイン」から管理者でログインする。ID/PW は PrimeDrive を参照のこと。
左上の「管理」から「ユーザ」を選択し、フィルタのステータスを「すべて」にする。
そうすると仮登録になっているユーザも表示されるので、承認することでそのユーザがログインできるようになる。

..
 [EOF]