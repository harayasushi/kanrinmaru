kanrinmaru02.akb.creationline.com (219.117.239.178)
===================================================

OS 基本情報
-----------

- CentOS 6.3 (x86_64)
- 最小インストール
- パーティション設定: 自動

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

 enStratus インストーラが /etc/hosts を書き換えることに注意。
 また、FQDN が設定されていないと enStratus インストーラが失敗する。

on-premise enStratus のインストール (2012/10/22 現在)
-----------------------------------------------------

詳細は http://on-premise.enstratus.com/installation/single_node/single_node.html を参照する。

インストーラを展開する。

.. code-block:: console

 root@kanrinmaru02:~# unzip enStratus-es-onpremise-chef-solo-hobbit-p0.1-0-*.zip
        :
 root@kanrinmaru02:~# 

まず、Java をインストールする。

.. code-block:: console

 root@kanrinmaru02:~# chmod +x jdk-6u33-linux-x64.bin 
 root@kanrinmaru02:~# ./jdk-6u33-linux-x64.bin 
        :
 root@kanrinmaru02:~# mv jdk1.6.0_33 jdk
 root@kanrinmaru02:~# tar cfz enStratus-es-onpremise-chef-solo/cookbooks/enstratus/files/default/jdk.tar.gz jdk
 root@kanrinmaru02:~# 

 root@kanrinmaru02:~# unzip jce_policy-6.zip 
        :
 root@kanrinmaru02:~# mv jce enStratus-es-onpremise-chef-solo/cookbooks/enstratus/files/default/
 root@kanrinmaru02:~# 

インストーラを起動する。-p には enStratus から提供されたアクセスキーと、FQDNを指定する。
インストーラはまず Chef クライアントのインストールや Java の設定が行われ、Chef-Solo を実行するための準備がされる。

.. code-block:: console

 root@kanrinmaru02:~/enStratus-es-onpremise-chef-solo# ./setup.sh -l "" -p '★★★★' -c kanrinmaru02.akb.creationline.com
	:
 Installing Chef 
	:
 Thank you for installing Chef!
 ## CHECKING JDK and JCE setup under /home/ubuntu/enStratus-es-onpremise-chef-solo/cookbooks/enstratus/files/default
 Found local_policy
 Found US_export_policy
 ## EXTRACTING temporary JDK - from /home/ubuntu/enStratus-es-onpremise-chef-solo/cookbooks/enstratus/files/default/jdk.tar.gz to /tmp
 ## COPYING JCE jars to temporary JDK
 Generating Keys
 Creating local_settings//tmp/tmp.AEm7kRrtCY/genkeys.txt file
 Writing JSON files to 'local_settings//tmp/tmp.AEm7kRrtCY/'
 Using erubis binary from: /opt/chef/bin/erubis
 Ready to run : chef-solo -j local_settings//tmp/tmp.AEm7kRrtCY/single_node.json -c solo.rb
 root@kanrinmaru02:~/enStratus-es-onpremise-chef-solo# 

.. code-block:: console

 root@kanrinmaru02:~/enStratus-es-onpremise-chef-solo# chef-solo -j local_settings//tmp/tmp.AEm7kRrtCY/single_node.json -c solo.rb
        :
 [2012-10-22T13:39:06+09:00] INFO: Chef Run complete in 878.103415757 seconds
 [2012-10-22T13:39:06+09:00] INFO: Running report handlers
 [2012-10-22T13:39:06+09:00] INFO: Report handlers complete
 root@kanrinmaru02:~/enStratus-es-onpremise-chef-solo# 

完了したら http://on-premise.enstratus.com/installation/single_node/post_install.html を参照して後設定を行う。

.. code-block:: console

 root@kanrinmaru02:~# /etc/init.d/enstratus-km start
 Starting Key Manager.
 Using CATALINA_BASE:   /services/km/tomcat
 Using CATALINA_HOME:   /services/km/tomcat
 Using CATALINA_TMPDIR: /services/km/tomcat/temp
 Using JRE_HOME:       /usr/local/jdk
 root@kanrinmaru02:~# 

 root@kanrinmaru02:~# netstat -tnlup | grep 2013
 tcp6       0      0 :::2013                 :::*                    LISTEN      29246/java      
 root@kanrinmaru02:~# 

 root@kanrinmaru02:~# /etc/init.d/enstratus-dispatcher start
 Starting pinger.
 Starting Dispatcher.
 Using CATALINA_BASE:   /services/dispatcher/tomcat
 Using CATALINA_HOME:   /services/dispatcher/tomcat
 Using CATALINA_TMPDIR: /services/dispatcher/tomcat/temp
 Using JRE_HOME:       /usr/local/jdk
 root@kanrinmaru02:~#

 root@kanrinmaru02:~# netstat -tnlup | grep 3302
 tcp6       0      0 :::3302                 :::*                    LISTEN      29296/java      
 root@kanrinmaru02:~# 
 
 root@kanrinmaru02:~# /etc/init.d/enstratus-console start
 Starting enstratus-console
 Using CATALINA_BASE:   /services/console/tomcat
 Using CATALINA_HOME:   /services/console/tomcat
 Using CATALINA_TMPDIR: /services/console/tomcat/temp
 Using JRE_HOME:        /usr/local/jdk
 Using CLASSPATH:       /services/console/tomcat/bin/bootstrap.jar:/services/console/tomcat/bin/tomcat-juli.jar
 root@kanrinmaru02:~# 

https://kanrinmaru02.akb.creationline.com/page/1/register.jsp にアクセスして登録する。

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

enStratus インストーラが /etc/hosts を書き換えるので、
recipe[cl-etc-common::hostname] は使わない。

ファイアウォールの設定
----------------------

run_list に以下を追加するだけでOK。

- recipe[lokkit::service_ssh]
- recipe[lokkit::https_tcp]

..
 [EOF]
