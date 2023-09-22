
SQL (Structured Query Language) Injection, mostly referred to as SQLi, is an attack on a web application database server that causes malicious queries to be executed. When a web application communicates with a database using input from a user that hasn't been properly validated, there runs the potential of an attacker being able to steal, delete or alter private and customer data and also attack the web applications authentication methods to private or customer areas.


**What is a database?**

A database is a way of electronically storing collections of data in an organised manner. A database is controlled by a DBMS which is an acronym for  Database Management System, DBMS's fall into two camps **Relational** or **Non-Relational**

Within a DBMS, you can have multiple databases, each containing its own set of related data. For example, you may have a database called "**shop**". Within this database, you want to store information about products available to **purchase**, **users** who have signed up to your online shop, and information about the **orders** you've received. You'd store this information separately in the database using something called tables, the tables are identified with a unique name for each one


![[databases.png]]


**What are tables?**

A table is made up of columns and rows, a useful way to imagine a table is like a grid with the columns going across the top from left to right containing the name of the cell and the rows going from top to bottom with each one having the actual data.


![[Databsetable.png]]


**Columns:**

Each column, better referred to as a field has a unique name per table. When creating a column, you also set the type of data it will contain, common ones being integer (numbers), strings (standard text) or dates. Some databases can contain much more complex data, such as geospatial, which contains location information. Setting the data type also ensures that incorrect information isn't stored, such as the string "hello world" being stored in a column meant for dates. If this happens, the database server will usually produce an error message. A column containing an integer can also have an auto-increment feature enabled; this gives each row of data a unique number that grows (increments) with each subsequent row, doing so creates what is called a **key** field, a key field has to be unique for every row of data which can be used to find that exact row in SQL queries.

**Rows:**

Rows or records are what contains the individual lines of data. When you add data to the table, a new row/record is created, and when you delete data, a row/record is removed.


**Relational Vs Non-Relational Databases:**  
A relational database, stores information in tables and often the tables have shared information between them, they use columns to specify and define the data being stored and rows to actually store the data. The tables will often contain a column that has a unique ID (primary key) which will then be used in other tables to reference it and cause a relationship between the tables, hence the name **relational** database.  

  
Non-relational databases sometimes called NoSQL on the other hand is any sort of database that doesn't use tables, columns and rows to store the data, a specific database layout doesn't need to be constructed so each row of data can contain different information which can give more flexibility over a relational database.  Some popular databases of this type are MongoDB, Cassandra and ElasticSearch.


SQL (Structured Query Language) is a feature-rich language used for querying databases, these SQL queries are better referred to as statements.Although somewhat similar, some databases servers have their own syntax and slight changes to how things work. All of these examples are based on a MySQL database


**SELECT**

the SELECT query used to retrieve data from the database.

`select * from users;`

|   |   |   |
|---|---|---|
|**id**|**username**|**password**|
|1|jon|pass123|
|2|admin|p4ssword|
|3|martin|secret123|

The first-word SELECT tells the database we want to retrieve some data, the * tells the database we want to receive back all columns from the table. For example, the table may contain three columns (id, username and password). "from users" tells the database we want to retrieve the data from the table named users. Finally, the semicolon at the end tells the database that this is the end of the query.

`select username,password from users;`

|   |   |
|---|---|
|**username**|**password**|
|jon|pass123|
|admin|p4ssword|
|martin|secret123|

`select * from users LIMIT 1;`

|   |   |   |
|---|---|---|
|**id**|**username**|**password**|
|1|jon|pass123|


"LIMIT 1" clause forces the database only to return one row of data. Changing the query to "LIMIT 1,1" forces the query to skip the first result, and then "LIMIT 2,1" skips the first two results, and so on


The WHERE this is how we can finely pick out the exact data we require by returning data that matches our specific clauses:

`select * from users where username='admin';`

|   |   |   |
|---|---|---|
|**id**|**username**|**password**|
|2|admin|p4ssword|



