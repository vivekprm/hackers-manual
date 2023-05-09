# Samba
Samba is the standard Windows interoperability suite of programs for Linux and Unix. It allows end users to access and use files, printers and other 
commonly shared resources on a companies intranet or internet. Its often referred to as a network file system.

Samba is based on the common client/server protocol of Server Message Block (SMB). SMB is developed only for Windows, without Samba, other computer 
platforms would be isolated from Windows machines, even if they were part of the same network.

Using nmap we can enumerate a machine for SMB shares.

Nmap has the ability to run to automate a wide variety of networking tasks. There is a script to enumerate shares!
```sh
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse MACHINE_IP
```

SMB has two ports, 445 and 139.
![image](https://user-images.githubusercontent.com/2403660/237002989-0c8c3c8e-a397-4c9e-a66c-fa03dc4594f3.png)

On most distributions of Linux smbclient is already installed. Lets inspect one of the shares.
```sh
smbclient //MACHINE_IP/anonymous
```

Using your machine, connect to the machines network share.

![image](https://user-images.githubusercontent.com/2403660/237003100-9686436f-f39f-487f-a3b5-e580dc684c01.png)

You can recursively download the SMB share too. Submit the username and password as nothing.
```sh
smbget -R smb://MACHINE_IP/anonymous
```

Open the file on the share. There is a few interesting things found.
- Information generated for Kenobi when generating an SSH key for the user
- Information about the ProFTPD server.


Your earlier nmap port scan will have shown port 111 running the service ```rpcbind```. This is just a server that converts remote procedure call (RPC) 
program number into universal addresses. When an RPC service is started, it tells ```rpcbind``` the address at which it is listening and the RPC program 
number its prepared to serve. 

In our case, port 111 is access to a network file system. Lets use nmap to enumerate this.
```sh
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount MACHINE_IP
```

We see /var mount.

# Gain initial access with ProFtpd
ProFtpd is a free and open-source FTP server, compatible with Unix and Windows systems. Its also been vulnerable in the past software versions.

We can use searchsploit to find exploits for a particular software version. Searchsploit is basically just a command line search tool for exploit-db.com.

You should have found an exploit from ProFtpd's ```mod_copy``` module, which is used to exploit version 1.3.5.

The ```mod_copy``` module implements ```SITE CPFR``` and ```SITE CPTO``` commands, which can be used to copy files/directories from one place to another 
on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination.

FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user. 

We're now going to copy Kenobi's private key using ```SITE CPFR``` and ```SITE CPTO``` commands.

![image](https://user-images.githubusercontent.com/2403660/237004928-18780464-216d-4001-8d4d-07b3cd081780.png)

Lets mount the /var/tmp directory to our machine
```sh
mkdir /mnt/kenobiNFS
mount MACHINE_IP:/var /mnt/kenobiNFS
ls -la /mnt/kenobiNFS
```

![image](https://user-images.githubusercontent.com/2403660/237006096-379a1ec3-d35f-40ce-8a0e-83ac2379e087.png)

We now have a network mount on our deployed machine! We can go to /var/tmp and get the private key then login to Kenobi's account.
![image](https://user-images.githubusercontent.com/2403660/237006313-ee3ddccf-7a6c-422d-8979-f3f6c9ea8071.png)


