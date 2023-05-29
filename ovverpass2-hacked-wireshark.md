Open WireShark and load the PCAP that was just downloaded. Look for a HTTP packet with POST request to see what was the page that was used to upload a reverse shell. Right-click on 
the HTTP packet, then click Follow, then HTTP stream:

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/1c22c104-6cb7-4430-9721-12962032758e)

What was the URL of the page they used to upload a reverse shell?
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/a8fb36fb-cb2e-4fe3-a317-2e9db7e726b2)

From the same HTTP Stream, we can answer the second question.

What payload did the attacker used to gain access?
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/61121aa5-9116-4023-97c2-79e09b799b07)

Next is to look for TCP packet with PSH, ACK flags as they are signs of more data getting transmitted, so mostly a sign of persistence for me. Right-click on the packet you think is interesting 
and choose Follow, and then TCP Stream:

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/cda751b3-c7f9-4daa-9399-be027e5bb8bb)

What password did the attacker use to privesc?
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/d8802de5-0174-4dfd-8665-ecd3700fbfb0)

On the same TCP stream where we found James’ password, you can find what the attacker used for persistence.

How did the attacker establish persistence?
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/708cab88-850c-4a61-926b-bde22e3e424a)

On the same TCP Stream, we can see the list of users through the /etc/shadow with their hashed passwords. First, put the entries you found from /etc/shadow to a file and use John the Ripper to 
crack the hashed passwords. 

Type ```sudo john –wordlist=/usr/share/wordlists/fasttrack.txt foundusers.txt```

- **–wordlist=/usr/share/wordlists/fasttrack.txt** – we were asked to use the wordlist fasttrack.txt. By using the –wordlist option, we are telling John to use a specific file instead of using its 
default john.lst 
- **foundusers.txt** – is the name I gave the file that has the list of users and hashed passwords found in /etc/shadow

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/5b637179-2fc1-438f-be7a-bc2adcd53c38)

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/23244a61-74dc-4faf-bb3b-5181d16857aa)

Using fasttrack wordlist, how many of the system passwords were crackable? We will have to use the -show option to display all of the cracked passwords. Type sudo john -show foundusers.txt
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/320e9b5c-8d79-4e84-8f0b-183b075c4182)

The next three questions can be answered by analyzing the script that was used for the persistent connection. Google the answer to question number 4 to see the code and answer the next questions

What’s the default hash for the backdoor?
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/682df8d7-995e-47a4-853e-346ba254305c)

What’s the hardcoded salt for the backdoor?
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/54742922-3bb3-4565-991f-5a0376f5d1b1)

What was the hash that the attacker used? We will have to go back to the PCAP file we downloaded at the beginning of this write-up and go to the same TCP stream that we analyzed to see the hash 
that was used in the attack:
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/3ae1aed3-6aac-43f2-bb72-e95e8a874195)

Now, let’s crack the hash to see what was the password so we can use it to SSH on to the victim machine.

First, we have to analyze which hash was used in conjunction with the hard-coded salt to retrieve our target password. There are only two hashes to choose from, and unluckily, I chose the wrong 
hash as my first trial and error. So, I put one of the hashes and the salt together in a $pass:$salt format. And used a hash analyzer, I was able to identify that it was using SHA512 algorithm.
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/539477b7-0655-4305-9d38-905931e3958b)

With the knowledge that the password was hashed using SHA512 algorithm, and it was salted, and uses the $pass:$salt format, I visited hash examples to check what hash mode I have to use when 
using Hashcat to crack it

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/52041a28-fe30-4654-9064-5399f0e1dc56)

- I combined the hash that was used and the hard-coded salt into one line and saved the file as hash.txt (you can name it whatever you want)
- Run Hashcat to crack the hash.txt we saved just above this line. I used a different machine with better GPU to run hashcat. Type 
```
hashcat -m 1710 -o results.txt hash.txt /usr/share/wordlists/rockyou.txt
```
In our case hash.txt contains:
```
6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed:1c362db832f3f864c8c2fe05f2002a05
```
- ```-m 1710``` – This is to tell hashcat to crack a file that was hashed with SHA512 algorithm and in a $pass:$salt format
- ```-o results.txt``` – is where I want the cracked hash to get redirected
- ```hash.txt``` – is the file where the hashed and salted password are put together in one line
- ```/usr/share/wordlists/rockyou.txt``` – we were instructed to use the rockyou wordlists to crack the hash

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/98bf28c2-3379-442a-9191-58b06268860c)

Crack the hash using rockyou and a cracking tool of your choice. What’s the password?
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/5db7c958-774e-4fbc-a41b-0ca906efc1ff)

The attacker defaced the website. What message did they leave as a heading? Just visit the site by typing the IP address on the URL bar of a browser.
![image](https://github.com/vivekprm/hackers-manual/assets/2403660/bed900c8-5a6f-4423-92df-b72ac7197ae5)

Now, let’s SSH into the victim machine using the password that we found in question 9. Type ```ssh james@10.10.10.128 -p 2222```
- james – we are using the username james to authenticate through SSH
- ```-p 2222``` – is to specify that we want to use port 2222 and not the default port 22 for SSH. From the nmap scan result, we saw that there are 2 open SSH service, and it just makes sense that a 
backdoor will use a non-default port to gain access

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/9f2c4aad-42a4-40f2-9f5e-c9f6ed1f7995)

Not to do previlege escalation. Let's look at files with suid bit set
```sh
find / 2>/dev/null -perm -u=s 
```

So we see a suspicious excecutable with suid permission , .suid_bash
Running it gives us root access.
