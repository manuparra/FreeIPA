# FreeIPA installation and deployment of the replica multimaster 
FreeIPA installation Tutorial, Scripts and Procedures

## Planning the server and architecture to deploy. Requirements.

In the IPA domain, there are three types of machines:

+ Servers, which manage all of the services used by domain members
+ Replicas, which are essentially copies of servers (and, once copied, are identical to serve)
+ Clients, which connect to the Servers to authentication and authorization

In our deployment we will create the next architecture:

Inline-style: 
![alt text](https://github.com/manuparra/FreeIPA/raw/master/achitecture.png "Logo Title Text 1")

