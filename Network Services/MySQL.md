

MySQL is a relational database management system (RDBMS) based on Structured Query Language (SQL).
A database is simply a persistent, organised collection of structured data

**RDBMS:**

A software or service used to create and manage databases based on a relational model. The word "relational" just means that the data stored in the dataset is organised as tables. Every table relates in some way to each other's "primary key" or other "key" factors.

**SQL:**

MYSQL is just a brand name for one of the most popular RDBMS software implementations. As we know, it uses a client-server model. But how do the client and server communicate? They use a language, specifically the Structured Query Language (SQL).

Many other products, such as PostgreSQL and Microsoft SQL server, have the word SQL in them. This similarly signifies that this is a product utilising the Structured Query Language syntax.

MySQL, as an RDBMS, is made up of the server and utility programs that help in the administration of MySQL databases.

The server handles all database instructions like creating, editing, and accessing data. It takes and manages these requests and communicates using the MySQL protocol. This whole process can be broken down into these stages:  

1. MySQL creates a database for storing and manipulating data, defining the relationship of each table.
2. Clients make requests by making specific statements in SQL.
3. The server will respond to the client with whatever information has been requested.  
    

***What runs MySQL?***

MySQL can run on various platforms, whether it's Linux or windows. It is commonly used as a back end database for many prominent websites and forms an essential component of the LAMP stack, which includes: Linux, Apache, MySQL, and PHP.


#### Enumerating MySQL

Port Scanning 

Version FIngerprinting

Default Port of MySQL = 3306

```sh
mysql -h [IP] -u [username] -p
```

 `-p`: This option tells MySQL to prompt you for the password after you press Enter.

 `-h [IP]`: This option specifies the hostname (or IP address) of the MySQL server you want to connect to. You should replace `[IP]` with the actual IP address of the MySQL server you want to connect to.
 `-u [username]`: This option specifies the MySQL username you want to use when connecting to the server. You should replace `[username]` with the actual username you want to use.


Metasploit Module

***auxiliary/admin/mysql/mysql_sql***


#### Exploiting MySQL

1. MySQL server credentials  

2. The version of MySQL running

3. The number of Databases, and their names

**Schema:**

In MySQL, physically, a _schema_ is synonymous with a _database_. You can substitute the keyword "SCHEMA" instead of DATABASE in MySQL SQL syntax, for example using CREATE SCHEMA instead of CREATE DATABASE. It's important to understand this relationship because some other database products draw a distinction. For example, in the Oracle Database product,


**Hashes:**  

Hashes are, very simply, the product of a cryptographic algorithm to turn a variable length input into a fixed length output.

In MySQL hashes can be used in different ways, for instance to index data into a hash table. Each hash has a unique ID that serves as a pointer to the original data. This creates an index that is significantly smaller than the original data, allowing the values to be searched and accessed more efficiently


Metasploit module

***auxiliary/admin/mysql/mysql_sql***

***auxiliary/scanner/mysql/mysql_schemadump***

***scanner/mysql/mysql_hashdump***

