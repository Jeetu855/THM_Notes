
NFS stands for "Network File System" and allows a system to share directories and files with others over a network. By using NFS, users and programs can access files on remote systems almost as if they were local files. It does this by mounting all, or a portion of a file system on a server. The portion of the file system that is mounted can be accessed by clients with whatever privileges are assigned to each file.

First, ***the client will request to mount a directory from a remote host on a local directory just the same way it can mount a physical device. The mount service will then act to connect to the relevant mount daemon using RPC(Remote Procedure Call).***

The server checks if the user has permission to mount whatever directory has been requested. It will then return a file handle which uniquely identifies each file and directory that is on the server.

***If someone wants to access a file using NFS, an RPC call is placed to NFSD (the NFS daemon) on the server.*** This call takes parameters such as:

-  The file handle
-  The name of the file to be accessed
-  The user's, user ID
-  The user's group ID

Using the NFS protocol, you can transfer files between computers running Windows and other non-Windows operating systems, such as Linux, MacOS or UNIX.

A computer running Windows Server can act as an NFS file server for other non-Windows client computers. Likewise, NFS allows a Windows-based computer running Windows Server to access files stored on a non-Windows NFS server.


#### Enumerating NFS

Port Scanning 
Version Fingerprinting

**Mounting NFS shares**

Your client’s system needs a directory where all the content shared by the host server in the export folder can be accessed. You can create  
this folder anywhere on your system. Once you've created this mount point, you can use the "mount" command to connect the NFS share to the mount point on your machine

To list the NFS share

```sh
showmount -e IPofNFSshare
```

To mount the share on our system

```sh
sudo mount -t nfs IP:share /tmp/mount/ -nolock
```

|   |   |
|---|---|
|sudo|Run as root|
|mount|Execute the mount command|
|-t nfs|Type of device to mount, then specifying that it's NFS|
|IP:share|The IP Address of the NFS server, and the name of the share we wish to mount|
|-nolock|Specifies not to use NLM locking|


Example : If NFS is sharing the home directory
To mount it

```sh
sudo mount -t nfs 10.10.245.213:/home /tmp/mount
```

**id_rsa →**this is the private key of the user which used to login to the system

**id_rsa.pub** →this is the key that used send others to encrypt when they send message to us.
#### Exploiting NFS

By default, on NFS shares- Root Squashing is enabled, and prevents anyone connecting to the NFS share from having root access to the NFS volume. Remote root users are assigned a user “nfsnobody” when connected, which has the least local privileges. Not what we want. However, if this is turned off, it can allow the creation of SUID bit files, allowing a remote user root access to the connected system.

/home		*(rw,no_root_squash)

   NFS Access ->

        Gain Low Privilege Shell ->

            Upload Bash Executable to the NFS share ->

                Set SUID Permissions Through NFS Due To Misconfigured Root Squash ->

                    Login through SSH ->

                        Execute SUID Bit Bash Executable ->

                            ROOT ACCESS

