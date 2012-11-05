jenkins-master.akb-sb.creationline (192.168.122.11)
===================================================

FQDN の設定
-----------

.. code-block:: console

 root@jenkins-master:~# hostname --fqdn
 jenkins-master.akb-sb.creationline.com
 root@jenkins-master:~# 

設定済。

chef omnibus インストール
-------------------------

.. code-block:: console

 root@jenkins-master:~# curl -L http://opscode.com/chef/install.sh | bash
 	:
 Downloading Chef  for ubuntu...
 Installing Chef 
 Selecting previously unselected package chef.
 (データベースを読み込んでいます ... 現在 53066 個のファイルとディレクトリがインストールされています。)
 (.../tmp.HSXO19Ah/chef__amd64.deb から) chef を展開しています...
 chef (10.16.2-1.ubuntu.11.04) を設定しています ...
 Thank you for installing Chef!
 root@jenkins-master:~# 

設定ファイルの設置
------------------

.. code-block:: console

 root@jenkins-master:~# mkdir /etc/chef
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# cat > /etc/chef/client.rb
 log_level		:info
 log_location		STDOUT
 chef_server_url	"https://219.117.239.177/organizations/kanrinmaru"
 validation_key		"/etc/chef/kanrinmaru-validator.pem"
 validation_client_name	"kanrinmaru-validator"
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# cat > /etc/chef/kanrinmaru-validator.pem
 -----BEGIN RSA PRIVATE KEY-----
 MIIEpAIBAAKCAQEArCDgwHiNeXifjnxYwaiM5n7mC47n7v5rqUy9rmt769ndyE7O
 
 	(中略)
 
 wPa3z7UMoyMm0aBH4GBw0P23/E7usCBYr43RlDJU4g1bT/Fy3UX8OQ==
 -----END RSA PRIVATE KEY-----
 root@jenkins-master:~# 

chef-client を実行
------------------

.. code-block:: console

 root@jenkins-master:~# chef-client
 [2012-11-02T15:24:02+09:00] INFO: *** Chef 10.16.2 ***
 [2012-11-02T15:24:02+09:00] INFO: Client key /etc/chef/client.pem is not present - registering
 [2012-11-02T15:24:03+09:00] INFO: Run List is []
 [2012-11-02T15:24:03+09:00] INFO: Run List expands to []
 [2012-11-02T15:24:03+09:00] INFO: Starting Chef Run for jenkins-master.akb-sb.creationline.com
 [2012-11-02T15:24:03+09:00] INFO: Running start handlers
 [2012-11-02T15:24:03+09:00] INFO: Start handlers complete.
 [2012-11-02T15:24:03+09:00] INFO: Loading cookbooks []
 [2012-11-02T15:24:03+09:00] WARN: Node jenkins-master.akb-sb.creationline.com has an empty run list.
 [2012-11-02T15:24:03+09:00] INFO: Chef Run complete in 0.207962253 seconds
 [2012-11-02T15:24:03+09:00] INFO: Running report handlers
 [2012-11-02T15:24:03+09:00] INFO: Report handlers complete
 root@jenkins-master:~# 

Chef Server に登録されたことを web で確認する。

run_list に追加
---------------

- recipe[chef-client::delete_validation]
- recipe[logwatch]
- recipe[postfix]

Web UI で行う。

chef-client を initscripts に登録
---------------------------------

.. code-block:: console

 root@jenkins-master:~# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.16.2/distro/debian/etc/default/chef-client /etc/default/
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# cp /opt/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.16.2/distro/debian/etc/init.d/chef-client /etc/init.d/
 root@jenkins-master:~# chmod +x /etc/init.d/chef-client
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# update-rc.d chef-client defaults 99
  Adding system startup for /etc/init.d/chef-client ...
    /etc/rc0.d/K99chef-client -> ../init.d/chef-client
    /etc/rc1.d/K99chef-client -> ../init.d/chef-client
    /etc/rc6.d/K99chef-client -> ../init.d/chef-client
    /etc/rc2.d/S99chef-client -> ../init.d/chef-client
    /etc/rc3.d/S99chef-client -> ../init.d/chef-client
    /etc/rc4.d/S99chef-client -> ../init.d/chef-client
    /etc/rc5.d/S99chef-client -> ../init.d/chef-client
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# ls -l /etc/*.d/*chef-client*
 -rwxr-xr-x 1 root root 4538 11月  2 15:25 /etc/init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月  2 15:25 /etc/rc0.d/K99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月  2 15:25 /etc/rc1.d/K99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月  2 15:25 /etc/rc2.d/S99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月  2 15:25 /etc/rc3.d/S99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月  2 15:25 /etc/rc4.d/S99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月  2 15:25 /etc/rc5.d/S99chef-client -> ../init.d/chef-client
 lrwxrwxrwx 1 root root   21 11月  2 15:25 /etc/rc6.d/K99chef-client -> ../init.d/chef-client
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# mkdir /var/log/chef
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# /etc/init.d/chef-client start
  * Starting chef-client  chef-client                                     [ OK ] 
 root@jenkins-master:~#

