# Understanding NFS
NFS stands for "Network File System" and allows a system to share directories and files with others over a network. By using NFS, users and programs 
can access files on remote systems almost as if they were local files. It does this by mounting all, or a portion of a file system on a server. The 
portion of the file system that is mounted can be accessed by clients with whatever privileges are assigned to each file.

## How does NFS work?
We don't need to understand the technical exchange in too much detail to be able to exploit NFS effectively- however if this is something that interests 
you, I would recommend this resource: https://docs.oracle.com/cd/E19683-01/816-4882/6mb2ipq7l/index.html

First, the client will request to mount a directory from a remote host on a local directory just the same way it can mount a physical device. The mount 
service will then act to connect to the relevant mount daemon using RPC.

The server checks if the user has permission to mount whatever directory has been requested. It will then return a file handle which uniquely identifies 
each file and directory that is on the server.

If someone wants to access a file using NFS, an RPC call is placed to NFSD (the NFS daemon) on the server. This call takes parameters such as:
- The file handle
- The name of the file to be accessed
- The user's, user ID
- The user's group ID

These are used in determining access rights to the specified file. This is what controls user permissions, I.E read and write of files.

## What runs NFS?
Using the NFS protocol, you can transfer files between computers running Windows and other non-Windows operating systems, such as Linux, MacOS or UNIX.

A computer running Windows Server can act as an NFS file server for other non-Windows client computers. Likewise, NFS allows a Windows-based computer 
running Windows Server to access files stored on a non-Windows NFS server.

## More Information
Here are some resources that explain the technical implementation, and working of, NFS in more detail than I have covered here.

https://www.datto.com/library/what-is-nfs-file-share
http://nfs.sourceforge.net/
https://wiki.archlinux.org/index.php/NFS

# Enumerating NFS
In order to do a more advanced enumeration of the NFS server, and shares- we're going to need a few tools. The first of which is key to interacting with any NFS share from your local machine: nfs-common.

## NFS-Common
It is important to have this package installed on any machine that uses NFS, either as client or server. It includes programs such as: lockd, statd, showmount, nfsstat, gssd, idmapd and mount.nfs. Primarily, we are concerned with "showmount" and "mount.nfs" as these are going to be most useful to us when it comes to extracting information from the NFS share. If you'd like more information about this package, feel free to read: https://packages.ubuntu.com/jammy/nfs-common.

You can install nfs-common using "sudo apt install nfs-common".

## Port Scanning
The first step of enumeration is to conduct a port scan, to find out as much information as you can about the services, open ports and operating system of the target machine. You can go as in-depth as you like on this, however, I suggest using nmap with the -A and -p- tags.

## Mounting NFS shares
Your client’s system needs a directory where all the content shared by the host server in the export folder can be accessed. You can create
this folder anywhere on your system. Once you've created this mount point, you can use the "mount" command to connect the NFS share to the mount point on your machine like so:
```sh
sudo mount -t nfs IP:share /tmp/mount/ -nolock
```

Let's break this down

**Tag**	  **Function**
sudo	    Run as root
mount	    Execute the mount command
-t nfs	  Type of device to mount, then specifying that it's NFS
IP:share	The IP Address of the NFS server, and the name of the share we wish to mount
-nolock	  Specifies not to use NLM locking

E.g.
```sh
mount -t nfs 10.10.210.59:/home /tmp/mount -nolock
```

# Exploiting NFS
## What is root_squash?
By default, on NFS shares- Root Squashing is enabled, and prevents anyone connecting to the NFS share from having root access to the NFS volume. Remote root users are assigned a user “nfsnobody” when connected, which has the least local privileges. Not what we want. However, if this is turned off, it can allow the creation of SUID bit files, allowing a remote user root access to the connected system.

## SUID
So, what are files with the SUID bit set? Essentially, this means that the file or files can be run with the permissions of the file(s) owner/group. In this case, as the super-user. We can leverage this to get a shell with these privileges!

## Method
This sounds complicated, but really- provided you're familiar with how SUID files work, it's fairly easy to understand. We're able to upload files to the NFS share, and control the permissions of these files. We can set the permissions of whatever we upload, in this case a bash shell executable. We can then log in through SSH, as we did in the previous task- and execute this executable to gain a root shell!

