
***Insecure deserialization is replacing data processed by an application with malicious code; allowing anything from DoS (Denial of Service) to RCE (Remote Code Execution) that the attacker can use to gain a foothold in a pentesting scenario.***

Specifically, this malicious code leverages the legitimate serialization and deserialization process used by web applications.

- Low exploitability. This vulnerability is often a case-by-case basis - there is no reliable tool/framework for it. Because of its nature, attackers need to have a good understanding of the inner-workings of the ToE.

- The exploit is only as dangerous as the attacker's skill permits, more so, the value of the data that is exposed. For example, someone who can only cause a DoS will make the application unavailable. The business impact of this will vary on the infrastructure - some organisations will recover just fine, others, however, will not.

***At summary, ultimately, any application that stores or fetches data where there are no validations or integrity checks in place for the data queried or retained***

A few examples of applications of this nature are:

- E-Commerce Sites  
- Forums  
- API's  
- Application Runtimes (Tomcat, Jenkins, Jboss, etc)


#### Objects

A prominent element of object-oriented programming (OOP), objects are made up of two things:

- State

- Behaviour

Simply, objects allow you to create similar lines of code without having to do the leg-work of writing the same lines of code again.
For example, a lamp would be a good object. Lamps can have different types of bulbs, this would be their state, as well as being either on/off - their behaviour!
Rather than having to accommodate every type of bulb and whether or not that specific lamp is on or off, you can use methods to simply alter the state and behaviour of the lamp.


#### Deserialization

A Tourist approaches you in the street asking for directions. They're looking for a local landmark and got lost. Unfortunately, English isn't their strong point and nor do you speak their dialect either. What do you do? You draw a map of the route to the landmark because pictures cross language barriers, they were able to find the landmark. Nice! You've just serialised some information, where the tourist then deserialised it to find the landmark.

***Serialisation is the process of converting objects used in programming into simpler, compatible formatting for transmitting between systems or networks for further processing or storage.
Alternatively, deserialisation is the reverse of this; converting serialised information into their complex form - an object that the application will understand.***

Say you have a password of "password123" from a program that needs to be stored in a database on another system. To travel across a network this string/output needs to be converted to binary. Of course, the password needs to be stored as "password123" and not its binary notation. Once this reaches the database, it is converted or deserialised back into "password123" so it can be stored.


![[serialization&deserialization.png]]

***Simply, insecure deserialization occurs when data from an untrusted party (I.e. a hacker) gets executed because there is no filtering or input validation; the system assumes that the data is trustworthy and will execute it no holds barred.***


#### Cookies

***Cookies are an essential tool for modern websites to function. Tiny pieces of data, these are created by a website and stored on the user's computer.
Cookies are not permanent storage solutions like databases. Some cookies such as session ID's will clear when the browser is closed, others, however, last considerably longer. This is determined by the "Expiry" timer that is set when the cookie is created.***


|   |   |   |
|---|---|---|
|Attribute|Description|Required?|
|Cookie Name|The Name of the Cookie to be set|Yes|
|Cookie Value|Value, this can be anything plaintext or encoded|Yes|
|Secure Only|If set, this cookie will only be set over HTTPS connections|No|
|Expiry|Set a timestamp where the cookie will be removed from the browser|No|
|Path|The cookie will only be sent if the specified URL is within the req|No|


Cookies can be set in various website programming languages. For example, Javascript, PHP or Python to name a few.

_If we have a cookie like 'userType:user' , we can try changing it to 'userType:admin', if insecure deserialization, we can become admin_


