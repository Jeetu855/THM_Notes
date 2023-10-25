
When a webapp accidentally divulges sensitive data, we refer to it as "Sensitive Data Exposure". This is often data directly linked to customers (e.g. names, dates-of-birth, financial information, etc), but could also be more technical information, such as usernames and passwords. At more complex levels this often involves techniques such as a "Man in The Middle Attack", whereby the attacker would force user connections through a device which they control, then take advantage of weak encryption on any transmitted data to gain access to the intercepted information (if the data is even encrypted in the first place...).

The most common way to store a large amount of data in a format that is easily accessible from many locations at once is in a database. This is obviously perfect for something like a web application, as there may be many users interacting with the website at any one time. Database engines usually follow the Structured Query Language (SQL) syntax

In a production environment it is common to see databases set up on dedicated servers, running a database service such as MySQL or MariaDB; however, ***databases can also be stored as files. These databases are referred to as "flat-file" databases, as they are stored as a single file on the computer.*** This is much easier than setting up a full database server, and so could potentially be seen in smaller web applications.

Flat-file databases are stored as a file on the disk of a computer. Usually this would not be a problem for a webapp, but what happens ***if the database is stored underneath the root directory of the website (i.e. one of the files that a user connecting to the website is able to access)? Well, we can download it and query it on our own machine, with full access to everything in the database.*** Sensitive Data Exposure indeed!

The most common (and simplest) format of flat-file database is an _sqlite_ database. These can be interacted with in most programming languages, and have a dedicated client for querying them on the command line. This client is called "_sqlite3_"


```sh
sqlite3 <database-name>
```

To see the tables

```sh
.tables
```

To find column names

```sh
PRAGMA table_info(TableName);
```

```sh
SELECT * FROM TableName;
```

