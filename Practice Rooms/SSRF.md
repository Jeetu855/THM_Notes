
When developing networked software, it's common to make requests to external servers. Developers often use these requests to **fetch remote resources** like software updates or import data from other applications. While these requests are typically safe, **improper implementation can lead to a vulnerability known as SSRF**.

![[ssrf.png]]

An SSRF vulnerability can arise when user-provided data is used to construct a request, such as forming a URL. To execute an SSRF attack, **an attacker can manipulate a parameter value within the vulnerable software**, effectively creating or controlling requests from that software and directing them towards other servers or even the same server.

SSRF vulnerabilities can be found in various types of computer software across a wide range of programming languages and platforms as long as the software operates in a networked environment. While most SSRF vulnerabilities are commonly discovered in web applications and other networked software, they can also be present in server software.

##### Risk of SSRF  

**Data Exposure**

As explained earlier, cybercriminals can gain unauthorised access by tampering with requests on behalf of the vulnerable web application to gain access to sensitive data hosted in the internal network.

**Reconnaissance**

An attacker can carry out port scanning of internal networks by running malicious scripts on vulnerable servers or redirecting to scripts hosted on some external server.

  

**Denial of Service**

It is a common scenario that internal networks or servers do not expect many requests; therefore, they are configured to handle low bandwidth. Attackers can flood the servers with multiple illegitimate requests, causing them to remain unavailable to handle genuine requests.


---

### Types of SSRF - Basic

﻿Basic SSRF is a web attack technique where an attacker tricks a server into making requests on their behalf, often targeting internal systems or third-party services. By exploiting vulnerabilities in input validation, the attacker can gain unauthorised access to sensitive information or control over remote resources, posing a significant security risk to the targeted application and its underlying infrastructure.
A basic SSRF can be employed against a local or an internal server.

##### SSRF Against a Local Server  

In this attack, the attacker makes an unauthorised request to the server hosting the web application. The attacker typically supplies a loopback IP address or localhost to receive the response. The vulnerability arises due to how the application handles input from the query parameter or API calls.

##### Scenario - II: Accessing an Internal Server

In complex web applications, it is common for front-end web applications to interact with back-end internal servers. These servers are generally hosted on **non-routable IP addresses**, so an internet user cannot access them. In this scenario, an attacker exploits a vulnerable web application's input validation to trick the server into requesting **internal resources on the same network**. They could provide a malicious URL as input, making the server interact with the internal server on their behalf.

For instance, if an internal server provides database management or administrative controls, the attacker could craft a URL that initiates an unintended action on these internal systems when processed by the vulnerable web application. Technically, this is achieved through manipulated input, such as special IP addresses (like `192.168.x.x` or `10.x.x.x` for IPv4) or domain names (e.g., `internal-database.hrms.thm`). When not properly sanitised, these inputs can be used in functions like HTTP requests or file inclusions within the web application. The server, interpreting these requests as legitimate, then inadvertently performs actions or retrieves data from other internal services.

Moreover, since these internal servers may lack the same level of security monitoring as external-facing servers, such exploitation can often go unnoticed. The attacker might also use this method to perform reconnaissance within the internal network, identifying other vulnerable systems or services to target.

---

### Types of SSRF - Blind

Blind SSRF refers to a scenario where the attacker can send requests to a target server, but they do not receive direct responses or feedback about the outcome of their requests. In other words, the attacker is blind to the server's responses. This type of SSRF can be more challenging to exploit because the attacker cannot directly see the results of their actions. We will discuss its various examples.

##### Blind SSRF With Out-Of-Band

Out-of-band SSRF is a technique where the attacker leverages a separate, out-of-band communication channel instead of directly receiving responses from the target server to receive information or control the exploited server. This approach is practical when the server's responses are not directly accessible to the attacker.

For instance, the attacker might manipulate the vulnerable server to make a DNS request to a domain he owns or to initiate a connection to an external server with specific data. This external interaction provides the attacker with evidence that the SSRF vulnerability exists and potentially allows him to gather additional information, such as internal IP addresses or the internal network's structure.

```py
           
from http.server import SimpleHTTPRequestHandler, HTTPServer
from urllib.parse import unquote
class CustomRequestHandler(SimpleHTTPRequestHandler):

    def end_headers(self):
        self.send_header('Access-Control-Allow-Origin', '*')  # Allow requests from any origin
        self.send_header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
        self.send_header('Access-Control-Allow-Headers', 'Content-Type')
        super().end_headers()

    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'Hello, GET request!')

    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length).decode('utf-8')

        self.send_response(200)
        self.end_headers()

        # Log the POST data to data.html
        with open('data.html', 'a') as file:
            file.write(post_data + '\n')
        response = f'THM, POST request! Received data: {post_data}'
        self.wfile.write(response.encode('utf-8'))

if __name__ == '__main__':
    server_address = ('', 8080)
    httpd = HTTPServer(server_address, CustomRequestHandler)
    print('Server running on http://localhost:8080/')
    httpd.serve_forever()

        
```

**The above code will receive all the content and save it to a file data.html on the server. Also sets up the server on port 8080**

##### Semi-Blind SSRF (Time-based)  

Time-based SSRF is a variation of SSRF where the attacker leverages timing-related clues or delays to infer the success or failure of their malicious requests. By **observing how long it takes for the application to respond**, the attacker can make educated guesses about whether their SSRF attack was successful. 

The attacker sends a series of requests, each targeting a different resource or URL. The attacker measures the response times for each request. If a response takes significantly longer, it may indicate that the server successfully accessed the targeted resource, implying a successful SSRF attack.

##### Crashing the Server

An attacker could abuse SSRF by crashing the server or creating a denial of service for other hosts. There are multiple instances (WordPress, CairoSVG) where attackers try to disrupt the availability of a system by launching SSRF attacks. 

---

### Remedial Measures

Mitigation measures for SSRF are essential for preserving the security and integrity of web applications. Implementing robust SSRF mitigation measures helps protect against these risks by **fortifying the application's defences**, **preventing malicious requests**, and **bolstering the overall security posture**. As a critical element of web application security, SSRF mitigation measures are instrumental in preserving user data, safeguarding against data breaches, and maintaining trust in the digital ecosystem. A few of the important policies are mentioned below:

![Various techniques for mitigation](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/24147651267eedd828e4c103b84795d9.svg)

- **Implement strict input validation** and sanitise all user-provided input, especially any URLs or input parameters the application uses to make external requests.
- Instead of trying to blocklist or filter out disallowed URLs, **maintain allowlists of trusted URLs or domains**. Only allow requests to these trusted sources.
- **Implement network segmentation** to isolate sensitive internal resources from external access.
- **Implement security headers**, such as Content-Security-Policy, that restricts the application's load of external resources.  
    
- **Implement strong access controls** for internal resources, so even if an attacker succeeds in making a request, they can't access sensitive data without proper authorisation.
- **Implement comprehensive logging and monitoring** to track and analyse incoming requests. Look for unusual or unauthorised requests and set up alerts for suspicious activity.

