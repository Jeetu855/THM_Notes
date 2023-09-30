
#### Telnet(Teletype Network) 

The Telnet protocol is an application layer protocol used to connect to a virtual terminal of another computer. Using Telnet, a user can log into another computer and access its terminal (console) to run programs, start batch processes, and perform system administration tasks remotely.

Telnet protocol is relatively simple. When a user connects, they will be asked for a username and password. Upon correct authentication, the user will access the remote system’s terminal. Unfortunately, all this communication between the Telnet client and the ***Telnet server is not encrypted, making it an easy target for attackers.***

A Telnet server uses the Telnet protocol to listen for incoming connections on port 23
`telnetd`, a Telnet server.

When a user tries to connect to a telnet server

1. First, he is asked to provide his login name (username). 
2. Then, he is asked for the password, `D2xc9CgD`. The password is not shown on the screen; 
3. Once the system checks his login credentials, he is greeted with a welcome message.
4. And the remote server grants him a command prompt, eg `frank@bento:~$`. The `$` indicates that this is not a root terminal.

***Although Telnet gave us access to the remote system’s terminal in no time, it is not a reliable protocol for remote administration as all the data are sent in cleartext***

Telnet is no longer considered a secure option, especially that anyone capturing your network traffic will be able to discover your usernames and passwords, which would grant them access to the remote system. The secure alternative is SSH


---

#### HTTP(Hyper Text Transfer Protocol)

Hypertext Transfer Protocol (HTTP) is the protocol used to transfer web pages. Your web browser connects to the webserver and uses HTTP to request HTML pages and images among other files and submit forms and upload various files. Anytime you browse the World Wide Web (WWW), you are certainly using the HTTP protocol.

![[http.png]]


HTTP sends and receives data as cleartext (not encrypted); therefore, you can use a simple tool, such as Telnet (or Netcat), to communicate with a web server and act as a “web browser”. The key difference is that you need to input the HTTP-related commands instead of the web browser doing that for you.

Using Telnet as a web browser to request HTML from a server

1. First, we connect to port 80 using `telnet 10.10.117.32 80`.
2. Next, we need to type `GET /index.html HTTP/1.1` to retrieve the page `index.html` or `GET / HTTP/1.1` to retrieve the default page.
3. Finally, you need to provide some value for the host like `host: telnet` and hit the Enter/Return key twice.

![[telnetcmd.png]]

Three popular choices for HTTP servers are:

  - Apache
  - Internet Information Services (IIS)
  - nginx


Apache and Nginx are free and open-source software. However, IIS is closed source software and requires paying for a license.


There are many web browsers available

- Chrome by Google
- Edge by Microsoft
- Firefox by Mozilla
- Safari by Apple.


---

#### FTP(File Transfer Protocol)

File Transfer Protocol (FTP) was developed to make the transfer of files between different computers with different systems efficient.

FTP also sends and receives data as cleartext; therefore, we can use Telnet (or Netcat) to communicate with an FTP server and act as an FTP client

1. We connected to an FTP server using a Telnet client. Since FTP servers listen on port 21 by default, we had to specify to our Telnet client to attempt connection to port 21 instead of the default Telnet port.
2. We needed to provide the username with the command `USER frank`.
3. Then, we provided the password with the command `PASS D2xc9CgD`.
4. If we supply correct username and password, we are logged in.

A command like `STAT` can provide some added information. The `SYST` command shows the System Type of the target (UNIX in this case). `PASV` switches the mode to passive. It is worth noting that there are two modes for FTP:

- ***Active: In the active mode, the data is sent over a separate channel originating from the FTP server’s port 20***.
- ***Passive: In the passive mode, the data is sent over a separate channel originating from an FTP client’s port above port number 1023.***

**The command `TYPE A` switches the file transfer mode to ASCII, while `TYPE I` switches the file transfer mode to binary. However, we cannot transfer a file using a simple client such as Telnet because FTP creates a separate connection for file transfer.***

![[ftp.png]]


***Considering the sophistication of the data transfer over FTP, let’s use an actual FTP client to download a text file. We only needed a small number of commands to retrieve the file. After logging in successfully, we get the FTP prompt, `ftp>`, to execute various FTP commands. We used `ls` to list the files and learn the file name; then, we switched to `ascii` since it is a text file (not binary). Finally, `get FILENAME` made the client and server establish another channel for file transfer.***

FTP servers and FTP clients use the FTP protocol. There are various FTP server software that you can select from if you want to host your FTP file server. Examples of FTP server software include:

-  vsftpd
-  ProFTPD
- uFTP

For FTP clients, in addition to the console FTP client commonly found on Linux systems, you can use an FTP client with GUI such as FileZilla. Some web browsers also support FTP protocol.

Because FTP sends the login credentials along with the commands and files in cleartext, FTP traffic can be an easy target for attackers.

---

#### SMTP(Simple Mail Transfer Protocol)

Email is one of the most used services on the Internet. There are various configurations for email servers; for instance, you may set up an email system to allow local users to exchange emails with each other with no access to the Internet. However, we will consider the more general setup where different email servers connect over the Internet.

