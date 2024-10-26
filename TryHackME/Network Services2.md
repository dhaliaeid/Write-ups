# Network services 2
## Understanding NFS
#### What is NFS?
NFS stands for "Network File System" and allows a system to share directories and files with others over a network.
#### How does NFS work?
-1- The client will request to mount a directory from a remote host on a local directory.

-2- The mount service will then act to connect to the relevant mount daemon using RPC.

-3- The server checks if the user has permission to mount whatever directory has been requested.

-4- It will then return a file handle which uniquely identifies each file and directory that is on the server.
#### What runs NFS?
Using the NFS protocol, you can transfer files between computers running Windows and other non-Windows operating systems, such as Linux, MacOS or UNIX.

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

    


