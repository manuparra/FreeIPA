# Introduction to FreeIPA
Manuel Parra & José Manuel Benítez, 2016

![imgfreeipa](https://scottlinux.com/wp-content/uploads/2015/11/freeipa.jpg)

## What is FreeIPA:

FreeIPA is an integrated security information management solution combining Linux , 389 Directory Server, MIT Kerberos, NTP, DNS, Dogtag (Certificate System).
It consists:

1. web interface
2. command-line administration tools.

A FreeIPA server provides centralized authentication, authorization and account information about user, groups, hosts.

One of the most important features consists on FreeIPA, can be configured in a FreeIPA Domain in order to provide redundancy and scalability.

- Main Datastore: 389 Directory Server and LDAPv3
- Sing-On provided by: MIT Kerberos KDC.
- Authentication capabilities are instatiated by the Dogtag

## The core:

* Directory Server
* Kerberos KDC: Authentication
* PKIServer: Certificates for services(web,LDAP, TLS)
* HTTP Server: Public web API

![freeIPACORE](https://sites.google.com/site/manuparra/home/CoreFreeIPA.png)


## Why I use freeIPA in my organization:

For efficiency, compliance and risk mitigation.
We need to centrally manage and correlate vital security information including:

* Identity (machine, user, virtual machines, groups, authentication credentials)
* Policy (host based access control)
* Identity, policy, and audit information should be interoperable, and manageable. 

![DesktopFreeIPA](https://www.adelton.com/docs/idm/freeipa-webui-user-crop.png)

*Image of Admin Desktop of freeIPA*


## Key Features:

* Integrated security information management solution combining:	
	* 389 Directory Server, 
	* MIT Kerberos, 
	* NTP, 
	* DNS, 
	* Dogtag certificate system, 
	* SSSD and others.
* Strong focus on ease of management and automation of installation and configuration tasks.
* <B>Hide complexity of LDAP+Kerberos+CA+ ...DEPLOYMENT</B>. Easy use of CLI/WebUI.
* Full multi master replication for higher redundancy and scalability. Fine for your organization. Fault tollerant!.
* Extensible management interfaces (CLI, Web UI, XMLRPC and JSONRPC API) and Python SDK


## Features: Identity manager

* Users and Groups:
	* Automatic and unique UIDS across replicas.
	* SSH public keys management.
	* Role-based account control.
* Hosts, host-groups, netgroups:
	* Manage host life-cicle.
* Automatic group membership based on rules:
	* Just add a user/host and all groups membership will be added.

## Features: Policy Management

<B>HBAC</B>

* Control who (can), when and what to do
* Using SSSD for auth throught PAM


# FreeIPA installation and deployment with replica multimaster on CENTOS7

FreeIPA installation Tutorial, Scripts and Procedures, step by step.

## Planning the server and architecture to deploy. Requirements.

In the IPA domain, there are three types of machines:

+ Servers, which manage all of the services used by domain members
+ Replicas, which are essentially copies of servers (and, once copied, are identical to serve)
+ Clients, which connect to the Servers to authentication and authorization

In our deployment we will create the next:

+ One IPA master server with name: `ipa.centos.local` and ip: `192.168.10.200`
+ One IPA replica of the master: `ipa2.centos.local` and ip: `192.168.10.201`

![alt text](https://github.com/manuparra/FreeIPA/raw/master/images/architecture.png "Architecture")

## Requeriments

+ Both servers with the same version of Operanting System. In this tutorial we install lastest version of CENTOS7.
+ Lastest versión of freeIPA from `yum`  (version in this moment: 4.2.0-15).
+ I had problem with low RAM, so I update memory of each server to 3GB.
+ Min. of 10GB of HDD storage for each server is okay.
+ Try to use freeIPA only for freeIPA services. It is recommended.



## Installing Master IPA Server.

1. First of all, execute command: 
`yum -y update`
2. Set the name of the IPA Server: 
`hostnamectl set-hostname ipa.centos.local`
3. Edit `/etc/hosts` and add: 
`192.168.10.200 ipa.centos.local ipa`
4. Download and install freeIPA packages with: 
`yum install ipa-server bind-dyndb-ldap ipa-server-dns`
5. Install and set freeIPA services: 
`ipa-server-install --setup-dns`
6. Follow the steps of previous item : [here!](questions.txt)
7. Next steps:
	1. You must make sure these network ports are open TCP Ports:
		  * 80, 443: HTTP/HTTPS
		  * 389, 636: LDAP/LDAPS
		  * 88, 464: kerberos
		  * 53: bind
	2. UDP Ports:
		  * 88, 464: kerberos
		  * 53: bind
		  * 123: ntp
8. Start authentication with Kerberos ticket: 
`kinit admin`
9. Set default shell for the users:
`ipa config-mod --defaultshell=/bin/bash`
10. Create a few users: `ipa user-add manuparra --first=Manuel --last=Parra --password` . Create the home folder for the user created:`mkdir -m0750 -p /home/mparra` and set permissions for the user: `chown XXXXXXXX:XXXXXXXX /home/mparra/` where `XXXXXXXX` is the UID returned by ``ipa user-add manuparra ...``.
11. (Mandatory) Create home folder for admin user, execute: `mkdir -m0750 -p /home/admin/` 
12. Execute `ipa user-show admin` and copy UID number. Then execute: `chown XXXXXX:XXXXXX /home/admin` where XXXXXXX is the UID number of admin user. This is mandatory due to in replica server it try to connect with this user and it needs the home be created.
13. Check if IPA works. Exit of the server and try to connect: `ssh manuparra@192.168.10.220` If it is working, ssh ask to you about change your password and retype it twice. If you can access to the server, IPA server now is Working.


## Installing freeIPA clients

First of all, go to the client and install:

```
yum install ipa-client ipa-admintools
```

then run:

```
ipa-client-install --enable-dns-updates
```

That's all.

### Configuring KERBEROS:

Modify the /etc/krb5.conf  

```
[libdefaults]
 default_realm = CENTOS.LOCAL
 dns_lookup_realm = true
 dns_lookup_kdc = true
 forwardable = yes
 ticket_lifetime = 24h

[realms]
 CENTOS.LOCAL = {
  kdc = ipa.centos.local:88
  admin_server = ipa.centos.local:749
  default_domain = centos.local
 }
[domain_realm]
 .centos.local = CENTOS.LOCAL
 centos.local = CENTOS.LOCAL
 ```

### Configuring Client SSH Access

It is a good idea to make sure that time on the IPA client and server is synchronized:

```
ntpdate -s -p 8 -u ipa.centos.local
```

We obtain a Kerberos ticket for the admin user as usual with admin labours:

 ```
 kinit admin
 ```

 Add a host service on the IPA client:

 ```
 ipa-addservice host/ipa.centos.com
 ```

and then:

```
ipa-getkeytab -s ipa.centos.com -p host/ipa.centos.com -k /etc/krb5.keytab
``

Client now is configured to accept incoming SSH connections and authenticate with the user's Kerberos credentials.

## Authentication from anywhere

Some external authentication methods to integrate freeIPA-

### APIREST

Full RESTAPI available from:

https://ipa.centos.local/ipa/session/json


Meanwhile create a C-Shell script to test: ```testcurl.sh``

```
s_username=<your freeIPA username>
s_password=<your password at freeIPA>
IPAHOSTNAME=ipa.centos.local
COOKIEJAR=the.cookie.jar

klist -s
use_kerberos=$?

if [ ! -f $COOKIEJAR ] ; then
 if [ $use_kerberos -eq 0 ] ; then
        # Login with Kerberos
        curl -v  \
        -H referer:https://$IPAHOSTNAME/ipa  \
        -c $COOKIEJAR -b $COOKIEJAR \
        --cacert /etc/ipa/ca.crt  \
        --negotiate -u : \
        -X POST \
        https://$IPAHOSTNAME/ipa/session/login_kerberos
  else
        # Login with user name and password
        curl -v  \
        -H referer:https://$IPAHOSTNAME/ipa  \
        -H "Content-Type:application/x-www-form-urlencoded" \
        -H "Accept:text/plain"\
        -c $COOKIEJAR -b $COOKIEJAR \
        --cacert /etc/ipa/ca.crt  \
        --data "user=$s_username&password=$s_password" \
        -X POST \
        https://$IPAHOSTNAME/ipa/session/login_password
  fi
fi


curl -v  \
	-H referer:https://$IPAHOSTNAME/ipa  \
        -H "Content-Type:application/json" \
        -H "Accept:applicaton/json"\
        -c $COOKIEJAR -b $COOKIEJAR \
        --cacert /etc/ipa/ca.crt  \
        -d  '{"method":"user_find","params":[[""],{}],"id":0}' \
        -X POST \
        https://$IPAHOSTNAME/ipa/session/json

```


### PHP

Directly from a PHP library for connect and use some features of the freeIPA:

Download and install PHP package:

![https://github.com/gnumoksha/php-freeipa]

Get the certificate of our freeIPA server:
```
wget --no-check-certificate https://ipa.centos.local/ipa/config/ca.crt -O certs/ipa.centos.local_ca.crt
```

Code:
```
<?php
require_once('./bootstrap.php');
$host = 'ipa.centos.local';
$certificate = __DIR__ . "./certs/ipa.centos.local_ca.crt";
$ipa = new \FreeIPA\APIAccess\Main($host, $certificate);
?>
```

Auth example:
```
<?php 

$user = 'admin';
$password = 'Secret123';
$auth = $ipa->connection()->authenticate($user, $password);
if ($auth) {
    print 'Logged in';
} else {
    $auth_info = $ipa->connection()->getAuthenticationInfo();
    var_dump($auth_info);
}
?>
```

Showing user information:
```
<?php
$r = $ipa->user()->get($user);
var_dump($r);
?>
```

### From Web Forms (virtually any webapplication)

To do. Review: http://www.freeipa.org/page/Web_App_Authentication

### APACHE

Install Apache, and modules
```
yum install httpd mod_auth_kerb mod_ssl
```

**Note**: localhost is your web server name. In our case: ``http://localhost``

Admin login and generate KeyTable:
```
kinit admin
ipa-getkeytab -s ipa.centos.local -p HTTP/localhost -k /etc/httpd/conf/httpd.keytab
```

Then:
```
chown apache /etc/httpd/conf/httpd.keytab
```

Create the SSL keys for freeIPA webserver:
```
ipa-getcert request -k /etc/pki/tls/private/webserver.key -f /etc/pki/tls/certs/webserver.crt -K HTTP/localhost -g 3072
```

Set the certificate paths in ``/etc/httpd/conf.d/ssl.conf``:

```
[...]
SSLCertificateFile /etc/pki/tls/certs/webserver.crt
SSLCertificateKeyFile /etc/pki/tls/private/webserver.key
SSLCertificateChainFile /etc/ipa/ca.crt
[...]
```

The final httpd authentication settings for ``mod_auth_kerb`` are done in ``/etc/httpd/conf.d/auth_kerb.conf`` or any vhost you want:

```
<Location />
  SSLRequireSSL
  AuthType Kerberos
  AuthName "Kerberos Login"
  KrbMethodNegotiate On
  KrbMethodK5Passwd On
  KrbAuthRealms centos.local
  Krb5KeyTab /etc/httpd/conf/httpd.keytab
  require valid-user
</Location>
```

That's all. Restart apache, and Location `/` will ask about authentication on freeIPA.

## Creating a replica

<B>Take care: Both servers must be the same time/hour or must by synced by ntpdate. Check it with ``date``<B>

This part of the installation requiere jump to replica server and jump to the main freeIPA server, so note this.


### Go to **ipa.centos.local** and **ipa2.centos.local** (replica), first of all we need to add to `/etc/hosts` the follow entries: 

```
192.168.10.220 ipa.centos.local ipa

192.168.10.221 ipa2.centos.local ipa2
```


### Go to Replica server **(ipa2.centos.local)**:

1. Update packages: `yum update`
2. Set the name of the server to ipa2.centos.local: `hostnamectl set-hostname ipa2.centos.local`
3. Install the packages of freeIPA with `yum install ipa-server bind bind-dyndb-ldap ipa-server-dns`
4. VERY IMPORTANT: DO NOT execute: ~~ipa-server-install --setup-dns~~

### Go to main server **(ipa.centos.local)**:

1. Start authentication with Kerberos ticket: 
`kinit admin`
2. Prepare a replica with this command: `ipa-replica-prepare ipa2.centos.local --ip-address 192.168.10.201`  Note IP of replica server is 192.168.10.201
3. The previous command creates a file in `/var/lib/ipa/replica-info-ipa2.centos.local.gpg`
4. This file `/var/lib/ipa/replica-info-ipa2.centos.local.gpg` must be copied to ipa2.centos.local, so we execute this: `scp  /var/lib/ipa/replica-info-ipa2.centos.local.gpg root@192.168.10.225:/var/lib/ipa/`

### Go to Replica server **(ipa2.centos.local)**:

1. Execute `ipa-replica-install --setup-ca --setup-dns --no-forwarders /var/lib/ipa/replica-info-ipa2.centos.local.gpg` . Note that `/var/lib/ipa/replica-info-ipa2.centos.local.gpg` must be there, because in the previous step it was copied from main server.
2. Wait a few minutes.
3. Execute again `kinit admin`
4. That's all. Verify if your new replica is working executing this command: `ipa-replica-manage list` . It returns the next:
  
```
ipa2.centos.local: master
  
ipa.centos.local: master

```





