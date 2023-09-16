
***Insecure Direct Object Refrence***

## **Encoded IDs**

When passing data from page to page either by post data, query strings, or cookies, web developers will often first take the raw data and encode it. Encoding ensures that the receiving web server will be able to understand the contents. Encoding changes binary data into an ASCII string commonly using the `a-z, A-Z, 0-9 and =` character for padding. The most common encoding technique on the web is base64 encoding and can usually be pretty easy to spot

![[idor1.png]]


## **Hashed IDs**

Hashed IDs are a little bit more complicated to deal with than encoded ones, but they may follow a predictable pattern, such as being the hashed version of the integer value. For example, the Id number 123 would become 202cb962ac59075b964b07152d234b70 if md5 hashing were in use.

## **Unpredictable IDs**

If the Id cannot be detected using the above methods, an excellent method of IDOR detection is to create two accounts and swap the Id numbers between them. If we can view the other users' content using their Id number while still being logged in with a different account (or not logged in at all), you've found a valid IDOR vulnerability.


## **Where are they located?**

The vulnerable endpoint you're targeting may not always be something we see in the address bar. It could be content your browser loads in via an AJAX request or something that we find referenced in a JavaScript file. 
Sometimes endpoints could have an unreferenced parameter that may have been of some use during development and got pushed to production. 

Example  we may notice a call to **/user/details** displaying your user information (authenticated through your session). But through an attack known as parameter mining, wediscover a parameter called **user_id** that we can use to display other users' information, for example, **/user/details?user_id=123**
