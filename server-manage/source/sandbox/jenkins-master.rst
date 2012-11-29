jenkins-master.akb-sb.creationline (192.168.122.11)
===================================================

FQDN の設定
-----------

.. code-block:: console

 root@jenkins-master:~# hostname --fqdn
 jenkins-master.akb-sb.creationline.com
 root@jenkins-master:~# 

Chef Client のインストールと Chef Private Server への登録
---------------------------------------------------------

PrimeDrive /咸臨丸@(uemura)/cl-chef-priv.sh を取得して実行する。
Chef Private Server に登録されたことを Web UI で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[logwatch]
- recipe[postfix]
- recipe[ntp]

Web UI で行う。

aliases の設定
--------------

.. code-block:: console

 root@jenkins-master:~# cp -a /etc/aliases /etc/aliases.2012-1102
 root@jenkins-master:~# vi /etc/aliases
 root@jenkins-master:~# diff -u /etc/aliases.2012-1102 /etc/aliases
 --- /etc/aliases.2012-1102	2012-11-02 15:27:44.535908703 +0900
 +++ /etc/aliases	2012-11-02 15:30:02.871899935 +0900
 @@ -1,2 +1,5 @@
  # See man 5 aliases for format
  postmaster:    root
 +# 2012/11/02 d-higuchi add
 +root:	solution@creationline.com
 +#
 root@jenkins-master:~# newaliases 
 root@jenkins-master:~# 

jenkins のインストール
----------------------

https://github.com/cl-lab-k/cl-jenkins

- recipe[apache2]
- recipe[cl-jenkins]
- recipe[cl-jenkins::sphinx]
- recipe[cl-jenkins::texlive]
- recipe[cl-jenkins::rabbit]

jenkins 設定メモ
----------------

Jenkins + bitbucket.org で Sphinx で作られた Web サイトを自動公開する
http://d.hatena.ne.jp/tk0miya/20111212/p2

Ubuntu 12.04 の TeXLive は古いので UTF-8 が通らず、日本語 PDF が生成できない。
そのため Ubuntu 12.10 から TeXLive を借りてくる(APT-Pinning)。
もしかしたら Ubuntu 12.10 に完全に上げてしまったほうがいいのかも？

bitbucket.org との連携
----------------------

意外と簡単。Jenkinsとbitbucket(Git)を連携する７ステップ
http://d.hatena.ne.jp/gungnir_odin/20120922/1348281247

.. code-block:: console

 jenkins@jenkins-master:~$ git config --global user.email 'jenkins@jenkins-master.akb-sb.creationline.com'
 jenkins@jenkins-master:~$ git config --global user.name 'jenkins'
 jenkins@jenkins-master:~$ 

jenkins ユーザに ssh キーペアの作成、パスフレーズは空で。

.. code-block:: console

 jenkins@jenkins-master:~$ ssh-keygen 
 Generating public/private rsa key pair.
 Enter file in which to save the key (/var/lib/jenkins/.ssh/id_rsa): /var/lib/jenkins/.ssh/id_rsa_bitbucket
 Created directory '/var/lib/jenkins/.ssh'.
 Enter passphrase (empty for no passphrase): 
 Enter same passphrase again: 
 Your identification has been saved in /var/lib/jenkins/.ssh/id_rsa_bitbucket.
 Your public key has been saved in /var/lib/jenkins/.ssh/id_rsa_bitbucket.pub.
 The key fingerprint is:
 42:d0:1a:36:cb:6c:bb:fc:28:91:42:e2:1e:d5:92:b1 jenkins@jenkins-master
 The key's randomart image is:

	:

 jenkins@jenkins-master:~$ 
 jenkins@jenkins-master:~$ ls -la .ssh/
 合計 16
 drwx------ 2 jenkins jenkins 4096 11月  5 11:49 .
 drwxr-xr-x 3 jenkins jenkins 4096 11月  5 11:49 ..
 -rw------- 1 jenkins jenkins 1675 11月  5 11:49 id_rsa_bitbucket
 -rw-r--r-- 1 jenkins jenkins  406 11月  5 11:49 id_rsa_bitbucket.pub
 jenkins@jenkins-master:~$ 

設定ファイルの作成。

.. code-block:: console

 jenkins@jenkins-master:~$ cat > .ssh/config
 Host bitbucket.org
 	Port 22
 	Hostname bitbucket.org
 	IdentityFile ~/.ssh/id_rsa_bitbucket
 	TCPKeepAlive yes
 	IdentitiesOnly yes
 jenkins@jenkins-master:~$ 

公開鍵を bitbucket.org (d_higuchi) に登録する。

Jenkins の Web UI にて、

Jenkinsの管理 > プラグインの管理 > 利用可能 > Git Plugin > インストール > インストール完了後、ジョブがなければJenkinsを再起動する

インストールが完了したら、

.. code-block:: console

 新規ジョブの作成
	ジョブ名: chef-training-p
	フリースタイル・プロジェクトのビルド

	ソースコード管理システム: git
	Repository URL: ssh://git@bitbucket.org/j_hotta/chef-training-p.git

	SCMをポーリング: on
	スケジュール: \*/10 \* \* \* \*

	ビルド手順の追加
	シェルの実行:
		make clean html
		rsync -av --delete /var/lib/jenkins/jobs/chef-traning-p/workspace/build/html/ /var/www/sphinx/chef-training-p

	E-mail通知: on
	宛先: solution@creationline.com
	不安定ビルドも逐一メールを送信: on
	ビルドを壊した個人にも別途メールを送信: off

..
 [EOF]
