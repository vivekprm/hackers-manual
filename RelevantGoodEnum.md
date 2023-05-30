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