.. code-block:: console

 root@jenkins-master:~# tail -f /var/log/chef/client.log
 # Logfile created on 2012-11-02 15:27:09 +0900 by logger.rb/31641
 [2012-11-02T15:27:09+09:00] INFO: Daemonizing..
 [2012-11-02T15:27:09+09:00] INFO: Forked, in 1411. Privileges: 0 0
 [2012-11-02T15:27:11+09:00] INFO: *** Chef 10.16.2 ***
 [2012-11-02T15:27:11+09:00] INFO: Run List is [recipe[chef-client::delete_validation], recipe[logwatch], recipe[postfix]]
 [2012-11-02T15:27:11+09:00] INFO: Run List expands to [chef-client::delete_validation, logwatch, postfix]
 [2012-11-02T15:27:11+09:00] INFO: Starting Chef Run for jenkins-master.akb-sb.creationline.com
 [2012-11-02T15:27:11+09:00] INFO: Running start handlers
 [2012-11-02T15:27:11+09:00] INFO: Start handlers complete.
 [2012-11-02T15:27:12+09:00] INFO: Loading cookbooks [chef-client, logwatch, perl, postfix]
  	:
 	:
 	:
 [2012-11-02T15:27:14+09:00] INFO: Processing file[/etc/chef/kanrinmaru-validator.pem] action delete (chef-client::delete_validation line 21)
 [2012-11-02T15:27:14+09:00] INFO: file[/etc/chef/kanrinmaru-validator.pem] deleted file at /etc/chef/kanrinmaru-validator.pem
 	:
 	:
 	:
 [2012-11-02T15:27:32+09:00] INFO: Processing package[logwatch] action install (logwatch::default line 22)
 [2012-11-02T15:27:45+09:00] INFO: Processing template[/etc/logwatch/conf/logwatch.conf] action create (logwatch::default line 25)
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] updated content
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] owner changed to 0
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] group changed to 0
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/logwatch/conf/logwatch.conf] mode changed to 644
 [2012-11-02T15:27:45+09:00] INFO: Processing package[postfix] action install (postfix::default line 21)
 [2012-11-02T15:27:45+09:00] INFO: Processing service[postfix] action enable (postfix::default line 32)
 [2012-11-02T15:27:45+09:00] INFO: Processing template[/etc/postfix/main.cf] action create (postfix::default line 51)
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/main.cf] backed up to /var/chef/backup/etc/postfix/main.cf.chef-20121102152745
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/main.cf] updated content
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/main.cf] owner changed to 0
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/main.cf] group changed to 0
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/main.cf] mode changed to 644
 [2012-11-02T15:27:45+09:00] INFO: Processing template[/etc/postfix/master.cf] action create (postfix::default line 51)
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/master.cf] backed up to /var/chef/backup/etc/postfix/master.cf.chef-20121102152745
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/master.cf] updated content
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/master.cf] owner changed to 0
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/master.cf] group changed to 0
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/master.cf] mode changed to 644
 [2012-11-02T15:27:45+09:00] INFO: template[/etc/postfix/master.cf] not queuing delayed action restart on service[postfix] (delayed), as it's already been queued
 [2012-11-02T15:27:45+09:00] INFO: Processing service[postfix] action start (postfix::default line 60)
 [2012-11-02T15:27:47+09:00] INFO: template[/etc/postfix/main.cf] sending restart action to service[postfix] (delayed)
 [2012-11-02T15:27:47+09:00] INFO: Processing service[postfix] action restart (postfix::default line 32)
 [2012-11-02T15:27:47+09:00] INFO: service[postfix] restarted
 [2012-11-02T15:27:47+09:00] INFO: Chef Run complete in 35.663317125 seconds
 [2012-11-02T15:27:47+09:00] INFO: Running report handlers
 [2012-11-02T15:27:47+09:00] INFO: Report handlers complete
 root@jenkins-master:~#

