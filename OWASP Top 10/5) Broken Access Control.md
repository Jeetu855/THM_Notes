
***Websites have pages that are protected from regular visitors, for example only the site's admin user should be able to access a page to manage other users. If a website visitor is able to access the protected page/pages that they are not authorised to view, the access controls are broken.***


A regular visitor being able to access protected pages, can lead to the following:

- Being able to view sensitive information
- Accessing unauthorized functionality

**Scenario #1**: The application uses unverified data in a SQL call that is accessing account information:

pstmt.setString(1, request.getParameter("acct"));

ResultSet results = pstmt.executeQuery( );

An attacker simply modifies the ‘acct’ parameter in the browser to send whatever account number they want. If not properly verified, the attacker can access any user’s account.

\http://example.com/app/accountInfo?acct=notmyacct


Scenario #2: An attacker simply force browses to target URLs. Admin rights are required for access to the admin page.

\http://example.com/app/getappInfo
\http://example.com/app/admin_getappInfo

***Broken access control allows attackers to bypass authorization which can allow them to view sensitive data or perform tasks as if they were a privileged user.***


#### IDOR(Insecure Direct Object Reference)

IDOR, or Insecure Direct Object Reference, is the act of exploiting a misconfiguration in the way user input is handled, to access resources you wouldn't ordinarily be able to access. IDOR is a type of access control vulnerability.

For example, let's say we're logging into our bank account, and after correctly authenticating ourselves, we get taken to a URL like this \https://example.com/bank?account_number=1234. On that page we can see all our important bank details, and a user would do whatever they needed to do and move along their way thinking nothing is wrong.

There is however a potentially huge problem here, a hacker may be able to change the account_number parameter to something else like 1235, and if the site is incorrectly configured, then he would have access to someone else's bank information.

![[IDOR.png]]














