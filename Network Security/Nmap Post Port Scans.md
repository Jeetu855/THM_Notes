
#### Service Detection

Once Nmap discovers open ports, you can probe the available port to detect the running service. Further investigation of open ports is an essential piece of information as the pentester can use it to learn if there are any known vulnerabilities of the service

Adding `-sV` to your Nmap command will collect and determine service and version information for the open ports. You can control the intensity with `--version-intensity LEVEL` where the level ranges between 0, the lightest, and 9, the most complete. `-sV --version-light` has an intensity of 2, while `-sV --version-all` has an intensity of 9.

-sV : Attempts to determine the version of the service running on port

```sh
sudo nmap -sV --version-light TargetIP
```

```sh
sudo nmap -sV --version-all TargetIP
```

***It is important to note that using `-sV` will force Nmap to proceed with the TCP 3-way handshake and establish the connection. The connection establishment is necessary because Nmap cannot discover the version without establishing a connection fully and communicating with the listening service. In other words, stealth SYN scan `-sS` is not possible when `-sV` option is chosen***

---

#### OS Detection and Traceroute

Nmap can detect the Operating System (OS) based on its behaviour and any telltale signs in its responses. OS detection can be enabled using `-O`; this is an _uppercase O_ as in OS.

```sh
sudo nmap -sS -O 10.10.133.52
```

***The OS detection is very convenient, but many factors might affect its accuracy. First and foremost, Nmap needs to find at least one open and one closed port on the target to make a reliable guess. Furthermore, the guest OS fingerprints might get distorted due to the rising use of virtualization and similar technologies. Therefore, always take the OS version with a grain of salt.***

##### Traceroute

If you want Nmap to find the routers between you and the target, just add `--traceroute`. In the following example, Nmap appended a traceroute to its scan results. Note that Nmap’s traceroute works slightly different than the `traceroute` command found on Linux and macOS or `tracert` found on MS Windows. Standard `traceroute` starts with a packet of low TTL (Time to Live) and keeps increasing until it reaches the target. Nmap’s traceroute starts with a packet of high TTL and keeps decreasing it.

```sh
sudo nmap -sS --traceroute TargetIP
```

many routers are configured not to send ICMP Time-to-Live exceeded, which would prevent us from discovering their IP addresses

---

#### Nmap Scripting Engine(NSE)

***A script is a piece of code that does not need to be compiled. In other words, it remains in its original human-readable form and does not need to be converted to machine language.*** Many programs provide additional functionality via scripts; moreover, scripts make it possible to add custom functionality that did not exist via the built-in commands. Similarly, Nmap provides support for scripts using the Lua language. A part of Nmap, Nmap Scripting Engine (NSE) is a Lua interpreter that allows Nmap to execute Nmap scripts written in Lua language.
Nmap default installation can easily contain close to 600 scripts in the directory
`/usr/share/nmap/scripts`


You can choose to run the scripts in the default category using --script=default or simply adding -sC. In addition to default, categories include auth, broadcast, brute, default, discovery, dos, exploit, external, fuzzer, intrusive, malware, safe, version, and vuln

|Script Category|Description|
|---|---|
|`auth`|Authentication related scripts|
|`broadcast`|Discover hosts by sending broadcast messages|
|`brute`|Performs brute-force password auditing against logins|
|`default`|Default scripts, same as `-sC`|
|`discovery`|Retrieve accessible information, such as database tables and DNS names|
|`dos`|Detects servers vulnerable to Denial of Service (DoS)|
|`exploit`|Attempts to exploit various vulnerable services|
|`external`|Checks using a third-party service, such as Geoplugin and Virustotal|
|`fuzzer`|Launch fuzzing attacks|
|`intrusive`|Intrusive scripts such as brute-force attacks and exploitation|
|`malware`|Scans for backdoors|
|`safe`|Safe scripts that won’t crash the target|
|`version`|Retrieve service versions|
|`vuln`|Checks for vulnerabilities or exploit vulnerable services|

Some scripts belong to more than one category. Moreover, some scripts launch brute-force attacks against services, while others launch DoS attacks and exploit systems.


```sh
sudo nmap -sS -sC 10.10.237.209
```

-sC : Scan with default NSE scripts. Considered useful for discovery and safe

You can also specify the script by name using `--script "SCRIPT-NAME"` or a pattern such as `--script "ftp*"`, which would include `ftp-brute`. If you are unsure what a script does, you can open the script file with a text reader, such as `less`, or a text edito

```sh
sudo nmap -sS --script "ftp*" TargetIP
```

In the case of `ftp-brute`, it states: “Performs brute force password auditing against FTP servers.” You have to be careful as some scripts are pretty intrusive. Moreover, some scripts might be for a specific server and, if chosen at random, will waste your time with no benefit. As usual, make sure that you are authorized to launch such tests on the target server.

```sh
sudo nmap -sS -n --script "http-date" TargetIP
```

---

#### Saving The Output

Whenever you run a Nmap scan, it is only reasonable to save the results in a file. Selecting and adopting a good naming convention for your filenames is also crucial. The number of files can quickly grow and hinder your ability to find a previous scan result. The three main formats are:

1. Normal
2. Grepable (`grep`able)
3. XML

There is a fourth one that we cannot recommend:

- Script Kiddie


##### Normal

As the name implies, the normal format is similar to the output you get on the screen when scanning a target. You can save your scan in normal format by using `-oN FILENAME`; N stands for normal.

```sh
sudo nmap -sS targetIP -oN scan.initial
```

##### Grepable

The grepable format has its name from the command `grep`; grep stands for Global Regular Expression Printer. In simple terms, it makes filtering the scan output for specific keywords or terms efficient. You can save the scan result in grepable format using `-oG FILENAME`
The main reason is that Nmap wants to make each line meaningful and complete when the user applies `grep`. As a result, in grepable output, the lines are so long and are not convenient to read compared to normal output.

```sh
sudo nmap -sS targetIP -oG scan.initial
```


##### XML

The third format is XML. You can save the scan results in XML format using `-oX FILENAME`. The XML format would be most convenient to process the output in other programs. Conveniently enough, you can save the scan output in all three formats using `-oA FILENAME` to combine `-oN`, `-oG`, and `-oX` for normal, grepable, and XML.

```sh
sudo nmap -sS targetIP -oX scan.initial
```

***To save as all 3 formats***

```sh
sudo nmap -sS targetIP -oA scan.initial
```


##### Script Kiddie

A fourth format is script kiddie. You can see that this format is useless if you want to search the output for any interesting keywords or keep the results for future reference. However, you can use it to save the output of the scan `nmap -sS 127.0.0.1 -oS FILENAME`, display the output filename, and look 31337 in front of friends who are not tech-savvy.

```sh
sudo nmap -sS targetIP -oS scan.initial
```


---

|Option|Meaning|
|---|---|
|`-sV`|determine service/version info on open ports|
|`-sV --version-light`|try the most likely probes (2)|
|`-sV --version-all`|try all available probes (9)|
|`-O`|detect OS|
|`--traceroute`|run traceroute to target|
|`--script=SCRIPTS`|Nmap scripts to run|
|`-sC` or `--script=default`|run default scripts|
|`-A`|equivalent to `-sV -O -sC --traceroute`|
|`-oN`|save output in normal format|
|`-oG`|save output in grepable format|
|`-oX`|save output in XML format|
|`-oA`|save output in normal, XML and Grepable formats|


