# FreeIPA installation and deployment of the replica multimaster on CENTOS7
FreeIPA installation Tutorial, Scripts and Procedures

## Planning the server and architecture to deploy. Requirements.

In the IPA domain, there are three types of machines:

+ Servers, which manage all of the services used by domain members
+ Replicas, which are essentially copies of servers (and, once copied, are identical to serve)
+ Clients, which connect to the Servers to authentication and authorization

In our deployment we will create the next:

+ One IPA master server with name: `ipa.centos.local` and ip: `192.168.10.200`
+ One IPA replica of the master: `ipa2.centos.local` and ip: `192.168.10.201`

![alt text](https://github.com/manuparra/FreeIPA/raw/master/architecture.png "Architecture")

## Requeriments

+ Both servers with the same version of Operanting System. In this tutorial we install lastest version of CENTOS7.
+ Lastest versión of freeIPA from `yum`  (version in this moment: 4.2.0-15).
+ I had problem with low RAM, so I update memory of each server to 3GB.
+ Min. of 10GB of HDD storage for each server is okay.




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
12. Execute `ipa user-show admin` and copy UID number. Then execute: `chown XXXXXX:XXXXXX /home/admin` where XXXXXXX is the UID number of admin user.
13. Check if IPA works. Exit of the server and try to connect: `ssh manuparra@192.168.10.220` If it is working, ssh ask to you about change your password and retype it twice. If you can access to the server, IPA server now is Working.

## Creating a replica

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

1. Execute again `kinit admin`
2. Execute `ipa-replica-install --setup-ca --setup-dns --no-forwarders /var/lib/ipa/replica-info-ipa2.centos.local.gpg` . Note that `/var/lib/ipa/replica-info-ipa2.centos.local.gpg` must be there, because in the previous step it was copied from main server.
3. Wait a few minutes.
4. That's all. Verify if your new replica is working executing this command: `ipa-replica-manage list` . It returns the next:
  
```
ipa2.centos.local: master
  
ipa.centos.local: master

```





