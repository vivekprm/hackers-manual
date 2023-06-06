You can log onto the machine using RDP with the following credentials: admin/password using the xfreerdp command: 
```sh
xfreerdp /u:admin /p:password /cert:ignore /v:10.10.49.205 /workarea
```

If Windows prompts you to choose a location for your network, choose the "Home" option.

On your Desktop there should be a folder called "vulnerable-apps". Inside this folder are a number of binaries which are vulnerable to simple stack based buffer 
overflows (the type taught on the PWK/OSCP course):

- The SLMail installer.
- The brainpan binary.
- The dostackbufferoverflowgood binary.
- The vulnserver binary.
- A custom written "oscp" binary which contains 10 buffer overflows, each with a different EIP offset and set of badchars.

I have also written a handy guide to exploiting buffer overflows with the help of 
mona: https://github.com/Tib3rius/Pentest-Cheatsheets/blob/master/exploits/buffer-overflows.rst

Please note that this room does not teach buffer overflows from scratch. It is intended to help OSCP students and also bring to their attention some features of mona 
which will save time in the OSCP exam.

Thanks go to [@Mojodojo_101](https://twitter.com/Mojodojo_101) for helping create the custom oscp.exe binary for this room!