Using the like clause allows you to specify data that isn't an exact match but instead either starts, contains or ends with certain characters by choosing where to place the wildcard character represented by a percentage sign %.

  

`select * from users where username like 'a%';`

|   |   |   |
|---|---|---|
|**id**|**username**|**password**|
|2|admin|p4ssword|

This returns any rows with username beginning with the letter a.


`select * from users where username like '%n';`

|   |   |   |
|---|---|---|
|**id**|**username**|**password**|
|1|jon|pass123|
|2|admin|p4ssword|
|3|martin|secret123|

This returns any rows with username ending with the letter n.


**UNION**

The UNION statement combines the results of two or more SELECT statements to retrieve data from either single or multiple tables; the rules to this query are that the UNION statement must retrieve the same number of columns in each SELECT statement, the columns have to be of a similar data type and the column order has to be the same

INSERT

The **INSERT** statement tells the database we wish to insert a new row of data into the table. **"into users"** tells the database which table we wish to insert the data into, **"(username,password)"** provides the columns we are providing data for and then **"values ('bob','password');"** provides the data for the previously specified columns.

`insert into users (username,password) values ('bob','password123');`  

|   |   |   |
|---|---|---|
|**id**|**username**|**password**|
|1|jon|pass123|
|2|admin|p4ssword|
|3|martin|secret123|
|4|bob|password123|


## UPDATE

The **UPDATE** statement tells the database we wish to update one or more rows of data within a table. You specify the table you wish to update using "**update %tablename% SET**" and then select the field or fields you wish to update as a comma-separated list such as "**username='root',password='pass123'**" then finally similar to the SELECT statement, you can specify exactly which rows to update using the where clause such as "**where username='admin;**".

`update users SET username='root',password='pass123' where username='admin';`

|   |   |   |
|---|---|---|
|**id**|**username**|**password**|
|1|jon|pass123|
|2|root|pass123|
|3|martin|secret123|
|4|bob|password123|


## DELETE

The **DELETE** statement tells the database we wish to delete one or more rows of data. Apart from missing the columns you wish to be returned, the format of this query is very similar to the SELECT. You can specify precisely which data to delete using the **where** clause and the number of rows to be deleted using the **LIMIT** clause.

  

`delete from users where username='martin';`

|   |   |   |
|---|---|---|
|**id**|**username**|**password**|
|1|jon|pass123|
|2|root|pass123|
|4|bob|password123|


`delete from users;`

  

Because no WHERE clause was being used in the query, all the data is deleted in the table.  

|   |   |   |
|---|---|---|
|**id**|**username**|**password**|


---


**What is SQL Injection?**
The point wherein a web application using SQL can turn into SQL Injection is when user-provided data gets included in the SQL query.  

**What does it look like?**  
Take the following scenario where you've come across an online blog, and each blog entry has a unique id number. The blog entries may be either set to public or private depending on whether they're ready for public release. The URL for each blog entry may look something like this:  
  
`https://website.thm/blog?id=1`  
  
From the URL above, you can see that the blog entry been selected comes from the id parameter in the query string. The web application needs to retrieve the article from the database and may use an SQL statement that looks something like the following:  
  
`SELECT * from blog where id=1 and private=0 LIMIT 1;`

`SELECT * from blog where id=2;-- and private=0 LIMIT 1;`  
  
**The semicolon in the URL signifies the end of the SQL statement, and the two dashes cause everything afterwards to be treated as a comment**. By doing this, you're just, in fact, running the query:  
  
`SELECT * from blog where id=2;--`  
  
Which will return the article with an id of 2 whether it is set to public or not.

---

**In-Band SQL Injection**

In-Band SQL Injection is the easiest type to detect and exploit; In-Band just refers to the same method of communication being used to exploit the vulnerability and also receive the results, for example, discovering an SQL Injection vulnerability on a website page and then being able to extract data from the database to the same page.

#### **Error-Based SQL Injection**

