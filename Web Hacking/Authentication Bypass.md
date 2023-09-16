

1) **Username Enumberation**

Website error messages are great resources for collating this information to build our list of valid usernames.

Example : If you try entering the username **admin** and fill in the other form fields with fake information, you'll see we get the error **An account with this username already exists**. We can use the existence of this error message to produce a list of valid usernames already signed up on the system

```sh
ffuf -w /usr/share/wordlists/SecLists/SecLists-master/Usernames/Names/names.txt -X POST -d "username=FUZZ&email=x&password=x&cpassword=x" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.89.98/customers/signup -mr "username already exists"
```

-H : Set HTTP header
-X : Set HTTP method
-mr : to specify regex pattern
-d : data to be sent with POST request

2) **Brute Force**

A brute force attack is an automated process that tries a list of commonly used passwords against either a single username or, like in our case, a list of usernames.

```sh
ffuf -w valid_usernames.txt:FUZZUSER,/usr/share/wordlists/SecLists/SecLists-master/Passwords/Common-Credentials/10-million-password-list-top-100.txt:FUZZPASS -X POST -d "username=FUZZUSER&password=FUZZPASS" -H "Content-Type: application/x-www-form-urlencoded" -u http://10.10.89.98/customers/login -fc 200
```

-fc : filter by the status code

3) **Logic Flaw**

Sometimes authentication processes contain logic flaws. A logic flaw is when the typical logical path of an application is either bypassed, circumvented or manipulated by a hacker

Code Example : 
```
if( url.substr(0,6) === '/admin') {
    # Code to check user is an admin
} else {
    # View Page
}
```

Because the above PHP code example uses three equals signs ( === ) it's looking for an exact match on the string, including the same letter casing. The code presents a logic flaw because an unauthenticated user requesting **/adMin** will not have their privileges checked and have the page displayed to them, totally bypassing the authentication checks.

4) **Cookie Tampering**

Examining and editing the cookies set by the web server during your online session can have multiple outcomes, such as unauthenticated access, access to another user's account, or elevated privileges

The contents of some cookies can be in plain text, and it is obvious what they do. Take, for example, if these were the cookie set after a successful login:
Set-Cookie: logged_in=true; Max-Age=3600; Path=/
Set-Cookie: admin=false; Max-Age=3600; Path=/

**Hashing**

Sometimes cookie values can look like a long string of random characters; these are called hashes which are an irreversible representation of the original text. Here are some examples that you may come across:

|   |   |   |
|---|---|---|
|**Original String**|**Hash Method**|**Output**|
|1|md5|c4ca4238a0b923820dcc509a6f75849b|
|1|sha-256|6b86b273ff34fce19d6b804eff5a3f5747ada4eaa22f1d49c01e52ddb7875b4b|
|1|sha-512|4dff4ea340f0a823f15d3f4f01ab62eae0e5da579ccb851f8db9dfe84c58b2b37b89903a740e1ee172da793a6e79d560e5f7f9bd058a12a280433ed6fa46510a|
|1|sha1|356a192b7913b04c54574d18c28d46e6395428ab|


**Encoding**

Encoding is similar to hashing in that it creates what would seem to be a random string of text, but in fact, the encoding is reversible. Encoding allows us to convert binary data into human-readable text that can be easily and safely transmitted over mediums that only support plain text ASCII characters.

**Set-Cookie:** session=eyJpZCI6MSwiYWRtaW4iOmZhbHNlfQ== ; Max-Age=3600; Path=/
This string base64 decoded has the value of **{"id":1,"admin": false}**
