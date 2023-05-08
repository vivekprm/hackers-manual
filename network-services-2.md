# Understanding NFS
NFS stands for "Network File System" and allows a system to share directories and files with others over a network. By using NFS, users and programs 
can access files on remote systems almost as if they were local files. It does this by mounting all, or a portion of a file system on a server. The 
portion of the file system that is mounted can be accessed by clients with whatever privileges are assigned to each file.

## How does NFS work?
We don't need to understand the technical exchange in too much detail to be able to exploit NFS effectively- however if this is something that interests 
you, I would recommend this resource: https://docs.oracle.com/cd/E19683-01/816-4882/6mb2ipq7l/index.html

First, the client will request to mount a directory from a remote host on a local directory just the same way it can mount a physical device. The mount 
service will then act to connect to the relevant mount daemon using RPC.

The server checks if the user has permission to mount whatever directory has been requested. It will then return a file handle which uniquely identifies 
each file and directory that is on the server.

If someone wants to access a file using NFS, an RPC call is placed to NFSD (the NFS daemon) on the server. This call takes parameters such as:
- The file handle
- The name of the file to be accessed
- The user's, user ID
- The user's group ID

These are used in determining access rights to the specified file. This is what controls user permissions, I.E read and write of files.

## What runs NFS?
Using the NFS protocol, you can transfer files between computers running Windows and other non-Windows operating systems, such as Linux, MacOS or UNIX.

A computer running Windows Server can act as an NFS file server for other non-Windows client computers. Likewise, NFS allows a Windows-based computer 
running Windows Server to access files stored on a non-Windows NFS server.

## More Information
Here are some resources that explain the technical implementation, and working of, NFS in more detail than I have covered here.

https://www.datto.com/library/what-is-nfs-file-share
http://nfs.sourceforge.net/
https://wiki.archlinux.org/index.php/NFS