Email delivery over the Internet requires the following components:

1. ***Mail Submission Agent (MSA)***
2. ***Mail Transfer Agent (MTA)***
3. ***Mail Delivery Agent (MDA)***
4. ***Mail User Agent (MUA)***


![[mail.png]]


The figure shows the following five steps that an email needs to go through to reach the recipient’s inbox:

1. ***A Mail User Agent (MUA), or simply an email client, has an email message to be sent. The MUA connects to a Mail Submission Agent (MSA) to send its message.***
2. ***The MSA receives the message, checks for any errors before transferring it to the Mail Transfer Agent (MTA) server, commonly hosted on the same server***.
3. ***The MTA will send the email message to the MTA of the recipient. The MTA can also function as a Mail Submission Agent (MSA).***
4. ***A typical setup would have the MTA server also functioning as a Mail Delivery Agent (MDA).***
5. ***The recipient will collect its email from the MDA using their email client.***


1. You (MUA) want to send postal mail.
2. The post office employee (MSA) checks the postal mail for any issues before your local post office (MTA) accepts it.
3. The local post office checks the mail destination and sends it to the post office (MTA) in the correct country.
4. The post office (MTA) delivers the mail to the recipient mailbox (MDA).
5. The recipient (MUA) regularly checks the mailbox for new mail. They notice the new mail, and they take it.



In the same way, we need to follow a protocol to communicate with an HTTP server, and ***we need to rely on email protocols to talk with an MTA and an MDA. The protocols are:***
1. ***Simple Mail Transfer Protocol (SMTP)***
2. ***Post Office Protocol version 3 (POP3) or Internet Message Access Protocol (IMAP)***


Simple Mail Transfer Protocol (SMTP) is used to communicate with an MTA server. ***Because SMTP uses cleartext, where all commands are sent without encryption, we can use a basic Telnet client to connect to an SMTP server and act as an email client (MUA) sending a message.***

SMTP server listens on port 25 by default.

After `helo`, we issue `mail from:`, `rcpt to:` to indicate the sender and the recipient. When we send our email message, we issue the command `data` and type our message. We issue `<CR><LF>.<CR><LF>` (or `Enter . Enter` to put it in simpler terms). The SMTP server now queues the message.


---

### POP3(Post Office Protocol 3)

Post Office Protocol version 3 (POP3) is a protocol used to download the email messages from a Mail Delivery Agent (MDA) server,
The mail client connects to the POP3 server, authenticates, downloads the new email messages before (optionally) deleting them.

![[smtppop3imap.png]]


***POP3 default port 110***

Authentication is required to access the email messages; the user authenticates by providing his username USER frank and password PASS D2xc9CgD. Using the command STAT, we get the reply +OK 1 179; based on RFC 1939, a positive response to STAT has the format +OK nn mm, where nn is the number of email messages in the inbox, and mm is the size of the inbox in octets (byte). The command LIST provided a list of new messages on the server, and RETR 1 retrieved the first message in the list. 

***In general, your mail client (MUA) will connect to the POP3 server (MDA), authenticate, and download the messages.***

Based on the default settings, the mail client deletes the mail message after it downloads it. The default behaviour can be changed from the mail client settings if you wish to download the emails again from another mail client. Accessing the same mail account via multiple clients using POP3 is usually not very convenient as one would lose track of read and unread messages. To keep all mailboxes synchronized, we need to consider other protocols, such as IMAP.

---

#### IMAP(Internet Message Access Protocol)

Internet Message Access Protocol (IMAP) is more sophisticated than POP3. IMAP makes it possible to keep your email synchronized across multiple devices (and mail clients). In other words, if you mark an email message as read when checking your email on your smartphone, ***the change will be saved on the IMAP server (MDA) and replicated on your laptop when you synchronize your inbox***

***IMAP server’s default port 143***

`LOGIN username password`

IMAP requires each command to be preceded by a random string to be able to track the reply. So we added `c1`, then `c2`, and so on. Then we listed our mail folders using `LIST "" "*"`, before checking if we have any new messages in the inbox using `EXAMINE INBOX`

***It is clear that IMAP sends the login credentials in cleartext, as we can see in the command `LOGIN frank D2xc9CgD`. Anyone watching the network traffic would be able to know Frank’s username and password.***



---

|Protocol|TCP Port|Application(s)|Data Security|
|---|---|---|---|
|FTP|21|File Transfer|Cleartext|
|HTTP|80|Worldwide Web|Cleartext|
|IMAP|143|Email (MDA)|Cleartext|
|POP3|110|Email (MDA)|Cleartext|
|SMTP|25|Email (MTA)|Cleartext|
|Telnet|23|Remote Access|Cleartext|


Servers implementing these protocols are subject to different kinds of attacks. To name a few, consider:

1. Sniffing Attack (Network Packet Capture)
2. Man-in-the-Middle (MITM) Attack
3. Password Attack (Authentication Attack)

