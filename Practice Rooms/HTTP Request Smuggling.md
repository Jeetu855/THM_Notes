
HTTP Request Smuggling is a vulnerability that arises when there are mismatches in different web infrastructure components. This includes proxies, load balancers, and servers that interpret the boundaries of HTTP requests. For example, consider a train station where tickets are checked at multiple points before boarding. If each checkpoint has different criteria for a valid ticket, a traveller could exploit these inconsistencies to board a train without a valid ticket. Similarly, in web requests, this vulnerability mainly involves the Content-Length and Transfer-Encoding headers, which indicate the end of a request body. When these headers are manipulated or interpreted inconsistently across components, it may result in one request being mixed with another.

![[HTTPRS1.png]]

***Request splitting or HTTP desync attacks are possible because of the nature of keep-alive connections and HTTP pipelining, which allow multiple requests to be sent over the same TCP connection. Without these mechanisms, request smuggling wouldn't be feasible.*** **When calculating the sizes for Content-Length (CL) and Transfer-Encoding (TE), it's crucial to consider the presence of carriage return `\r` and newline `\n` characters. These characters are not only part of the HTTP protocol's formatting but also impact the calculation of content sizes.**

While testing for request smuggling vulnerabilities, it's important to note that some tools might automatically "fix" the Content-Length header by default. This means if you're using such tools to run payloads, your Content-Length values might get overwritten, potentially changing the test results.

Note that testing for request smuggling can potentially break a website in many ways (cache poisoning, other user requests may start failing, or even the back-end pipeline might get fully desynced), so extreme care should be taken when testing this on a production website.

#### Importance of Understanding HTTP Request Smuggling

1. Smuggled requests might evade security mechanisms like Web Application Firewalls. This potentially leads to unauthorized access or data leaks.
2. Attackers can poison web caches by smuggling malicious content, causing users to see incorrect or harmful data.
3. Smuggled requests can be chained to exploit other vulnerabilities in the system, amplifying the potential damage.
4. Due to the intricate nature of this vulnerability, it can often go undetected, making it crucial for security professionals to understand and mitigate it

---

### Modern Infrastructure

##### Components of Modern Web Applications

Modern web applications are no longer straightforward, monolithic structures. They are composed of different components that work with each other. Below are some of the components that a modern web application usually consists of:

1. **Front-end server**: This is usually the reverse proxy or load balancer that forwards the requests to the back-end.
2. **Back-end server**: This server-side component processes user requests, interacts with databases, and serves data to the front-end. It's often developed using languages like PHP, Python, and Javascript and frameworks like Laravel, Django, or Node.js.
3. **Databases**: Persistent storage systems where application data is stored. Examples of this are databases like MySQL, PostgreSQL, and NoSQL.
4. **APIs (Application Programming Interfaces)**: Interfaces allow the front and back-end to communicate and integrate with other services.
5. **Microservices**: Instead of a single monolithic back-end, many modern applications use microservices, which are small, independent services that communicate over a network, often using HTTP/REST or gRPC.

##### Load Balancers and Reverse Proxies

1. **Load Balancers**: These devices or services distribute incoming network traffic across multiple servers to ensure no single server is overwhelmed with too much traffic. This distribution ensures high availability and reliability by redirecting requests only to online servers that can handle them. Load balancing for web servers is often done by reverse proxies. Examples include AWS Elastic Load Balancing, HAProxy, and F5 BIG-IP.
2. **Reverse Proxies**: A reverse proxy sits before one or more web servers and forwards client requests to the appropriate web server. While they can also perform load balancing, their primary purpose is to provide a single access point and control for back-end servers. Examples include NGINX, Apache with mod_proxy, and Varnish.


![[httprs2.png]]


##### Role of Caching Mechanisms

Caching is a technique used to store and reuse previously fetched data or computed results to speed up subsequent requests and computations. In the context of web infrastructure:

1. **Content Caching**: By storing web content that doesn't change frequently (like images, CSS, and JS files), caching mechanisms can reduce the load on web servers and speed up content delivery to users.
2. **Database Query Caching**: Databases can cache the results of frequent queries, reducing the time and resources needed to fetch the same data repeatedly.
3. **Full-page Caching**: Entire web pages can be cached, so they don't need to be regenerated for each user. This is especially useful for websites with high traffic.
4. **Edge Caching/CDNs**: Content Delivery Networks (CDNs) cache content closer to the users (at the "edge" of the network), reducing latency and speeding up access for users around the world.
5. **API Caching**: Caching the responses can significantly reduce back-end processing for APIs that serve similar requests repeatedly.

![[httprs3.png]]

Caching, when implemented correctly, can significantly enhance the performance and responsiveness of web applications. However, managing caches properly is essential to avoid serving stale or outdated content.

---

### Behind The Scenes

##### Understanding HTTP Request Structure

Every HTTP request comprises two main parts: the header and the body.

![[httprs4.png]]