実行されたことを実際に確認する。

.. code-block:: console

 root@jenkins-master:~# ls -la /etc/chef/
 合計 16
 drwxr-xr-x  2 root root 4096 11月  2 15:27 .
 drwxr-xr-x 89 root root 4096 11月  2 15:27 ..
 -rw-------  1 root root 1675 11月  2 15:24 client.pem
 -rw-r--r--  1 root root  205 11月  2 15:23 client.rb
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# dpkg -l logwatch
 要望=(U)不明/(I)インストール/(R)削除/(P)完全削除/(H)維持
 | 状態=(N)無/(I)インストール済/(C)設定/(U)展開/(F)設定失敗/(H)半インストール/(W)トリガ待ち/(T)トリガ保留
 |/ エラー?=(空欄)無/(R)要再インストール (状態,エラーの大文字=異常)
 ||/ 名前         バージョ   説明
 +++-==============-==============-============================================
 ii  logwatch       7.4.0+svn20111 log analyser with nice output written in Per
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# dpkg -l postfix
 要望=(U)不明/(I)インストール/(R)削除/(P)完全削除/(H)維持
 | 状態=(N)無/(I)インストール済/(C)設定/(U)展開/(F)設定失敗/(H)半インストール/(W)トリガ待ち/(T)トリガ保留
 |/ エラー?=(空欄)無/(R)要再インストール (状態,エラーの大文字=異常)
 ||/ 名前         バージョ   説明
 +++-==============-==============-============================================
 ii  postfix        2.9.3-2~12.04. High-performance mail transport agent
 root@jenkins-master:~# 
 root@jenkins-master:~# ps auxwwwf | grep '[ p]ostfix'
 root     11334  0.0  0.0  25104  1528 ?        Ss   15:27   0:00 /usr/lib/postfix/master
 postfix  11335  0.0  0.0  27168  1468 ?        S    15:27   0:00  \_ pickup -l -t fifo -u
 postfix  11336  0.0  0.0  27220  1496 ?        S    15:27   0:00  \_ qmgr -l -t fifo -u
 root@jenkins-master:~# 

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
 +root:	kanrinmaru@creationline.com
 +#
 root@jenkins-master:~# newaliases 
 root@jenkins-master:~# 

jenkins のインストール
----------------------

