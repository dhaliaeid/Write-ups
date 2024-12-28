# Network services 2 (NFS, SMTP, and MySQL)

## Overview

This write-up focuses on exploring and exploiting various network services, with a primary emphasis on NFS (Network File System). Additionally, we delve into the enumeration and exploitation of SMTP and MySQL services. Each section is interconnected, showing how vulnerabilities in these services can provide a pathway for attackers to infiltrate systems. By following this structured guide, you’ll gain an understanding of the practical applications of enumeration techniques, misconfiguration exploitation, and lateral movement strategies commonly used in penetration testing.

We’ll cover the following topics:


![image](https://github.com/user-attachments/assets/c3734f5c-dbf5-43a6-ba17-acb0f3112366)


Each section will explain how these services interact and demonstrate how a small piece of information obtained from one service can lead to full system compromise when chained together. We’ll use tools, commands, and techniques relevant to real-world penetration testing scenarios.



## 1.1) Understanding NFS

#### What is NFS?
Network File System (NFS) allows systems to share directories and files over a network, enabling seamless access across devices. It is widely used in environments requiring shared storage.

NFS facilitates shared access to files and directories across platforms like Windows, Linux, and macOS. However, its misconfigurations, such as improper permissions or disabled root squashing, often create security vulnerabilities. 

#### How NFS Works

![image](https://github.com/user-attachments/assets/9773bad1-d8c6-4a92-98e6-7db4ea1a0986)

   ###### _figure 1_

1. **Mounting a Directory**: A client requests to mount a directory from the NFS server.

2. **Remote Procedure Call (RPC)**: The mount service communicates with the NFS daemon using RPC.

3. **Permission Check**: The server validates access permissions for the user.

4. **File Handle Assignment**: The server provides a unique file handle to access files and directories.



**_THM Section Answers_**

   
![image](https://github.com/user-attachments/assets/9ca8078b-d3c0-4917-9495-7498e6006e9e)

______
## 1.2) Enumerating NFS

#### What is Enumeration?

Enumeration involves establishing an active connection to a target host to extract details such as usernames, services, network shares, and configurations. For NFS, tools like `nfs-common` and `showmount` are particularly useful.

#### Steps for NFS Enumeration

1. **Port Scanning** : Use `nmap` to identify open ports and services.
  > nmap -A -p- <target-IP>
2. **Inspect NFS Shares**:list all available NFS shares using showmount:
  > showmount -e <target-IP>
3. **Mounting the Share**:Mount the NFS share locally to access its contents.
  > sudo mount -t nfs <IP>:<share> /tmp/mount/ -nolock


    
**_THM Section Answers_**

1. Conduct a thorough port scan of your choosing, how many ports are open?

![image](https://github.com/user-attachments/assets/d460f3f2-c5ef-46ad-8500-f16f63eb7488)

2. Which port contains the service we're looking to enumerate?

![image](https://github.com/user-attachments/assets/49d7e34a-cbc9-4390-8ccf-a555e5b66b74)

3. what is the name of the visible share?

![image](https://github.com/user-attachments/assets/ea79e1f9-6618-4d78-a911-5159eac48e7b)

4. what is the name of the folder inside?

![image](https://github.com/user-attachments/assets/5ffab10d-6cb2-4414-8276-441c115fbdbc)

5. Which of these folders could contain keys that would give us remote access to the server?

 ![image](https://github.com/user-attachments/assets/034eccfe-1147-453f-9cc5-5b1b267c4ff9)

6. Which of these keys is most useful to us?

 ![image](https://github.com/user-attachments/assets/8228e6ca-2e06-40fa-8a76-3c5972f56ec4)

7. Copy this file to a different location your local machine, and change the permissions to "600" using "chmod 600 [file]". Assuming we were right about what type of directory this is, we can pretty easily work out the name of the user this key corresponds to.Can we log into the machine using ssh -i <key-file> <username>@<ip> ? (Y/N)

![image](https://github.com/user-attachments/assets/c909cad3-1f57-4c30-b3a5-14ece34c4cde)

___
## 1.3) Exploiting NFS

#### Root Squashing and Its Risks
NFS often enables root squashing by default, which maps root user requests to a non-privileged user. If disabled, attackers can create files with the SUID bit (means The files can be run with the permissions of the file(s) owner/group.), granting root access.

#### Exploitation Steps
1.  **Copy a Malicious Binary**: Copy a bash binary to the NFS share and set its owner to root.
  > sudo cp /bin/bash /tmp/mount/
  > sudo chown root:root /tmp/mount/bash
2. **Set the SUID Bit**: Add SUID permission to the binary.
  > sudo chmod +s /tmp/mount/bash
3. **Execute as Root**: Run the binary with elevated privileges.
  > ./bash -p



**_THM Section Answers_**

1. First, change directory to the mount point on your machine, where the NFS share should still be mounted, and then into the user's home directory.(password:password)
     
![image](https://github.com/user-attachments/assets/6149195f-946d-4797-b0e2-fc8d6b7530ac)

2.  Download the bash executable to your Downloads directory. Then use "cp ~/Downloads/bash ." to copy the bash executable to the NFS share. The copied bash shell must be owned by a root user, you can set this using "sudo chown root bash"
   
 first: copy the  .bash from  its directory to the nfs share 

 ![image](https://github.com/user-attachments/assets/65039afd-37f6-4bb5-b3c4-bf0447447982)

second: change the file owner by a root user

![image](https://github.com/user-attachments/assets/59a39c19-2545-45f1-af53-e8dd512f789f)

3. Now, we're going to add the SUID bit permission to the bash executable we just copied to the share using "sudo chmod +[permission] bash". What letter do we use to set the SUID bit set using chmod?
   
 ![image](https://github.com/user-attachments/assets/6a879b90-ce73-485b-88af-526ce71d2505)

5. Let's do a sanity check, let's check the permissions of the "bash" executable using "ls -la bash". What does the permission set look like? Make sure that it ends with -sr-x.
 - -rwxr-xr-x
6.  Now, SSH into the machine as the user. List the directory to make sure the bash executable is there. Now, the moment of truth. Lets run it with "./bash -p". The -p persists the permissions, so that it can run as root with SUID- as otherwise bash will sometimes drop the permissions. Great! If all's gone well you should have a shell as root! What's the root flag?
- THM{nfs_got_pwned}

![image](https://github.com/user-attachments/assets/e638c2f3-5f1f-44c8-a08a-eb9caa6f7ce4)
 
![image](https://github.com/user-attachments/assets/18ea8c1e-720a-4fff-85ed-695677d6696a)

![image](https://github.com/user-attachments/assets/f3ccd626-31ec-4b11-9e93-ce1caf9ff177)

7. What option do we need to set to the wordlist's path?

  to install seclist with kali : 

![image](https://github.com/user-attachments/assets/6782606c-653a-48ec-bd52-eaa93ce3287e)

![image](https://github.com/user-attachments/assets/34784548-c9ce-47f5-b0ca-1bded73a9a18)

8. what is the other essential paramater we need to set?

![image](https://github.com/user-attachments/assets/7e70a7be-9406-4d06-97d3-05bafaa108a7)

9. what username is returned?

![image](https://github.com/user-attachments/assets/b76645a5-80d2-4a6b-b154-8fb5649f271b)


___
## 2.1) Understanding STMP
SMTP (Simple Mail Transfer Protocol) is a standard protocol used for sending emails. It works alongside POP (Post Office Protocol) or IMAP (Internet Message Access Protocol) to handle email communication, where SMTP manages outgoing mail while POP/IMAP handles incoming mail.

#### Key Functions of an SMTP Server:
**Authentication**: Verifies the sender's identity.
**Mail Delivery**: Sends outgoing mail to the recipient's server.
**Error Handling**: Returns undelivered mail to the sender.

SMTP configuration is often required when setting up email clients (e.g., Thunderbird) for outgoing emails.
SMTP server software is available for windows(Native server options) and linux (Multiple open-source SMTP variants).

#### How SMTP Works
SMTP operates like a digital postal service, delivering emails through a structured process:

![image](https://github.com/user-attachments/assets/8069edc0-4d63-4f53-86f0-f46bf8ab097d)


![image](https://github.com/user-attachments/assets/73d09d95-6918-4eb1-b3b0-68fb854c9412)
   ###### _figure2_
1. **Connection**: Your email client connects to the SMTP server (e.g., smtp.google.com) via port 25, initiating an SMTP handshake.
2. **Email Submission**: The client sends the sender’s address, recipient’s address, email body, and attachments to the SMTP server.
3. **Domain Check**: The SMTP server verifies whether the sender’s and recipient’s domains match.
4. **Relaying**: The server connects to the recipient’s SMTP server to transfer the email. If unavailable, the message enters an SMTP queue.
5. **Verification**: The recipient’s SMTP server confirms the domain and username.
6.** Delivery**: The email is passed to the recipient’s POP/IMAP server, where it appears in the inbox.

**_THM Section Answers_**


![image](https://github.com/user-attachments/assets/f1d679d1-9a91-4d6e-bc23-4451a5941de0)
______
## 2.2) Enumerating STMP

SMTP (Simple Mail Transfer Protocol) is not only essential for email communication but can also be leveraged for reconnaissance during a penetration test or attack. By enumerating server details and users, an attacker can gather crucial information, such as vulnerable mail server versions and valid email addresses, to further their exploit efforts.
1. **Server Fingerprinting with Metasploit**
Use the smtp_version module in Metasploit to scan IP ranges and identify the versions of mail servers. This helps in targeting poorly configured or vulnerable servers.
2. **User Enumeration via SMTP**
SMTP also provides two internal commands, VRFY and EXPN, that can be exploited to enumerate valid email users and their aliases:
**VRFY**: Confirms whether an email address is valid on the server.
**EXPN**: Reveals mailing lists and the actual addresses of aliases.
Using these commands, an attacker can gather a list of valid users or identify valuable targets. While these commands can be tested manually via Telnet, Metasploit’s smtp_enum module automates this process, making it easier and faster to enumerate users.

___
## 2.3) Exploiting STMP

#### SMTP Overview

SMTP (Simple Mail Transfer Protocol) is primarily used for email transmission. Its misconfigurations can leak information about users and servers.
#### Hydra
`Hydra`, is a tool for customizable, adaptive password attacks against services like SSH. Hydra primarily uses dictionary attacks, and wordlists can be found in `/usr/share/wordlists` including `rockyou.txt` Additional wordlists can be sourced from `SecLists` for broader use.

##### Steps for Exploitation
1. **Enumerate Users**: Use tools like Hydra to brute-force user credentials.
>> hydra -t 16 -l <username> -P <wordlist> <IP> smtp
2. **Brute-Force SSH**: Use retrieved credentials to brute-force SSH access.
>> hydra -t 16 -l <username> -P <wordlist> <IP> ssh

**_THM Section Answers_**

1. What is the password of the user we found during our enumeration stage?

![image](https://github.com/user-attachments/assets/86beb2fe-3b4a-47a3-8ecc-f909a1befd47)

![image](https://github.com/user-attachments/assets/3e378ef0-5f92-4d39-8fb8-9489365e6b20)

2. Great! Now, let's SSH into the server as the user, what is contents of smtp.txt

![image](https://github.com/user-attachments/assets/eef21be2-a0e5-4c62-9b77-d61d1a5b0ecd)

____
## 3.1) Understanding MySQL

MySQL is a popular open-source Relational Database Management System (RDBMS) that utilizes SQL (Structured Query Language) to manage and manipulate structured data. It organizes data into tables that can be related to each other through keys. It’s widely used in various applications, from web development to enterprise systems, due to its speed, reliability, and ease of use.

**Key Concepts**:
**Database**: A structured collection of persistent data.
**RDBMS**: Software that manages databases by using a relational model, where data is stored in tables.
**SQL**: The language used to interact with the database for querying, updating, and managing the data.

MySQL runs on various platforms (e.g., Linux, Windows) and is widely used as a backend for websites, often as part of the LAMP stack (Linux, Apache, MySQL, PHP).

##### How MySQL Works
MySQL follows a client-server architecture where the client sends requests to the MySQL server, which processes those requests and returns the required data.

![image](https://github.com/user-attachments/assets/adab0bdc-f49c-46c3-81d6-72e498d608dc)

**Steps in MySQL Operations:**
1. **Database Creation**: The server creates a database with defined relationships between tables.
2. **Client Requests**: Clients use SQL commands to interact with the database, such as querying for data or modifying tables.
3. **Server Response**: The server processes the request and responds with the requested data or status.

**_THM Section Answers_**

![image](https://github.com/user-attachments/assets/da728627-22d6-4d25-af8c-c9e58b1a9026)


___
## 3.2) Enumerating MySQL

#### When to Target MySQL

MySQL is typically not the primary focus during initial enumeration of a server. However, once credentials are obtained from other services, MySQL can be a valuable target. Brute-forcing MySQL passwords is an option, but it's rarely required in CTFs or structured penetration testing exercises.
**Common Scenarios**
- **Credential Discovery**: Credentials (e.g., username and password) are uncovered from other sources such as configuration files, subdomain enumeration, or web application vulnerabilities.
- **Testing Access**: After attempting to use the credentials on other services (e.g., SSH) without success, they are tested on MySQL.
- **Gaining Access**: Successful login to MySQL opens possibilities to extract sensitive information or escalate privileges.

**Example Scenario**: Suppose the credentials root were discovered through web server enumeration. After failing to log in via SSH, you try these credentials on the MySQL service.

**Requirements for MySQL Enumeration**
efore interacting with MySQL, ensure the MySQL client is installed on your system. You can install it with the following command:
>> sudo apt install default-mysql-client

This tool enables you to connect to the MySQL database server and execute SQL commands for enumeration and exploitation.

**Steps of Enumeration**
1. **Identify Running Port**: Use nmap to locate the MySQL service.
>>  nmap -p 3306 <target-IP>
      
**_THM Section Answers_**

1. As always, let's start out with a port scan, so we know what port the service we're trying to attack is running on. What port is MySQL using?

![image](https://github.com/user-attachments/assets/c81dcbc1-f535-4e17-9928-42ea11d9beac)

![image](https://github.com/user-attachments/assets/7bb59a5f-8cec-4650-aa7b-ec3167cadc24)

4. We're going to be using the "mysql_sql" module. Search for, select and list the options it needs. What three options do we need to set? (in descending order).
- PASSWORD/RHOSTS/USERNAME

![image](https://github.com/user-attachments/assets/51d9d7ec-37ef-4d5e-b412-083318c2b19a)
5. Run the exploit. By default it will test with the "select version()" command, what result does this give you?

![image](https://github.com/user-attachments/assets/efc5a666-bb94-4f94-b0fc-c71205d4fc4b)

6. Great! We know that our exploit is landing as planned. Let's try to gain some more ambitious information. Change the "sql" option to "show databases". how many databases are returned?

![image](https://github.com/user-attachments/assets/a94f2a8f-15e6-481c-b1c4-4d56e279ec46)


___
## Exploiting MySQL

#### Current Knowledge Recap
Before further exploiting the MySQL database, here’s what we know:
1. MySQL server credentials.
2. MySQL version.
3. Number of databases and their names.

#### Key Terminology
**Schema**: In MySQL, schema is interchangeable with database. For example, `CREATE SCHEMA` can be used in place of `CREATE DATABASE`. Note that in some database systems, like Oracle, schema only refers to part of a database (tables and objects owned by a user).
**Hashes**: A cryptographic process that turns variable-length input into a fixed-length output. In MySQL, hashes are used for password storage (protecting against plaintext storage) and data indexing, enabling efficient searching and access.

**Steps for Exploitation**
1. **Login to MySQL**: Use credentials obtained from enumeration.
 >> mysql -u root -p -h <target-IP>
2. **Dump Schemas and Hashes**:Extract database schemas and password hashes for further analysis.

Hash Cracking
Use John the Ripper to crack MySQL password hashes:
>> john hash.txt

**_THM Section Answers_**

1. First, let's search for and select the "mysql_schemadump" module. What's the module's full name?

![image](https://github.com/user-attachments/assets/52ed08ee-11a5-4887-b703-bb7f0ea59cbe)

2. Great! Now, you've done this a few times by now so I'll let you take it from here. Set the relevant options, run the exploit. What's the name of the last table that gets dumped?

![image](https://github.com/user-attachments/assets/0ed2e4e9-b522-47f1-966c-39a92c8ea953)

![image](https://github.com/user-attachments/assets/16151266-3802-458f-a168-6402c93e8f3f)

3. Awesome, you have now dumped the tables, and column names of the whole database. But we can do one better... search for and select the "mysql_hashdump" module. What's the module's full name?

![image](https://github.com/user-attachments/assets/fe80823a-f0bd-4769-853b-95023d3677de)

4. Again, I'll let you take it from here. Set the relevant options, run the exploit. What non-default user stands out to you?

![image](https://github.com/user-attachments/assets/bd5dcee5-fa09-4709-8def-198b1f4c7077)

5. Another user! And we have their password hash. This could be very interesting. Copy the hash string in full, like: bob:*HASH to a text file on your local machine called "hash.txt". What is the user/hash combination string?

![image](https://github.com/user-attachments/assets/5d9638ee-8a6c-49ab-bbd4-5546197146eb)

6. Now, we need to crack the password! Let's try John the Ripper against it using: "john hash.txt" what is the password of the user we found?

![image](https://github.com/user-attachments/assets/8ce3a778-ab8e-44e6-bfe7-7e55d1abfec1)

7. Awesome. Password reuse is not only extremely dangerous, but extremely common. What are the chances that this user has reused their password for a different service? What's the contents of MySQL.txt?

![image](https://github.com/user-attachments/assets/036c8bfa-7cf0-40c8-8767-030bc21f9e6d)




## References 
**figures**
1.https://www.lysator.liu.se/~mats-g/pubs/Lic/node27.html
2. 






        

    



  




    