1. **Request Line**: The first line of the request `POST /admin/login HTTP/1.1` is the request line. It consists of at least three items. First is the method, which in this case is "POST". The method is a one-word command that tells the server what to do with the resource. Second is the path component of the URL for the request. The path identifies the resource on the server, which in this case is "/admin/login". Lastly, the HTTP version number shows the HTTP specification to which the client has tried to make the message comply. Note that HTTP/2 and HTTP/1.1 have different structures.
2. **Request Headers**: This section contains metadata about the request, such as the type of content being sent, the desired response format, and authentication tokens. It's like the envelope of a letter, providing information about the sender, receiver, and the nature of the content inside.
3. **Message Body**: This is the actual content of the request. The body might be empty for a GET request, but for a POST request, it could contain form data, JSON payloads, or file uploads.

##### Content-Length Header

The Content-Length header indicates the ***request or response body size in bytes.*** It informs the receiving server how much data to expect, ensuring the entire content is received.

##### Transfer-Encoding Header

The Transfer-Encoding header specifies how the message body is formatted or encoded. One of the most common values for this header is **chunked**, which means the body is sent in multiple chunks, each with its size defined. Other directives of this header are compress, deflate, and gzip.
***In chunked encoding, each chunk starts with the number of bytes in that chunk (in hexadecimal), followed by the actual data and a new line.***

##### How Headers Affect Request Processing

Headers play an important role in guiding the server to process the request. This is because they determine how to parse the request body and influence caching behaviours. They can also affect authentication, redirection, and other server responses.

![[httprs5.png]]

This is why when headers like Content-Length and Transfer-Encoding are manipulated or misinterpreted, it can lead to vulnerabilities. For example, if a server receives both headers and needs to handle them correctly, it might misinterpret where one request ends and another begins.

#### HTTP Request Smuggling Origin

HTTP Request Smuggling primarily occurs due to discrepancies in how different servers (like a front-end server and a back-end server) interpret HTTP request boundaries. For example:

1. If both Content-Length and Transfer-Encoding headers are present, ambiguities can arise.
2. Some components prioritize Content-Length, while others prioritize Transfer-Encoding.
3. This discrepancy can lead to one component believing the request has ended while another thinks it's still ongoing, leading to smuggling.

**Example:** Suppose a front-end server uses the Content-Length header to determine the end of a request while a back-end server uses the Transfer-Encoding header. An attacker can craft a request that appears to have one boundary to the front-end server but a different boundary to the back-end server. This can lead to one request being "smuggled" inside another, causing unexpected behaviour and potential vulnerabilities.


![[httprs6.png]]


---

### Request Smuggling CL.TE


##### Introduction to CL.TE request smuggling

**CL.TE** stands for **Content-Length/Transfer-Encoding**. The name **CL.TE** comes from the two headers involved: **Content-Length** and **Transfer-Encoding**. In CL.TE technique, the attacker exploits discrepancies between how different servers (typically a front-end and a back-end server) prioritize these headers. For example:

- The proxy uses the Content-Length header to determine the end of a request.
- The back-end server uses the Transfer-Encoding header.

![[httprs7.png]]

Because of this discrepancy, it's possible to craft ambiguous requests that are interpreted differently by each server. For example, Imagine sending a request with both `Content-Length` and `Transfer-Encoding` headers. The front-end server might use the Content-Length header and think the request ends at a certain point due to the provided number of bytes. In contrast, the back-end server, relying on the Transfer-Encoding header, might interpret the request differently, leading to unexpected behaviour.

##### Exploiting CL.TE for Request Smuggling

To exploit the CL.TE technique, an attacker crafts a request that includes both headers, ensuring that the front-end and back-end servers interpret the request boundaries differently. For example, an attacker sends a request like:

```HTTP
POST /search HTTP/1.1
Host: example.com
Content-Length: 80
Transfer-Encoding: chunked

0

POST /update HTTP/1.1
Host: example.com
Content-Length: 13
Content-Type: application/x-www-form-urlencoded

isadmin=true
```

Here, the front-end server sees the `Content-Length` of 80 bytes and believes the request ends after  `isadmin=true`. However, the back-end server sees the `Transfer-Encoding: chunked` and interprets the `0` as the end of a chunk, making the second request the start of a new chunk. This can lead to the back-end server treating the `POST /update HTTP/1.1` as a separate, new request, potentially giving the attacker unauthorized access.

##### Incorrect Content-Length

When creating a request smuggling payload, if the `Content-Length` is not equal to the actual length of the content, several problems might arise. First, the server might process only the portion of the request body that matches the `Content-Length`. This could result in the smuggled part of the request being ignored or not processed as intended.


---

### Request Smuggling TE.CL

##### Introduction to TE.CL Technique

**TE.CL** stands for **Transfer-Encoding/Content-Length**. This technique is the opposite of the CL.TE method. In the TE.CL approach, the discrepancy in header interpretation is flipped because the front-end server uses the Transfer-Encoding header to determine the end of a request, and the back-end server uses the Content-Length header.

The TE.CL technique arises when the proxy prioritizes the `Transfer-Encoding` header while the back-end server prioritizes the `Content-Length` header.

![How TE.CL smuggling works](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/5a35839a4359c02cfe2c785e6beddc8f.svg)