.. code-block:: console

 root@jenkins-master:~# aptitude install jenkins
 以下の新規パッケージがインストールされます:
   ant{a} ant-optional{a} bsh{a} bsh-gcj{a} ca-certificates-java{a} 
   default-jre-headless{a} fontconfig{a} fontconfig-config{a} fop{a} 
   gcj-4.6-base{a} gcj-4.6-jre-lib{a} hicolor-icon-theme{a} 
   icedtea-6-jre-cacao{a} icedtea-6-jre-jamvm{a} icedtea-netx{a} 
   icedtea-netx-common{a} java-common{a} java-wrappers{a} jenkins 
   jenkins-cli{a} jenkins-common{a} junit{a} junit4{a} 
   libanimal-sniffer-java{a} libapache-pom-java{a} libargs4j-java{a} 
   libasm3-java{a} libasound2{a} libasyncns0{a} libatk-wrapper-java{a} 
   libatk-wrapper-java-jni{a} libatk1.0-0{a} libatk1.0-data{a} 
   libavahi-client3{a} libavahi-common-data{a} libavahi-common3{a} 
   libavalon-framework-java{a} libbackport-util-concurrent-java{a} 
   libbatik-java{a} libbsf-java{a} libcairo2{a} libclassworlds-java{a} 
   libcommons-beanutils-java{a} libcommons-cli-java{a} 
   libcommons-codec-java{a} libcommons-collections3-java{a} 
   libcommons-configuration-java{a} libcommons-digester-java{a} 
   libcommons-httpclient-java{a} libcommons-io-java{a} 
   libcommons-jexl-java{a} libcommons-jxpath-java{a} libcommons-lang-java{a} 
   libcommons-logging-java{a} libcommons-net2-java{a} 
   libcommons-parent-java{a} libcommons-vfs-java{a} libconstantine-java{a} 
   libcups2{a} libdatrie1{a} libdoxia-java{a} libeasymock-java{a} 
   libflac8{a} libfontconfig1{a} libfop-java{a} libganymed-ssh2-java{a} 
   libgcj-bc{a} libgcj-common{a} libgcj12{a} libgdk-pixbuf2.0-0{a} 
   libgdk-pixbuf2.0-common{a} libgif4{a} libgoogle-collections-java{a} 
   libgtk2.0-0{a} libgtk2.0-bin{a} libgtk2.0-common{a} libhamcrest-java{a} 
   libhttpclient-java{a} libhttpcore-java{a} libice6{a} libitext1-java{a} 
   libjaffl-java{a} libjasper1{a} libjaxp1.3-java{a} 
   libjenkins-remoting-java{a} libjetty-java{a} libjffi-java{a} 
   libjffi-jni{a} libjline-java{a} libjna-java{a} libjnr-posix-java{a} 
   libjpeg-turbo8{a} libjpeg8{a} libjsch-java{a} libjson0{a} 
   libjsoup-java{a} libjsr305-java{a} liblog4j1.2-java{a} 
   libmaven-common-artifact-filters-java{a} libmaven-dependency-tree-java{a} 
   libmaven-enforcer-plugin-java{a} libmaven-invoker-java{a} 
   libmaven-plugin-testing-java{a} libmaven-scm-java{a} 
   libmaven2-core-java{a} libmodello-java{a} libnetbeans-cvsclient-java{a} 
   libnspr4{a} libnss3{a} libnss3-1d{a} libogg0{a} libpango1.0-0{a} 
   libpixman-1-0{a} libplexus-ant-factory-java{a} libplexus-archiver-java{a} 
   libplexus-bsh-factory-java{a} libplexus-build-api-java{a} 
   libplexus-cipher-java{a} libplexus-classworlds-java{a} 
   libplexus-container-default-java{a} libplexus-containers-java{a} 
   libplexus-i18n-java{a} libplexus-interactivity-api-java{a} 
   libplexus-interpolation-java{a} libplexus-io-java{a} 
   libplexus-sec-dispatcher-java{a} libplexus-utils-java{a} libpulse0{a} 
   libregexp-java{a} librhino-java{a} libsaxon-java{a} libservlet2.5-java{a} 
   libslf4j-java{a} libsm6{a} libsndfile1{a} libthai-data{a} libthai0{a} 
   libtiff4{a} libvorbis0a{a} libvorbisenc2{a} libwagon-java{a} 
   libxalan2-java{a} libxbean-java{a} libxcb-render0{a} libxcb-shm0{a} 
   libxcomposite1{a} libxcursor1{a} libxdamage1{a} libxerces2-java{a} 
   libxfixes3{a} libxft2{a} libxi6{a} libxinerama1{a} 
   libxml-commons-external-java{a} libxml-commons-resolver1.1-java{a} 
   libxmlgraphics-commons-java{a} libxrandr2{a} libxrender1{a} libxt6{a} 
   libxtst6{a} openjdk-6-jre{a} openjdk-6-jre-headless{a} 
   openjdk-6-jre-lib{a} rhino{a} shared-mime-info{a} ttf-dejavu-core{a} 
   ttf-dejavu-extra{a} tzdata-java{a} unzip{a} x11-common{a} 
 0 個のパッケージを更新、 170 個を新たにインストール、 0 個を削除予定、0 個が更新されていない。
 148 M バイトのアーカイブを取得する必要があります。 展開後に 301 M バイトのディスク領域が新たに消費されます。
 先に進みますか? [Y/n/?] y
 	:
 	:
 	:
 jenkins (1.424.6+dfsg-1ubuntu0.1) を設定しています ...
 jenkins start/running, process 14445
 	:
 	:
 	:
 ldconfig deferred processing now taking place
                                          
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# ps auxwwwf | grep '[ j]enkins'
 jenkins  14445  7.3  1.2 4307948 102572 ?      Ssl  15:34   0:04 /usr/bin/java -jar /usr/share/jenkins/jenkins.war --webroot=/var/run/jenkins/war --httpPort=8080 --ajp13Port=-1 --preferredClassLoader=java.net.URLClassLoader --logfile=/var/log/jenkins/jenkins.log
 root@jenkins-master:~# 

