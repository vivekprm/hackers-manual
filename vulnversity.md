# Reconnaissance
Gather information about the machine using a network scanning tool called Nmap. 

![image](https://user-images.githubusercontent.com/2403660/236682740-3e8c8948-b650-4bcb-8f80-24003f03717e.png)

## Locating directories using Gobuster
Using a fast directory discovery tool called ```Gobuster```, you will locate a directory to which you can use to upload a shell.

Gobuster is a tool used to brute-force URIs (directories and files), DNS subdomains, and virtual host names. For this machine, we will focus on using it to brute-force directories.

Download Gobuster [here](https://github.com/OJ/gobuster), or if you're on Kali Linux run ```sudo apt-get install gobuster```

To get started, you will need a wordlist for Gobuster (which will be used to quickly go through the wordlist to identify if a public directory is available. If you are using Kali Linux, you can find many wordlists under /usr/share/wordlists. You can also use the wordlist for directories located at /usr/share/wordlists/dirbuster/directory-list-1.0.txt in the AttackBox.

Now let's run Gobuster with a wordlist using ```gobuster dir -u http://10.10.56.80:3333 -w <word list location>```

![image](https://user-images.githubusercontent.com/2403660/236683237-497f5b3b-5937-4ace-ab3d-bafb679dc83b.png)

After running gobuster we might find few directories where we can upload files.
```sh
gobuster dir -u http://10.10.56.80:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

E.g. in this case /internal

## Compromise the web server
We will fuzz the upload form to identify which extensions are not blocked.

To do this, we're going to use BurpSuite's intruder module.


We're going to use Intruder (used for automating customised attacks).

To begin, make a wordlist with the following extensions:

- .php
- .php3
- .php4
- .php5
- .phtml

![image](https://user-images.githubusercontent.com/2403660/236683633-f8a3597d-8e89-4a4a-b2a6-db588c9e2fb6.png)

Now make sure BurpSuite is configured to intercept all your browser traffic. Upload a file; once this request is captured, send it to the Intruder. 
Click on "Payloads" and select the "Sniper" attack type.

Click the "Positions" tab now, find the filename and "Add §" to the extension. It should look like so:

![image](https://user-images.githubusercontent.com/2403660/236683657-feac011e-ded1-48df-b9b8-4cf0833bf0b3.png)

Run this attack, what extension is allowed?

We are going to use a PHP reverse shell as our payload. A reverse shell works by being called on the remote host and forcing this host to make a 
connection to you. So you'll listen for incoming connections, upload and execute your shell, which will beacon out to you to control!

Download the following reverse PHP shell [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

To gain remote access to this machine, follow these steps:

- Edit the php-reverse-shell.php file and edit the ip to be your tun0 ip (you can get this by going to http://10.10.10.10 in the browser of your 
TryHackMe connected device).
- Rename this file to php-reverse-shell.phtml
- We're now going to listen to incoming connections using netcat. Run the following command: ```nc -lvnp 1234```

Upload your shell and navigate to http://10.10.56.80:3333/internal/uploads/php-reverse-shell.phtml - This will execute your payload

You should see a connection on your Netcat session