**Example:** If an attacker sends a request with both headers, the front-end server or proxy might interpret the request based on the `Transfer-Encoding` header, while the back-end server might rely on the `Content-Length` header. This difference in interpretation might interpret the request differently, leading to unexpected behaviour.

##### Exploiting TE.CL for Request Smuggling

To exploit the TE.CL technique, an attacker crafts a specially designed request that includes both the **Transfer-Encoding** and **Content-Length** headers, aiming to create ambiguity in how the front-end and back-end servers interpret the request.  

For example, an attacker sends a request like:

```HTTP
POST / HTTP/1.1
Host: example.com
Content-Length: 4
Transfer-Encoding: chunked

4c
POST /update HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

isadmin=true
0
```

In the above payload, the front-end server sees the `Transfer-Encoding: chunked` header and processes the request as chunked. The `4c` (hexadecimal for 76) indicates that the next 76 bytes are part of the current request's body. The front-end server considers everything up to the `0` (indicating the end of the chunked message) as part of the body of the first request.

The back-end server, however, uses the Content-Length header, which is set to 4. It processes only the first 4 bytes of the request, not including the entire smuggled request `POST /update`. The remaining part of the request, starting from **POST /update**, is then interpreted by the back-end server as a separate, new request.

The smuggled request is processed by the back-end server as if it were a legitimate, separate request. This request includes the `isadmin=true` parameter, which could potentially elevate the attacker's privileges or alter data on the server, depending on the application's functionality.

---

### Transfer Encoding Obfuscation

##### Introduction to TE.TE Technique

**Transfer Encoding Obfuscation,** also known as **TE.TE** stands for **Transfer-Encoding/Transfer-Encoding**. Unlike the CL.TE or TE.CL methods, the TE.TE technique arises when both the front-end and the back-end servers use the Transfer-Encoding header. In TE.TE technique, the attacker takes advantage of the servers' inconsistent handling of  Transfer-Encoding present in the HTTP headers.

The TE.TE vulnerability doesn't always require multiple Transfer-Encoding headers. Instead, it often involves a single, malformed Transfer-Encoding header that is interpreted differently by the front-end and back-end servers. In some cases, the front-end server might ignore or strip out the malformed part of the header and process the request normally, while the back-end server might interpret the request differently due to the malformed header, leading to request smuggling.  


![How TE.TE request smuggling works](https://tryhackme-images.s3.amazonaws.com/user-uploads/645b19f5d5848d004ab9c9e2/room-content/185d7a5ec0d23cad9c5947ed338f00cb.svg)


**Example**: An attacker might send a request with two `Transfer-Encoding` headers, one with a value the front-end server understands, and another with a value the front-end server ignores but the back-end server understands. This discrepancy can cause the servers to disagree on the request's boundaries.

#### Exploiting TE.TE for Request Smuggling

To exploit the TE.TE technique, an attacker may craft a request that includes Transfer-Encoding headers that use different encodings. For example, an attacker sends a request like:

```HTTP
POST / HTTP/1.1
Host: example.com
Content-length: 4
Transfer-Encoding: chunked
Transfer-Encoding: chunked1

4c
POST /update HTTP/1.1
Host: example.com
Content-length: 15

isadmin=true
0
```

In the above payload, the front-end server encounters two `Transfer-Encoding` headers. The first one is a standard chunked encoding, but the second one, `chunked1`, is non-standard. Depending on its configuration, the front-end server might process the request based on the first `Transfer-Encoding: chunked` header and ignore the malformed `chunked1`, interpreting the entire request up to the `0` as a single chunked message.

The back-end server, however, might handle the malformed `Transfer-Encoding: chunked1` differently. It could either reject the malformed part and process the request similarly to the front-end server or interpret the request differently due to the presence of the non-standard header. If it processes only the first 4 bytes as indicated by the `Content-length: 4`, the remaining part of the request starting from `POST /update` is then treated as a separate, new request.

The smuggled request with the `isadmin=true` parameter is processed by the back-end server as if it were a legitimate, separate request. This could lead to unauthorized actions or data modifications, depending on the server's functionality and the nature of the /update endpoint.

---

### Conclusion

HTTP request smuggling occurs due to differences in interpreting request headers by servers, mainly through the manipulation of Content-Length and Transfer-Encoding headers, leading to unclear request boundaries. This can cause servers in environments with both front-end and back-end components to incorrectly define request limits, potentially allowing security bypasses and unauthorized data access.

##### Mitigation Approaches

1. **Uniform Header Handling:** Ensure all servers handle headers in the same manner to prevent smuggling opportunities.
2. **Embrace HTTP/2:** Switching to HTTP/2 can enhance the management of request boundaries, reducing the risk of smuggling.
3. **Ongoing Surveillance and Reviews:** Keep an eye on server traffic for smuggling signs and perform periodic checks to maintain secure server setups.
4. **Team Awareness:** Make sure both development and operations teams understand the dangers of request smuggling and the preventive measures.

To summarize, despite the significant risk posed by HTTP request smuggling, it is a manageable issue with the right knowledge, strategies, and resources. Emphasizing security in web application development and management is crucial for protecting both the application and its users.

