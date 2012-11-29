redmine.akb-sb.creationline (192.168.122.21)
============================================

FQDN の設定
-----------

.. code-block:: console

 root@redmine:~# hostname --fqdn
 redmine.akb-sb.creationline.com
 root@redmine:~# 

Chef Client のインストールと Chef Private Server への登録
---------------------------------------------------------

PrimeDrive /咸臨丸@(uemura)/cl-chef-priv.sh を取得して実行する。
Chef Private Server に登録されたことを Web UI で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[ntp]
- recipe[postfix]
- recipe[logwatch]

Web UI で行う。

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
