
What is CSRF?

CSRF is a type of security vulnerability where an attacker tricks a user's web browser into performing an unwanted action on a trusted site where the user is authenticated. This is achieved by exploiting the fact that the browser includes any relevant cookies (credentials) automatically, allowing the attacker to forge and submit unauthorised requests on behalf of the user (through the browser). The attacker's website may contain HTML forms or JavaScript code that is intended to send queries to the targeted web application.

A CSRF attack has **three** essential phases:

- The attacker already knows the format of the web application's requests to carry out a particular task and sends a malicious link to the user.
- The victim's identity on the website is verified, typically by cookies transmitted automatically with each domain request and clicks on the link shared by the attacker. This interaction could be a click, mouse over, or any other action.
![phases of csrf](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/61e098019fb1d8611c0aa16f23c34633.svg)

- Insufficient security measures prevent the web application from distinguishing between authentic user requests and those that have been falsified.

The risks associated with CSRF include:

- **Unauthorised Access**: Attackers can access and control a user's actions, putting them at risk of losing money, damaging their reputation, and facing legal consequences.
- **Exploiting Trust**: CSRF exploits the trust websites put in their users, undermining the sense of security in online browsing.
- **Stealthy Exploitation**: CSRF works quietly, using standard browser behaviour without needing advanced malware. Users might be unaware of the attack, making them susceptible to repeated exploitation.


Traditional CSRF

Conventional CSRF attacks frequently concentrate on state-changing actions carried out by submitting forms. The victim is tricked into submitting a form without realising the associated data like cookies, URL parameters, etc. The victim's web browser sends an HTTP request to a web application form where the victim has already been authenticated. These forms are made to transfer money, modify account information, or alter an email address.

![traditional csrf](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/ef1cd0a1d90c6fbdaeed3b0a29987e01.svg)


- The victim is already logged on to his bank website. The attackers create a crafted malicious link and email it to the victim.
- The victim opens the email in the same browser.
- Once clicked, the malicious link enables the auto-transfer of the amount from the victim's browser to the attacker's bank account.


XMLHttpRequest CSRF

An asynchronous CSRF exploitation occurs when operations are initiated without a complete page request-response cycle. This is typical of contemporary online apps that leverage asynchronous server communication (via **XMLHttpRequest** or the **Fetch** API) and JavaScript to produce more dynamic user interfaces. These attacks use asynchronous calls instead of the more conventional form submissions. Still, they exploit the same trust relationship between the user and the online service.


Flash-based CSRF

The term "Flash-based CSRF" describes the technique of conducting a CSRF attack by taking advantage of flaws in Adobe Flash Player components. Internet applications with features like **interactive content, video streaming, and intricate animations**![flash based csrf](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/328966e16f936388539965b9bf110581.svg) have been made possible with Flash. But over time, security flaws in Flash, particularly those that can be used to launch CSRF attacks, have become a major source of worry.


***A covert technique known as hidden link/image exploitation*in CSRF involves an attacker inserting a 0x0 pixel image or a link into a webpage that is nearly undetectable to the user. Typically, the `src or href` element of the image is set to a destination URL intended to act on the user's behalf without the user's awareness. It takes benefit of the fact that the user's browser transfers credentials like cookies automatically.***

```html
<!-- Website --> 
<a href="https://mybank.thm/transfer.php" target="_blank">Click Here</a>  
<!-- User visits attacker's website while authenticated -->
```


```html
<a href="http://mybank.thm:8080/dashboard.php?to_account=GB82MYBANK5698&amount=1000" target="_blank">Click Here to Redeem</a>
```

Securing the Breach

- From a **pentester/red teamer perspective**, it is vital to pentest each request and response parameter of the form.  
    
- ***From a secure coder perspective, it is important to ensure that each request submitted to the web server carries a unique token so that the server can identify if it is clicked through a valid source.***


##### Double Submit Cookie Bypass

**A CSRF token is a unique, unpredictable value associated with a user's session, ensuring each request comes from a legitimate source. One effective implementation is the Double Submit Cookies technique, where a cookie value corresponds to a value in a hidden form field. When the server receives a request, it checks that the cookie value matches the form field value, providing an additional layer of verification.**



###### How it works

