
File Transfer Protocol (FTP) is, as the name suggests , a protocol used to allow remote transfer of files over a network. It uses a client-server model to do this, and- as we'll come on to later- relays commands and data in a very efficient way.

A typical FTP session operates using two channels:

- a command (sometimes called the control) channel
- a data channel.

The command channel is used for transmitting commands as well as replies to those commands, while the data channel is used for transferring data.

FTP operates using a client-server protocol. The client initiates a connection with the server, the server validates whatever login credentials are provided and then opens the session.

While the session is open, the client may execute FTP commands on the server.


**Active vs Passive**

The FTP server may support either Active or Passive connections, or both. 

- In an Active FTP connection, the client opens a port and listens. The server is required to actively connect to it.  _**In the active mode, the data is sent over a separate channel originating from the FTP server’s port 20**_
- In a Passive FTP connection, the server opens a port and listens (passively) and the client connects to it. _**In the passive mode, the data is sent over a separate channel originating from an FTP client’s port above port number 1023.**_

This separation of command information and data into separate channels is a way of being able to send commands to the server without having to wait for the current data transfer to finish. If both channels were interlinked, you could only enter commands in between data transfers, which wouldn't be efficient for either large file transfers, or slow internet connections.


#### Enumerating FTP

We're can exploit an anonymous FTP login, to see what files we can access- and if they contain any information that might allow us to pop a shell on the system

```sh
ftp serverIP
```

#### Exploiting FTP

When using FTP both the command and data channels are unencrypted. Any data sent over these channels can be intercepted and read.

With data from FTP being sent in plaintext, if a man-in-the-middle attack took place an attacker could reveal anything sent through this protocol (such as passwords)

Using ARP-Poisoning to trick a victim into sending sensitive information to an attacker, rather than a legitimate source.

