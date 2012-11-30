kanrinmaru01.akb.creationline.com (219.117.239.177)
===================================================

CentOS 6.3 に Private Chef Server のインストール (Standalone 編)
----------------------------------------------------------------

`Universal Prerequisites`_ と `Standalone Installation`_ を参照すること。

.. _Universal Prerequisites: http://private-chef-docs.opscode.com/installation/prereqs.html
.. _Standalone Installation: http://private-chef-docs.opscode.com/installation/standalone.html

FQDN の設定
-----------

.. code-block:: console

 [root@kanrinmaru01 ~]# hostname --fqdn
 hostname: Unknown host
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# cp -a /etc/hosts /etc/hosts.2012-1015
 [root@kanrinmaru01 ~]# vi /etc/hosts                        
 [root@kanrinmaru01 ~]# diff -u /etc/hosts.2012-1015 /etc/hosts
 --- /etc/hosts.2012-1015	2010-01-12 22:28:22.000000000 +0900
 +++ /etc/hosts	2012-10-15 16:17:09.518586743 +0900
 @@ -1,2 +1,4 @@
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
 +
 +219.117.239.177	kanrinmaru01.akb.creationline.com	kanrinmaru01
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# hostname --fqdn
 kanrinmaru01.akb.creationline.com
 [root@kanrinmaru01 ~]# 

NTP の起動
----------

.. code-block:: console

 [root@kanrinmaru01 ~]# ps auxwwwf | grep "[ n]tp"
 ntp       1137  0.0  0.0  30160  1624 ?        Ss   Aug29   0:03 ntpd -u ntp:ntp -p /var/run/ntpd.pid -g
 [root@kanrinmaru01 ~]# 

メール配送の確認
----------------

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -q postfix
 postfix-2.6.6-2.2.el6_1.x86_64
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# telnet localhost 25        
 Trying ::1...
 Connected to localhost.
 Escape character is '^]'.
 220 kanrinmaru01.localdomain ESMTP Postfix
 HELO localhost
 250 kanrinmaru01.localdomain
 MAIL FROM: <cluser>
 250 2.1.0 Ok
 RCPT TO: <d-higuchi@creationline.com>
 250 2.1.5 Ok
 DATA
 354 End data with <CR><LF>.<CR><LF>
 From: cluser
 To: d-higuchi@creationline.com
 Subject: test 1619
      
 test 1619
 .
 250 2.0.0 Ok: queued as BCCC01780285
 RSET
 250 2.0.0 Ok
 QUIT
 221 2.0.0 Bye
 Connection closed by foreign host.
 [root@kanrinmaru01 ~]# 

届いたメール

.. code-block:: console

 Delivered-To: d-higuchi@creationline.com
 Received: by 10.194.19.72 with SMTP id c8csp47987wje;
         Mon, 15 Oct 2012 00:19:44 -0700 (PDT)
 Received: by 10.68.237.231 with SMTP id vf7mr2535465pbc.63.1350285583457;
         Mon, 15 Oct 2012 00:19:43 -0700 (PDT)
 Return-Path: <cluser@kanrinmaru01.localdomain>
 Received: from kanrinmaru01.localdomain (219.117.239.177.static.zoot.jp. [219.117.239.177])
         by mx.google.com with ESMTP id pm3si21276736pbc.341.2012.10.15.00.19.42;
         Mon, 15 Oct 2012 00:19:43 -0700 (PDT)
 Received-SPF: neutral (google.com: 219.117.239.177 is neither permitted nor denied by best guess record for domain of cluser@kanrinmaru01.localdomain) client-ip=219.117.239.177;
 Authentication-Results: mx.google.com; spf=neutral (google.com: 219.117.239.177 is neither permitted nor denied by best guess record for domain of cluser@kanrinmaru01.localdomain) smtp.mail=cluser@kanrinmaru01.localdomain
 Received: from localhost (localhost [IPv6:::1])
 	by kanrinmaru01.localdomain (Postfix) with SMTP id BCCC01780285
 	for <d-higuchi@creationline.com>; Mon, 15 Oct 2012 16:19:17 +0900 (JST)
 From: cluser@kanrinmaru01.localdomain
 To: d-higuchi@creationline.com
 Subject: test 1619
 Message-Id: <20121015071926.BCCC01780285@kanrinmaru01.localdomain>
 Date: Mon, 15 Oct 2012 16:19:17 +0900 (JST)