- **Token Generation**: When a user logs in or initiates a session, the server generates a unique CSRF token.![how double submit cookie attack works](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/63340a0ec9d725fcb640fbc3eedcbcf3.svg) This token is sent to the user's browser both as a cookie (CSRF-Token cookie) and embedded in hidden form fields of web forms where actions are performed (like money transfers).
- **User Action**: Suppose the user wants to transfer money. They fill out the transfer form on the website, which includes the hidden CSRF token.
- **Form Submission**: Upon submitting the form, two versions of the CSRF token are sent to the server: one in the cookie and the other as part of the form data.
- **Server Validation**: The server then checks if the CSRF token in the cookie matches the one sent in the form data. If they match, the request is considered legitimate and processed; if not, the request is rejected.


##### Possible Vulnerable Scenarios

Despite its effectiveness, it's crucial to acknowledge that hackers are persistent and have identified various methods to bypass Double Submit Cookies:

- Session Cookie Hijacking (Man in the Middle Attack): If the CSRF token is not appropriately isolated and safeguarded from the session, an attacker may also be able to access it by other means (such as malware, network spying, etc.).
- **Subverting the Same-Origin Policy (Attacker Controlled Subdomain)**: An attacker can set up a situation where the browser's same-origin policy is broken. Browser vulnerabilities or deceiving the user into sending a request through an attacker-controlled subdomain with permission to set cookies for its parent domain could be used.
- **Exploiting XSS Vulnerabilities**: An attacker may be able to obtain the CSRF token from the cookie or the page itself if the web application is susceptible to Cross-Site Scripting (XSS). By creating fraudulent requests with the double-submitted cookie CSRF token, the attacker can get around the defence once they have the CSRF token.
- **Predicting or Interfering with Token Generation**: An attacker may be able to guess or modify the CSRF token if the tokens are not generated securely and are predictable or if they can tamper with the token generation process.
- **Subdomain Cookie Injection**: Injecting cookies into a user's browser from a related subdomain is another potentially sophisticated technique that might be used. This could fool the server's CSRF protection system by appearing authentic to the main domain.


##### Samesite Cookie Bypass

SameSite cookies. These cookies come with a special attribute designed to control when they are sent along with cross-site requests. Implementing the SameSite cookie property is a reliable safeguard against cross-origin data leaks, CSRF, and XSS attacks. Depending on the request's context, it tells the browser when to transmit the cookie. **Strict**, **Lax**, and **None** are the three potential values for the attribute.

The most substantial level of protection is offered by setting it to strict, which guarantees that the cookie is only sent if the request comes from the same origin as the cookie. Specific cross-site usage is permitted by lax, such as top-level navigations, which are less likely to raise red flags. None of them need the secure attribute, and all requests made by websites that belong to third parties will send cookies.


##### Different Types of SameSite Cookies![different type of samesite cookies](https://tryhackme-images.s3.amazonaws.com/user-uploads/62a7685ca6e7ce005d3f3afe/room-content/1cea15bc2776b3af859b1259bb399c1f.svg)

- **Lax**: Lax SameSite cookies are like a friendly neighbour. They provide a moderate level of protection by allowing cookies to be sent in top-level navigations and safe HTTP methods like GET, HEAD, and OPTIONS. This means that cookies will not be sent with cross-origin POST requests, helping to mitigate certain types of CSRF attacks. However, cookies are still included in GET requests initiated by external websites, which may pose a security risk if sensitive information is stored in cookies.
- **Strict**: Strict SameSite cookies act as vigilant guards. They offer the highest level of protection by restricting cookies to be sent only in a first-party context. This means that cookies are only sent with requests originating from the same site that set the cookie, effectively preventing cross-site request forgery attacks. By enforcing strict isolation between different origins, strict SameSite cookies significantly enhance the security of web applications, especially in scenarios where sensitive user data is involved.
- **None**: None SameSite cookies behave like carefree globetrotters. They are sent with both first-party and cross-site requests, making them convenient for scenarios where cookies need to be accessible across different origins. However, to prevent potential security risks associated with cross-site requests, None SameSite cookies require the **Secure** attribute if the request is made over HTTPS. This ensures that cookies are only transmitted over secure connections, reducing the likelihood of interception or tampering by malicious actors during transit.
