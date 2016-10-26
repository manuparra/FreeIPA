# FreeIPA installation and deployment of the replica multimaster 
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
5. Install and set freeIPA services. Follow the steps of this part here: 
`ipa-server-install --setup-dns`
6. Start authentication: 
`kinit admin`
7. Set default shell for the users:
`ipa config-mod --defaultshell=/bin/bash`
8. Create a few users: `ipa user-add manuparra --first=Manuel --last=Parra --password` . Create the home folder for the user created:`mkdir -m0750 -p /home/mparra` and set permissions for the user: `chown XXXXXXXX:XXXXXXXX /home/mparra/` where `XXXXXXXX` is the UID returned by item `8`
9. Check if IPA works. Exit of the server and try to connect: `ssh manuparra@192.168.10.220` If it is working, ssh ask to you about change your password and retype it twice. If you can access to the server, IPA server is Working.