This type of SQL Injection is the most useful for easily obtaining information about the database structure as error messages from the database are printed directly to the browser screen. This can often be used to enumerate a whole database.


#### **Union-Based SQL Injection**

This type of Injection utilises the SQL UNION operator alongside a SELECT statement to return additional results to the page. This method is the most common way of extracting large amounts of data via an SQL Injection vulnerability.

 **information_schema** database; every user of the database has access to this, and it contains information about all the databases and tables the user has access to.


**Blind SQLi**

Unlike In-Band SQL injection, where we can see the results of our attack directly on the screen, blind SQLi is when we get little to no feedback to confirm whether our injected queries were, in fact, successful or not, this is because the error messages have been disabled, but the injection still works regardless.

**Authentication Bypass**

One of the most straightforward Blind SQL Injection techniques is when bypassing authentication methods such as login forms. In this instance, we aren't that interested in retrieving data from the database; We just want to get past the login. 

Login forms that are connected to a database of users are often developed in such a way that the web application isn't interested in the content of the username and password but more whether the two make a matching pair in the users table. In basic terms, the web application is asking the database "do you have a user with the username **bob** and the password **bob123**?", and the database replies with either yes or no (true/false) and, depending on that answer, dictates whether the web application lets you proceed or not. 

Taking the above information into account, it's unnecessary to enumerate a valid username/password pair. We just need to create a database query that replies with a yes/true.


`select * from users where username='' and password='' OR 1=1;`

Because 1=1 is a true statement and we've used an **OR** operator, this will always cause the query to return as true, which satisfies the web applications logic that the database found a valid username/password combination and that access should be allowed.


**Boolean Based**

Boolean based SQL Injection refers to the response we receive back from our injection attempts which could be a true/false, yes/no, on/off, 1/0 or any response which can only ever have two outcomes. That outcome confirms to us that our SQL Injection payload was either successful or not. On the first inspection, you may feel like this limited response can't provide much information. Still, in fact, with just these two responses, it's possible to enumerate a whole database structure and contents.

`select * from users where username = '%username%' LIMIT 1;`

our first task is to establish the number of columns in the users table, which we can achieve by using the UNION statement.

`admin123' UNION SELECT 1,2,3;--`

Now that our number of columns has been established, we can work on the enumeration of the database. Our first task is discovering the database name. We can do this by using the built-in **database()** method and then using the **like** operator to try and find results that will return a true status.

`admin123' UNION SELECT 1,2,3 where database() like '%';--`

Now you move onto the next character of the database name until you find another **true** response, for example, 'sa%', 'sb%', 'sc%' etc. Keep on with this process until you discover all the characters of the database name, which is **sqli_three**.



**Time-Based**

A time-based blind SQL Injection is very similar to the above Boolean based, in that the same requests are sent, but there is no visual indicator of your queries being wrong or right this time. Instead, your indicator of a correct query is based on the time the query takes to complete. This time delay is introduced by using built-in methods such as **SLEEP(x)** alongside the UNION statement. The SLEEP() method will only ever get executed upon a successful UNION SELECT statement.


when trying to establish the number of columns in a table, you would use the following query:

`admin123' UNION SELECT SLEEP(5);--`

If there was no pause in the response time, we know that the query was unsuccessful, so like previously, we add another column:

`admin123' UNION SELECT SLEEP(5),2;--`


Out-of-Band SQL Injection isn't as common as it either depends on specific features being enabled on the database server or the web application's business logic, which makes some kind of external network call based on the results from an SQL query.  
  
An Out-Of-Band attack is classified by having two different communication channels, one to launch the attack and the other to gather the results. For example, the attack channel could be a web request, and the data gathering channel could be monitoring HTTP/DNS requests made to a service you control.

1) An attacker makes a request to a website vulnerable to SQL Injection with an injection payload.

2) The Website makes an SQL query to the database which also passes the hacker's payload.

3) The payload contains a request which forces an HTTP request back to the hacker's machine containing data from the database.


![[outOfBandSQLI.png]]


