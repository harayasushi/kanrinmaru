data bag
========

[cl-etc-common::aliases]
------------------------

.. code-block:: console

 [cluser@kanrinmaru01 chef-repo]$ cat data_bags/cl-etc-common/aliases.json
 {
        "id": "aliases",
        "aliases": {
                "root": "solution@creationline.com"
        }
 }
 [cluser@kanrinmaru01 chef-repo]$

[cl-etc-common::hosts-access]
-----------------------------

- 219.117.239.160/255.255.255.224 : CL global server
- 192.168.1.0/255.255.255.0 : CL wifi 1
- 192.168.2.0/255.255.255.0 : CL wifi 2
- 192.168.10.0/255.255.255.0 : CL private server
- 192.168.122.0/255.255.255.0 : CL sandbox
- .tyma.nt.ftth4.ppp.infoweb.ne.jp : d-higuchi
- 221.249.136.50/255.255.255.248 : j-hotta
- 124.35.220.7 : y-uemura

.. code-block:: console

 [cluser@kanrinmaru01 chef-repo]$ cat data_bags/cl-etc-common/hosts-access.json
 {
        "id": "hosts-access",
        "allow": {
                "sshd": [
                        "219.117.239.160/255.255.255.224",
                        "192.168.1.0/255.255.255.0",
                        "192.168.2.0/255.255.255.0",
                        "192.168.10.0/255.255.255.0",
			"192.168.122.0/255.255.255.0",
                        ".tyma.nt.ftth4.ppp.infoweb.ne.jp",
                        "221.249.136.50/255.255.255.248",
                        "124.35.220.7"
                ]
        },
        "deny": {
                "sshd": [
                        "ALL"
                ]
        }
 }
 [cluser@kanrinmaru01 chef-repo]$

[cl-reverse-proxy::access]
--------------------------

- 219.117.239.160/255.255.255.224 : CL global server
- 192.168.1.0/255.255.255.0 : CL wifi 1
- 192.168.2.0/255.255.255.0 : CL wifi 2
- 192.168.10.0/255.255.255.0 : CL private server
- 192.168.122.0/255.255.255.0 : CL sandbox
- .tyma.nt.ftth4.ppp.infoweb.ne.jp : d-higuchi
- 221.249.136.50/255.255.255.248 : j-hotta
- 124.35.220.7 : y-uemura

.. code-block:: console

 [cluser@kanrinmaru01 chef-repo]$ cat data_bags/cl-reverse-proxy/access.json 
 {
	"id": "access",
	"deny": [
		"all"
	],
	"allow": [
		"219.117.239.160/255.255.255.224",
		"192.168.1.0/255.255.255.0",
		"192.168.2.0/255.255.255.0",
		"192.168.10.0/255.255.255.0",
		"192.168.122.0/255.255.255.0",
		".tyma.nt.ftth4.ppp.infoweb.ne.jp",
		"221.249.136.50/255.255.255.248",
		"124.35.220.7"
	]
 }
 [cluser@kanrinmaru01 chef-repo]$ 

[cl-reverse-proxy::backend]
---------------------------

.. code-block:: console

 [cluser@kanrinmaru01 chef-repo]$ cat data_bags/cl-reverse-proxy/backend.json 
 {
	"id": "backend",
	"jenkins-master": {
		"path": "/jenkins",
		"uri": "http://192.168.122.11:8080/jenkins",
		"htpasswd": "/etc/apache2/htpasswd.jenkins-master",
		"realm": "jenkins-master"
	},
	"sphinx": {
		"path": "/sphinx",
		"uri": "http://192.168.122.11/sphinx",
		"htpasswd": "/etc/apache2/htpasswd.jenkins-master",
		"realm": "jenkins-master"
	},
	"rabbit": {
		"path": "/rabbit",
		"uri": "http://192.168.122.11/rabbit",
		"htpasswd": "/etc/apache2/htpasswd.jenkins-master",
		"realm": "jenkins-master"
	},
	"redmine": {
		"path": "/redmine",
		"uri": "http://192.168.122.21/redmine",
		"htpasswd": "/etc/apache2/htpasswd.redmine",
		"realm": "redmine"
	}
 }
 [cluser@kanrinmaru01 chef-repo]$ 

..
 [EOF]