## The Executable
Due to compatibility reasons, we'll use a standard Ubuntu Server 18.04 bash executable, the same as the server's- as we know from our nmap scan. You can download it here. If you want to download it via the command line, be careful not to download the github page instead of the raw script. You can use wget https://github.com/polo-sec/writing/raw/master/Security%20Challenge%20Walkthroughs/Networks%202/bash.

## Mapped Out Pathway:
If this is still hard to follow, here's a step by step of the actions we're taking, and how they all tie together to allow us to gain a root shell:

    NFS Access ->
        Gain Low Privilege Shell ->
            Upload Bash Executable to the NFS share ->
                Set SUID Permissions Through NFS Due To Misconfigured Root Squash ->
                    Login through SSH ->
                        Execute SUID Bit Bash Executable ->
                            ROOT ACCESS



Let's do a sanity check, let's check the permissions of the "bash" executable using "ls -la bash". What does the permission set look like? Make sure that it ends with -sr-x.
```sh
bash-4.4$ ls -ltr
total 1088
-rwsr-sr-x 1 root root 1113504 May  8 14:21 bash
```

SSH into the machine as the user. List the directory to make sure the bash executable is there. Now, the moment of truth. Lets run it with "./bash -p". The -p persists the permissions, so that it can run as root with SUID- as otherwise bash will sometimes drop the permissions.

# Understanding SMTP
## What is SMTP?
SMTP stands for "Simple Mail Transfer Protocol". It is utilised to handle the sending of emails. In order to support email services, a protocol pair is required, comprising of SMTP and POP/IMAP. Together they allow the user to send outgoing mail and retrieve incoming mail, respectively.

The SMTP server performs three basic functions:
- It verifies who is sending emails through the SMTP server.
- It sends the outgoing mail
- If the outgoing mail can't be delivered it sends the message back to the sender

Most people will have encountered SMTP when configuring a new email address on some third-party email clients, such as Thunderbird; as when you configure a new email client, you will need to configure the SMTP server configuration in order to send outgoing emails.

## POP and IMAP
POP, or "Post Office Protocol" and IMAP, "Internet Message Access Protocol" are both email protocols who are responsible for the transfer of email between a client and a mail server. The main differences is in POP's more simplistic approach of downloading the inbox from the mail server, to the client. Where IMAP will synchronise the current inbox, with new mail on the server, downloading anything new. This means that changes to the inbox made on one computer, over IMAP, will persist if you then synchronise the inbox from another computer. The POP/IMAP server is responsible for fulfiling this process.

## How does SMTP work?
Email delivery functions much the same as the physical mail delivery system. The user will supply the email (a letter) and a service (the postal delivery service), and through a series of steps- will deliver it to the recipients inbox (postbox). The role of the SMTP server in this service, is to act as the sorting office, the email (letter) is picked up and sent to this server, which then directs it to the recipient.
We can map the journey of an email from your computer to the recipient’s like this:

![image](https://user-images.githubusercontent.com/2403660/236851772-0dec6395-4e40-465a-b772-659d80270da7.png)

1. The mail user agent, which is either your email client or an external program. connects to the SMTP server of your domain, e.g. smtp.google.com. This initiates the SMTP handshake. This connection works over the SMTP port- which is usually 25. Once these connections have been made and validated, the SMTP session starts.

2. The process of sending mail can now begin. The client first submits the sender, and recipient's email address- the body of the email and any attachments, to the server.

3. The SMTP server then checks whether the domain name of the recipient and the sender is the same.

4. The SMTP server of the sender will make a connection to the recipient's SMTP server before relaying the email. If the recipient's server can't be accessed, or is not available- the Email gets put into an SMTP queue.

5. Then, the recipient's SMTP server will verify the incoming email. It does this by checking if the domain and user name have been recognised. The server will then forward the email to the POP or IMAP server, as shown in the diagram above.

6. The E-Mail will then show up in the recipient's inbox.

This is a very simplified version of the process, and there are a lot of sub-protocols, communications and details that haven't been included. If you're looking to learn more about this topic, this is a really friendly to read breakdown of the finer technical details- I actually used it to write this breakdown:

https://computer.howstuffworks.com/e-mail-messaging/email3.htm

## What runs SMTP?
SMTP Server software is readily available on Windows server platforms, with many other variants of SMTP being available to run on Linux.

## More Information:
Here is a resource that explain the technical implementation, and working of, SMTP in more detail than I have covered here.

https://www.afternerd.com/blog/smtp/

# Enumerating SMTP
## Enumerating Server Details
Poorly configured or vulnerable mail servers can often provide an initial foothold into a network, but prior to launching an attack, we want to fingerprint the server to make our targeting as precise as possible. **We're going to use the "smtp_version" module in MetaSploit** to do this. As its name implies, it will scan a range of IP addresses and determine the version of any mail servers it encounters.

## Enumerating Users from SMTP
The SMTP service has two internal commands that allow the enumeration of users: ```VRFY``` (confirming the names of valid users) and ```EXPN``` (which reveals the actual address of user’s aliases and lists of e-mail (mailing lists). Using these SMTP commands, we can reveal a list of valid users

We can do this manually, over a telnet connection- however Metasploit comes to the rescue again, providing a handy module appropriately called **"smtp_enum"** that will do the legwork for us! Using the module is a simple matter of feeding it a host or range of hosts to scan and a wordlist containing usernames to enumerate.

## Requirements
As we're going to be using Metasploit for this, it's important that you have Metasploit installed. It's always worth doing a quick update to make sure that you're on the latest version before launching any attacks. You can do this with a simple "sudo apt update", and accompanying upgrade- if any are required.

## Alternatives
It's worth noting that this enumeration technique will work for the majority of SMTP configurations; however there are other, non-metasploit tools such as **smtp-user-enum** that work even better for enumerating OS-level user accounts on Solaris via the SMTP service. Enumeration is performed by inspecting the responses to ```VRFY```, ```EXPN```, and ```RCPT TO``` commands.

This technique could be adapted in future to work against other vulnerable SMTP daemons, but this hasn’t been done as of the time of writing. It's an alternative that's worth keeping in mind if you're trying to distance yourself from using Metasploit e.g. in preparation for OSCP.

```sh
msf6 auxiliary(scanner/smtp/smtp_version) > set RHOSTS 10.10.49.47
RHOSTS => 10.10.49.47
msf6 auxiliary(scanner/smtp/smtp_version) > set THREADS 3
THREADS => 3
msf6 auxiliary(scanner/smtp/smtp_version) > run

[+] 10.10.49.47:25        - 10.10.49.47:25 SMTP 220 polosmtp.home ESMTP Postfix (Ubuntu)\x0d\x0a
```

```sh
msf6 auxiliary(scanner/smtp/smtp_enum) > set RHOSTS 10.10.49.47
RHOSTS => 10.10.49.47
msf6 auxiliary(scanner/smtp/smtp_enum) > set USER_FILE /usr/share/wordlists/SecLists/Usernames/top-usernames-shortlist.txt
USER_FILE => /usr/share/wordlists/SecLists/Usernames/top-usernames-shortlist.txt
msf6 auxiliary(scanner/smtp/smtp_enum) > exploit
```

# Exploiting SMTP
## What do we know?
Okay, at the end of our Enumeration section we have a few vital pieces of information:

1. A user account name
2. The type of SMTP server and Operating System running.

We know from our port scan, that the only other open port on this machine is an SSH login. We're going to use this information to try and bruteforce the password of the SSH login for our user using Hydra.

## Preparation
It's advisable that you exit Metasploit to continue the exploitation of this section of the room. Secondly, it's useful to keep a note of the information you gathered during the enumeration stage, to aid in the exploitation.

## Hydra
There is a wide array of customisability when it comes to using Hydra, and it allows for adaptive password attacks against of many different services, including SSH. Hydra comes by default on both Parrot and Kali, however if you need it, you can find the GitHub [here](https://github.com/vanhauser-thc/thc-hydra).

Hydra uses dictionary attacks primarily, both Kali Linux and Parrot OS have many different wordlists in the "/usr/share/wordlists" directory- if you'd like to browse and find a different wordlists to the widely used "rockyou.txt". Likewise I recommend checking out SecLists for a wider array of other wordlists that are extremely useful for all sorts of purposes, other than just password cracking. E.g. subdomain enumeration
The syntax for the command we're going to use to find the passwords is this:
```sh
hydra -t 16 -l USERNAME -P /usr/share/wordlists/rockyou.txt -vV 10.10.49.47 ssh
```
Let's break it down:

SECTION	                FUNCTION
hydra	                  Runs the hydra tool
-t 16                   Number of parallel connections per target
-l [user]	              Points to the user who's account you're trying to compromise
-P [path to dictionary]	Points to the file containing the list of possible passwords
-vV                     Sets verbose mode to very verbose, shows the login+pass combination for each attempt
[machine IP]	          The IP address of the target machine
ssh / protocol	        Sets the protocol

# Understanding MySQL
## How does MySQL work?
MySQL, as an RDBMS, is made up of the server and utility programs that help in the administration of MySQL databases.

The server handles all database instructions like creating, editing, and accessing data. It takes and manages these requests and communicates using the MySQL protocol. This whole process can be broken down into these stages:
- MySQL creates a database for storing and manipulating data, defining the relationship of each table.
- Clients make requests by making specific statements in SQL.
- The server will respond to the client with whatever information has been requested.

## What runs MySQL?
MySQL can run on various platforms, whether it's Linux or windows. It is commonly used as a back end database for many prominent websites and forms an essential component of the LAMP stack, which includes: Linux, Apache, MySQL, and PHP.

## More Information:
Here are some resources that explain the technical implementation, and working of, MySQL in more detail than I have covered here:

https://dev.mysql.com/doc/dev/mysql-server/latest/PAGE_SQL_EXECUTION.html 
https://www.w3schools.com/php/php_mysql_intro.asp

# Enumerating MySQL
## When you would begin attacking MySQL

MySQL is likely not going to be the first point of call when getting initial information about the server. You can, as we have in previous tasks, attempt to brute-force default account passwords if you really don't have any other information; however, in most CTF scenarios, this is unlikely to be the avenue you're meant to pursue.

## The Scenario
Typically, you will have gained some initial credentials from enumerating other services that you can then use to enumerate and exploit the MySQL service. As this room focuses on exploiting and enumerating the network service, for the sake of the scenario, we're going to assume that you found the credentials: "root:password" while enumerating subdomains of a web server. After trying the login against SSH unsuccessfully, you decide to try it against MySQL.

## Requirements
You will want to have MySQL installed on your system to connect to the remote MySQL server. In case this isn't already installed, you can install it using sudo apt install default-mysql-client. Don't worry- this won't install the server package on your system- just the client.

Again, we're going to be using Metasploit for this; it's important that you have Metasploit installed, as it is by default on both Kali Linux and Parrot OS.

## Alternatives
As with the previous task, it's worth noting that everything we will be doing using Metasploit can also be done either manually or with a set of non-Metasploit tools such as **nmap's mysql-enum** script: https://nmap.org/nsedoc/scripts/mysql-enum.html or https://www.exploit-db.com/exploits/23081.

# Exploiting MySQL
Let's take a sanity check before moving on to try and exploit the database fully, and gain more sensitive information than just database names. We know:

1. MySQL server credentials
2. The version of MySQL running
3. The number of Databases, and their names.

## Key Terminology
In order to understand the exploits we're going to use next- we need to understand a few key terms.

### Schema:
In MySQL, physically, a *schema* is synonymous with a *database*. You can substitute the keyword "SCHEMA" instead of DATABASE in MySQL SQL syntax, for example using CREATE SCHEMA instead of CREATE DATABASE. It's important to understand this relationship because some other database products draw a distinction. For example, in the Oracle Database product, a schema represents only a part of a database: the tables and other objects owned by a single user.

### Hashes:
Hashes are, very simply, the product of a cryptographic algorithm to turn a variable length input into a fixed length output.

In MySQL hashes can be used in different ways, for instance to index data into a hash table. Each hash has a unique ID that serves as a pointer to the original data. This creates an index that is significantly smaller than the original data, allowing the values to be searched and accessed more efficiently

However, the data we're going to be extracting are password hashes which are simply a way of storing passwords not in plaintext format.

## Further Reading
Here's some things that might be useful to read after completing this room, if it interests you:

https://web.mit.edu/rhel-doc/4/RH-DOCS/rhel-sg-en-4/ch-exploits.html
https://www.nextgov.com/cybersecurity/2019/10/nsa-warns-vulnerabilities-multiple-vpn-services/160456/