リレーされている。

crontabs
--------

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -q crontabs
 crontabs-1.10-33.el6.noarch
 [root@kanrinmaru01 ~]# 

git
---

.. code-block:: console

 [root@kanrinmaru01 ~]# yum install git
 	:
 Installed:
   git.x86_64 0:1.7.1-2.el6_0.1
 
 Dependency Installed:
   perl.x86_64 4:5.10.1-127.el6
   perl-Error.noarch 1:0.17015-4.el6
   perl-Git.noarch 0:1.7.1-2.el6_0.1
   perl-Module-Pluggable.x86_64 1:3.90-127.el6
   perl-Pod-Escapes.x86_64 1:1.04-127.el6
   perl-Pod-Simple.x86_64 1:3.13-127.el6
   perl-libs.x86_64 4:5.10.1-127.el6
   perl-version.x86_64 3:0.77-127.el6
   rsync.x86_64 0:3.0.6-9.el6
 
 Complete!
 [root@kanrinmaru01 ~]# 

freetype, libpng
----------------

.. code-block:: console

 [root@kanrinmaru01 ~]# yum install freetype libpng
 	:
 Installed:
   freetype.x86_64 0:2.3.11-6.el6_2.9       libpng.x86_64 2:1.2.49-1.el6_2
 
 Complete!
 [root@kanrinmaru01 ~]# 

apache qpid
-----------

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -qa | grep qpid
 python-qpid-0.14-8.el6.noarch
 qpid-qmf-0.14-7.el6_2.x86_64
 qpid-cpp-server-0.14-16.el6.x86_64
 qpid-tools-0.14-2.el6_2.noarch
 qpid-cpp-client-0.14-16.el6.x86_64
 python-qpid-qmf-0.14-7.el6_2.x86_64
 qpid-cpp-client-ssl-0.14-16.el6.x86_64
 qpid-cpp-server-ssl-0.14-16.el6.x86_64
 [root@kanrinmaru01 ~]# 

停止ではなく削除しておく。

.. code-block:: console

 [root@kanrinmaru01 ~]# yum erase '*qpid*'
 	:
 Removed:
   python-qpid.noarch 0:0.14-8.el6      python-qpid-qmf.x86_64 0:0.14-7.el6_2   
   qpid-cpp-client.x86_64 0:0.14-16.el6 qpid-cpp-client-ssl.x86_64 0:0.14-16.el6
   qpid-cpp-server.x86_64 0:0.14-16.el6 qpid-cpp-server-ssl.x86_64 0:0.14-16.el6
   qpid-qmf.x86_64 0:0.14-7.el6_2       qpid-tools.noarch 0:0.14-2.el6_2        
 
 Dependency Removed:
   matahari.x86_64 0:0.6.0-14.el6
   matahari-agent-lib.x86_64 0:0.6.0-14.el6
   matahari-broker.x86_64 0:0.6.0-14.el6
   matahari-consoles.x86_64 0:0.6.0-14.el6
   matahari-host.x86_64 0:0.6.0-14.el6
   matahari-network.x86_64 0:0.6.0-14.el6
   matahari-python.x86_64 0:0.6.0-14.el6
   matahari-rpc.x86_64 0:0.6.0-14.el6
   matahari-service.x86_64 0:0.6.0-14.el6
   matahari-sysconfig.x86_64 0:0.6.0-14.el6
 
 Complete!
 [root@kanrinmaru01 ~]# 

インストール
------------

