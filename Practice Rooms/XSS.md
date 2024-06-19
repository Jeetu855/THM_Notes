
XSS attacks rely on injecting a malicious script in a benign website to run on a user’s browser. In other words, XSS attacks exploit the user’s trust in the vulnerable web application, hence the damage.
Consequently, they bypass the **Same-Origin Policy (SOP)**; SOP is a security mechanism implemented in modern web browsers to prevent a malicious script on one web page from obtaining access to sensitive data on another page. SOP defines origin based on the protocol, hostname, and port. Consequently, a malicious ad cannot access data or manipulate the page or its functionality on another origin, such as an online shop or bank page. XSS dodges SOP as it is executing from the same origin.

- **Alert**: You can use the `alert()` function to display a JavaScript alert in a web browser. Try `alert(1)` or `alert('XSS')` (or `alert("XSS")`) to display an alert box with the number 1 or the text XSS.
- **Console log**: Similarly, you can display a value in the browser’s JavaScript console using `console.log()`. Try the following two examples in the console log: `console.log(1)` and `console.log("test text")` to display a number or a text string.
- **Encoding**: The JavaScript function `btoa("string")` encodes a string of binary data to create a base64-encoded ASCII string. This is useful to remove white space and special characters or encode other alphabets. The reverse function is `atob("base64_string")`.

- **Reflected XSS**: This attack relies on the user-controlled input reflected to the user. For instance, if you search for a particular term and the resulting page displays the term you searched for (**_reflected_**), the attacker would try to embed a malicious script within the search term.
- **Stored XSS**: This attack relies on the user input stored in the website’s database. For example, if users can write product reviews that are saved in a database (**_stored_**) and being displayed to other users, the attacker would try to insert a malicious script in their review so that it gets executed in the browsers of other users.
- **DOM-based XSS**: This attack exploits vulnerabilities within the Document Object Model (**DOM**) to manipulate existing page elements without needing to be reflected or stored on the server. This vulnerability is the least common among the three.


---

####  Reflected XSS

**Reflected XSS** is a type of XSS vulnerability where a malicious script is reflected to the user’s browser, often via a **crafted URL** or **form submission**. Consider a search query containing `<script>alert(document.cookie)</script>`; many users wouldn’t be suspicious about such a URL, even if they look at it up close. If processed by a vulnerable web application, it will be executed within the context of the user’s browser.

Vulnerable code

```javascript
const express = require('express');
const app = express();

app.get('/search', function(req, res) {
    var searchTerm = req.query.q;
    res.send('You searched for: ' + searchTerm);
});

app.listen(80);
```

Fixed Code

```javascript
const express = require('express');
const sanitizeHtml = require('sanitize-html');

const app = express();

app.get('/search', function(req, res) {
    const searchTerm = req.query.q;
    const sanitizedSearchTerm = sanitizeHtml(searchTerm);
    res.send('You searched for: ' + sanitizedSearchTerm);
});

app.listen(80);
```


**The solution is achieved by using the `sanitizeHtml()` from the `sanitize-html` library. This function removes unsafe elements and attributes. This includes removing script tags, among other elements that could be used for malicious purposes.**
**Another approach would be by using the `escapeHtml()` function instead of the `sanitizeHtml()` function. As the name indicates, the `escapeHtml()` function aims to escape characters such as `<`, `>`, `&`, `"`, and `'`.**

---

#### Stored XSS

Stored XSS begins with an attacker injecting a malicious script in an input field of a vulnerable web application. The vulnerability might lie in how the web application processes the data in the comment box, forum post, or profile information section. When other users access this stored content, the injected malicious script executes within their browsers. The script can perform a wide range of actions, from stealing session cookies to performing actions on behalf of the user without their consent.

Vulnerable Code

```javascript
app.get('/comments', (req, res) => {
  let html = '<ul>';
  for (const comment of comments) {
    html += `<li>${comment}</li>`;
  }
  html += '</ul>';
  res.send(html);
});
```

Fixed Code

```javascript
const sanitizeHtml = require('sanitize-html');

app.get('/comments', (req, res) => {
  let html = '<ul>';
  for (const comment of comments) {
    const sanitizedComment = sanitizeHtml(comment);
    html += `<li>${sanitizedComment}</li>`;
  }
  html += '</ul>';
  res.send(html);
});
```


---

#### DOM XSS

The DOM is a programming interface representing a web document as a tree. The DOM makes it possible to programmatically access and manipulate the different parts of a website using JavaScript. Let’s consider a practical example.



##### Some payloads

```javascript
<IMG SRC="jav&#x09;ascript:alert('XSS');">
<IMG SRC="jav&#x0A;ascript:alert('XSS');">
<IMG SRC="jav&#x0D;ascript:alert('XSS');">
```