---

**Remediation**

As impactful as SQL Injection vulnerabilities are, developers do have a way to protect their web applications from them by following the below advice:

  

**Prepared Statements (With Parameterized Queries):**

In a prepared query, the first thing a developer writes is the SQL query and then any user inputs are added as a parameter afterwards. Writing prepared statements ensures that the SQL code structure doesn't change and the database can distinguish between the query and the data. As a benefit, it also makes your code look a lot cleaner and easier to read.

  

**Input Validation:**

Input validation can go a long way to protecting what gets put into an SQL query. Employing an allow list can restrict input to only certain strings, or a string replacement method in the programming language can filter the characters you wish to allow or disallow. 

  

**Escaping User Input:**

Allowing user input containing characters such as ' " $ \ can cause SQL Queries to break or, even worse, as we've learnt, open them up for injection attacks. Escaping user input is the method of prepending a backslash (**\**) to these characters, which then causes them to be parsed just as a regular string and not a special character.


---

### SQL Map

**Basic** commands:

|   |   |
|---|---|
|**Options**|**Description**|
|-u URL, --url=URL|Target URL|
|--data=DATA|Data string to be sent through POST (e.g. "id=1")|
|--random-agent|Use randomly selected HTTP User-Agent header value|
|-p TESTPARAMETER|Testable parameter(s)|
|--level=LEVEL|Level of tests to perform (1-5, default 1)|
|--risk=RISK|Risk of tests to perform (1-3, default 1)|


**Enumeration** commands:

These options can be used to enumerate the back-end database management system information, structure, and data contained in tables._

|   |   |
|---|---|
|Options|Description|
|-a, --all|Retrieve everything|
|-b, --banner|Retrieve DBMS banner|
|--current-user|Retrieve DBMS current user|
|--current-db|Retrieve DBMS current database|
|--passwords|Enumerate DBMS users password hashes|
|--dbs|Enumerate DBMS databases|
|--tables|Enumerate DBMS database tables|
|--columns|Enumerate DBMS database table columns|
|--schema|Enumerate DBMS schema|
|--dump|Dump DBMS database table entries|
|--dump-all|Dump all DBMS databases tables entries|
|--is-dba|Detect if the DBMS current user is DBA|
|-D \<DB NAME>|DBMS database to enumerate|
|-T \<TABLE NAME>|DBMS database table(s) to enumerate|
|-C COL|DBMS database table column(s) to enumerate|


 Operating System access commands

_These options can be used to access the back-end database management system on the target operating system._

|   |   |
|---|---|
|Options|Description|
|--os-shell|Prompt for an interactive operating system shell|
|--os-pwn|Prompt for an OOB shell, Meterpreter or VNC|
|--os-cmd=OSCMD|Execute an operating system command|
|--priv-esc|Database process user privilege escalation|
|--os-smbrelay|One-click prompt for an OOB shell, Meterpreter or VNC|


```sh
sqlmap -r <request_file> -p <vulnerable_parameter> --dbs
```

 -r to read the file, -p to supply the vulnerable parameter, and --dbs to enumerate the database.


After we have the databases,  extract tables from the database

**Using GET based Method**

```sh
sqlmap -u https://testsite.com/page.php?id=7 -D <database_name> --tables
```


**Using POST based Method**

```sh
sqlmap -r req.txt -p <vulnerable_parameter> -D <database_name> --tables
```


Once we have available tables,  gather the columns from the table

**Using GET based Method**

```sh
sqlmap -u https://testsite.com/page.php?id=7 -D <database_name> -T <table_name> --columns
```

**Using POST based Method**

```sh
sqlmap -r req.txt -D <database_name> -T <table_name> --columns
```


OR
we can simply dump all the available databases and tables using

**Using GET based Method**

```sh
sqlmap -u https://testsite.com/page.php?id=7 -D <database> --dump-all
```


**Using POST based Method**

```sh
sqlmap -r req.txt-p  -D <database_name> --dump-all
```