.. code-block:: console

 [root@kanrinmaru01 ~]# rpm -Uvh ~cluser/private-chef-demo-1.el6.x86_64.rpm 
 Preparing...                ########################################### [100%]
    1:private-chef           ########################################### [100%]
 Thank you for installing Chef!
 [root@kanrinmaru01 ~]# 

ファイアウォールの開放
----------------------

.. code-block:: console

 [root@kanrinmaru01 ~]# iptables -n -L INPUT 
 Chain INPUT (policy ACCEPT)
 target     prot opt source               destination         
 ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
 ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
 ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
 ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
 REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 
 [root@kanrinmaru01 ~]#

.. code-block:: console

 [root@kanrinmaru01 ~]# lokkit --service http
 [root@kanrinmaru01 ~]# lokkit --service https
 [root@kanrinmaru01 ~]# 

.. code-block:: console

 [root@kanrinmaru01 ~]# iptables -n -L INPUT  
 Chain INPUT (policy ACCEPT)
 target     prot opt source               destination         
 ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
 ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
 ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
 ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
 ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:80 
 ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:443 
 REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 
 [root@kanrinmaru01 ~]# 

後設定
------

.. code-block:: console

 [root@kanrinmaru01 ~]# private-chef-ctl reconfigure 2>&1 | tee reconfigure.log
 	:
 Chef Server Reconfigured!
 [root@kanrinmaru01 ~]# 

Chef クライアント化設定
-----------------------

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

run_list に以下を追加するだけでOK。

- recipe[lokkit::service_ssh]
- recipe[lokkit::https_tcp]
- recipe[lokkit::8443_tcp]

リバースプロキシの設定
----------------------

run_list に以下を追加する。

- recipe[cl-chef-proxy]

パスワードを設定する。PrimeDrive を参照のこと。

.. code-block:: console

 [root@kanrinmaru01 ~]# htpasswd -c /etc/httpd/htpasswd.chefui chefui
 New password:
 Re-type new password:
 Adding password for user chefui
 [root@kanrinmaru01 ~]#

Private Chef Server の nginx のアクセス制限を強める。
nginx はホスト名で allow/deny が書けないことに注意。

.. code-block:: console

 [root@kanrinmaru01 ~]# cp -a /var/opt/opscode/nginx/etc/nginx.conf /var/opt/opscode/nginx/etc/nginx.conf.2012-1114
 [root@kanrinmaru01 ~]# vi /var/opt/opscode/nginx/etc/nginx.conf
 [root@kanrinmaru01 ~]# diff -u /var/opt/opscode/nginx/etc/nginx.conf.2012-1017 /var/opt/opscode/nginx/etc/nginx.conf
 --- /var/opt/opscode/nginx/etc/nginx.conf.2012-1017     2012-10-15 16:28:49.293812507 +0900
 +++ /var/opt/opscode/nginx/etc/nginx.conf       2012-11-14 16:36:30.440394225 +0900
 @@ -52,6 +52,12 @@
    server {
      listen 80;
      server_name kanrinmaru01.akb.creationline.com;
 +
 +# 2012/11/14 add d-higuchi
 +    allow 127.0.0.1;
 +    deny  all;
 +#
 +
      access_log /var/log/opscode/nginx/rewrite-port-80.log;
      rewrite ^(.*) https://$server_name$1 permanent;
    }
 @@ -59,6 +65,12 @@
    server {
      listen 443;
      server_name kanrinmaru01.akb.creationline.com;
 +
 +# 2012/11/14 add d-higuchi
 +    allow 127.0.0.1;
 +    deny  all;
 +#
 +
      access_log /var/log/opscode/nginx/access.log opscode;
      ssl on;
      ssl_certificate /var/opt/opscode/nginx/ca/kanrinmaru01.akb.creationline.com.crt;
 [root@kanrinmaru01 ~]#

Private Chef Server の nginx を再起動する。

.. code-block:: console

 [root@kanrinmaru01 ~]# private-chef-ctl nginx restart
 ok: run: nginx: (pid 9394) 0s
 [root@kanrinmaru01 ~]#

..
 [EOF]
