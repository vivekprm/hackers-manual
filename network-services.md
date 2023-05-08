WiFi Hacking 101 by NinjaJc01 (https://tryhackme.com/room/wifihacking101)

# Understanding SMB
SMB - Server Message Block Protocol - is a client-server communication protocol used for sharing access to files, printers, serial ports and other 
resources on a network.

Servers make file systems and other resources (printers, named pipes, APIs) available to clients on the network. Client computers may have their own hard 
disks, but they also want access to the shared file systems and printers on the servers.

The SMB protocol is known as a response-request protocol, meaning that it transmits multiple messages between the client and server to establish a 
connection. Clients connect to servers using TCP/IP (actually NetBIOS over TCP/IP as specified in RFC1001 and RFC1002), NetBEUI or IPX/SPX.

## How does SMB work?
![image](https://user-images.githubusercontent.com/2403660/236742092-0eeacbd4-d0e2-4c9b-8eb2-53d62868f0c5.png)

Once they have established a connection, clients can then send commands (SMBs) to the server that allow them to access shares, open files, read and write 
files, and generally do all the sort of things that you want to do with a file system. However, in the case of SMB, these things are done over the network.

## What runs SMB?
Microsoft Windows operating systems since Windows 95 have included client and server SMB protocol support. Samba, an open source server that supports the 
SMB protocol, was released for Unix systems.

# Enumerating SMB
Enumeration is the process of gathering information on a target in order to find potential attack vectors and aid in exploitation.

This process is essential for an attack to be successful, as wasting time with exploits that either don't work or can crash the system can be a waste of 
energy. Enumeration can be used to gather usernames, passwords, network information, hostnames, application data, services, or any other information that 
may be valuable to an attacker.

Typically, there are SMB share drives on a server that can be connected to and used to view or transfer files. SMB can often be a great starting point for 
an attacker looking to discover sensitive information — you'd be surprised what is sometimes included on these shares.

## Enum4Linux
Enum4linux is a tool used to enumerate SMB shares on both Windows and Linux systems. It is basically a wrapper around the tools in the Samba package and 
makes it easy to quickly extract information from the target pertaining to SMB. It's installed by default on Parrot and Kali, however if you need to 
install it, you can do so from the official [github](https://github.com/portcullislabs/enum4linux).

The syntax of Enum4Linux is nice and simple: "enum4linux [options] ip"

TAG            FUNCTION

-U             get userlist
-M             get machine list
-N             get namelist dump (different from -U and-M)
-S             get sharelist
-P             get password policy information
-G             get group and member list

-a             all of the above (full basic enumeration)

# Exploiting SMB
While there are vulnerabilities such as [CVE-2017-7494](https://www.cvedetails.com/cve/CVE-2017-7494/) that can allow remote code execution by exploiting SMB, you're more likely to encounter a situation where the best way into a system is due to misconfigurations in the system. In this case, we're going to be exploiting anonymous SMB share access- a common misconfiguration that can allow us to gain information that will lead to a shell.

## Method Breakdown
So, from our enumeration stage, we know:
- The SMB share location
- The name of an interesting SMB share

## SMBClient
Because we're trying to access an SMB share, we need a client to access resources on servers. We will be using SMBClient because it's part of the default samba suite. While it is available by default on Kali and Parrot, if you do need to install it, you can find the documentation [here](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html).

We can remotely access the SMB share using the syntax:
```
smbclient //[IP]/[SHARE]
```

Followed by the tags:

-U [name] : to specify the user
-p [port] : to specify the port

# Understanding Telnet
Telnet is an application protocol which allows you, with the use of a telnet client, to connect to and execute commands on a remote machine that's hosting a telnet server.

The telnet client will establish a connection with the server. The client will then become a virtual terminal- allowing you to interact with the remote host.

Telnet sends all messages in clear text and has no specific security mechanisms. Thus, in many applications and services, Telnet has been replaced by SSH in most implementations.
 
## How does Telnet work?
The user connects to the server by using the Telnet protocol, which means entering "telnet" into a command prompt. The user then executes commands on the server by using specific Telnet commands in the Telnet prompt. You can connect to a telnet server with the following syntax: "telnet [ip] [port]"

# Exploiting Telnet
Telnet, being a protocol, is in and of itself insecure for the reasons we talked about earlier. It lacks encryption, so sends all communication over plaintext, and for the most part has poor access control. There are CVE's for Telnet client and server systems, however, so when exploiting you can check for those on:

https://www.cvedetails.com/
https://cve.mitre.org/

A CVE, short for Common Vulnerabilities and Exposures, is a list of publicly disclosed computer security flaws. When someone refers to a CVE, they usually mean the CVE ID number assigned to a security flaw.

However, you're far more likely to find a misconfiguration in how telnet has been configured or is operating that will allow you to exploit it.

## Method Breakdown
So, from our enumeration stage, we know:

- There is a poorly hidden telnet service running on this machine
- The service itself is marked "backdoor"
- We have possible username of "Skidy" implicated

Using this information, let's try accessing this telnet port, and using that as a foothold to get a full reverse shell on the machine!

## Connecting to Telnet
You can connect to a telnet server with the following syntax:

    "telnet [ip] [port]"

We're going to need to keep this in mind as we try and exploit this machine.

## What is a Reverse Shell?
A "shell" can simply be described as a piece of code or program which can be used to gain code or command execution on a device.

![image](https://user-images.githubusercontent.com/2403660/236754618-1b7febc0-97ab-4a78-860b-2ef13ea6573c.png)

A reverse shell is a type of shell in which the target machine communicates back to the attacking machine.

The attacking machine has a listening port, on which it receives the connection, resulting in code or command execution being achieved.
