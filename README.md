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

## Installing Master IPA Server.

1. First of all, execute command: 
`yum -y update`
2. Set the name of the IPA Server: 
`hostnamectl set-hostname ipa.centos.local`
3. Edit `/etc/hosts` and add:
`192.168.10.200 ipa.centos.local ipa`