Jenkinsの設定変更
-----------------

Jenkins + bitbucket.org で Sphinx で作られた Web サイトを自動公開する
http://d.hatena.ne.jp/tk0miya/20111212/p2

.. code-block:: console

 root@jenkins-master:~# cp -a /etc/default/jenkins /etc/default/jenkins.2012-1102
 root@jenkins-master:~# vi /etc/default/jenkins
 root@jenkins-master:~# diff -u /etc/default/jenkins.2012-1102 /etc/default/jenkins
 --- /etc/default/jenkins.2012-1102	2012-03-27 19:40:15.000000000 +0900
 +++ /etc/default/jenkins	2012-11-02 15:58:41.763901017 +0900
 @@ -55,4 +55,7 @@
  # --argumentsRealm.passwd.$ADMIN_USER=[password]
  # --argumentsRealm.$ADMIN_USER=admin
  # --webroot=~/.jenkins/war
 -JENKINS_ARGS="--webroot=$JENKINS_RUN/war --httpPort=$HTTP_PORT --ajp13Port=$AJP_PORT --preferredClassLoader=java.net.URLClassLoader"
 +# 2012/11/02 d-higuchi add --prefix=/jenkins
 +#JENKINS_ARGS="--webroot=$JENKINS_RUN/war --httpPort=$HTTP_PORT --ajp13Port=$AJP_PORT --preferredClassLoader=java.net.URLClassLoader"
 +JENKINS_ARGS="--webroot=$JENKINS_RUN/war --httpPort=$HTTP_PORT --ajp13Port=$AJP_PORT --preferredClassLoader=java.net.URLClassLoader --prefix=/jenkins"
 +#
 root@jenkins-master:~# 

.. code-block:: console

 root@jenkins-master:~# /etc/init.d/jenkins restart
 jenkins stop/waiting
 jenkins start/running, process 15758
 root@jenkins-master:~# 

sphinx との連携
---------------

.. code-block:: console

 root@jenkins-master:~# aptitude install python-sphinx
 以下の新規パッケージがインストールされます:
   docutils-common{a} docutils-doc{a} libjs-sphinxdoc{a} libjs-underscore{a} 
   liblcms1{a} libpaper-utils{a} libpaper1{a} libxslt1.1{a} 
   python-docutils{a} python-imaging{a} python-jinja2{a} python-lxml{a} 
   python-markupsafe{a} python-pygments{a} python-roman{a} python-sphinx 
   sphinx-common{a} sphinx-doc{a} 
 0 個のパッケージを更新、 18 個を新たにインストール、 0 個を削除予定、1 個が更新されていない。
 5,260 k バイトのアーカイブを取得する必要があります。 展開後に 19.0 M バイトのディスク領域が新たに消費されます。
 先に進みますか? [Y/n/?] y
 	:
 	:
 	:
 ldconfig deferred processing now taking place
                                          
 root@jenkins-master:~# 

chef レシピ recipe[apache2] を web UI で追加。

.. code-block:: console

 root@jenkins-master:~# mkdir /var/www/sphinx
 root@jenkins-master:~# chown jenkins.jenkins /var/www/sphinx
 root@jenkins-master:~# 

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
 +--[ RSA 2048]----+
 |    ..           |
 |   .+..          |
 |   +==.          |
 |.. E*o           |
 |+ .o... S        |
 |.oo .  .         |
 |...o .           |
 | .. o.           |
 |   ....          |
 +-----------------+
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

新規ジョブの作成
	ジョブ名: chef-training-p
	フリースタイル・プロジェクトのビルド

	ソースコード管理システム: git
	Repository URL: ssh://git@bitbucket.org/j_hotta/chef-training-p.git

	SCMをポーリング: on
	スケジュール: */10 * * * *

	ビルド手順の追加
	シェルの実行:
		make clean html
		rsync -av --delete /var/lib/jenkins/jobs/chef-traning-p/workspace/build/html/ /var/www/sphinx/chef-training-p

	E-mail通知: on
	宛先: kanrinmaru@creationline.com
	不安定ビルドも逐一メールを送信: on
	ビルドを壊した個人にも別途メールを送信: off

..
 [EOF]
