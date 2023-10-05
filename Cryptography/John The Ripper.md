
A hash is a way of taking a piece of data of any length and  representing it in another form that is a fixed length. This masks the original value of the data. This is done by running the original data through a hashing algorithm. There are many popular hashing algorithms, such as MD4,MD5, SHA1 and NTLM.

Hashing algorithms are designed so that they only operate one way. This means that a calculated hash cannot be reversed using just the output given. This ties back to a fundamental mathematical problem known as the P vs NP relationship . 

If you have the hashed version of a password, for example- and you know the hashing algorithm- you can use that hashing algorithm to hash a large number of words, called a dictionary. You can then compare these hashes to the one you're trying to crack, to see if any of them match. If they do, you now know what word corresponds to that hash- you've cracked it!

This process is called a **dictionary attack** and John the Ripper, or John as it's commonly shortened to, is a tool to allow you to conduct fast brute force attacks on a large array of different hash types.


**Wordlists**

As we explained in the first task, in order to dictionary attack hashes, you need a list of words that you can hash and compare, unsurprisingly this is called a wordlist


---

#### Cracking Basic Hashes

```sh
john [options] [pathToFile]
```

john - Invokes the John the Ripper program

path to file - The file containing the hash you're trying to crack, if it's in the same directory you won't need to name a path, just the file.


**Automatic Cracking**

John has built-in features to detect what type of hash it's being given, and to select appropriate rules and formats to crack it for you, this isn't always the best idea as it can be unreliable- but if you can't identify what hash type you're working with and just want to try cracking it, it can be a good option
To do this

```sh
john --wordlist=[pathToWordlist] [pathToFile]
```

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
```


**Format Specific Cracking**

```sh
john --format=[format] --wordlist=[path to wordlist] [path to file]
```


--format= - This is the flag to tell John that you're giving it a hash of a specific format, and to use the following format to crack it

```sh
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
```

When you are telling john to use formats, if you're dealing with a standard hash type, e.g. md5 as in the example above, you have to prefix it with `raw-` to tell john you're just dealing with a standard hash type, though this doesn't always apply. To check if you need to add the prefix or not, you can list all of John's formats using `john --list=formats` and either check manually, or grep for your hash type using something like `john --list=formats | grep -iF "md5"`.


---

#### Cracking Windows Authentication Hashes


Authentication hashes are the hashed versions of passwords that are stored by operating systems, it is sometimes possible to crack them using the brute-force methods that we're using. To get your hands on these hashes, you must often already be a privileged user

#### NTHash / NTLM

NThash is the hash format that modern Windows Operating System machines will store user and service passwords in. It's also commonly referred to as "NTLM" which references the previous version of Windows format for hashing passwords known as "LM", thus "NT/LM".

You can acquire NTHash/NTLM hashes by dumping the SAM database on a Windows machine, by using a tool like Mimikatz or from the Active Directory database: NTDS.dit. You may not have to crack the hash to continue privilege escalation- as you can often conduct a "pass the hash" attack instead, but sometimes hash cracking is a viable option if there is a weak password policy.

```sh
john --format=nt --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
```


---

#### Cracking /etc/shadow Hashes


The /etc/shadow file is the file on Linux machines where password hashes are stored. It also stores other information, such as the date of last password change and password expiration information. It contains one entry per line for each user or user account of the system. This file is usually only accessible by the root user- so in order to get your hands on the hashes you must have sufficient privileges, but if you do- there is a chance that you will be able to crack some of the hashes.

**Unshadowing**

John can be very particular about the formats it needs data in to be able to work with it, for this reason- in order to crack /etc/shadow passwords, you must combine it with the /etc/passwd file in order for John to understand the data it's being given. To do this, we use a tool built into the John suite of tools called unshadow.

```sh
unshadow [path to passwd] [path to shadow]
```


`unshadow` - Invokes the unshadow tool  

`[path to passwd]` - The file that contains the copy of the /etc/passwd file you've taken from the target machine  

`[path to shadow]` - The file that contains the copy of the /etc/shadow file you've taken from the target machine

```sh
unshadow local_passwd local_shadow > unshadowed.txt
```


**Cracking**

We're then able to feed the output from unshadow, in our example use case called "unshadowed.txt" directly into John. We should not need to specify a mode here as we have made the input specifically for John, however in some cases you will need to specify the format as we have done previously using: `--format=sha512crypt`

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt --format=sha512crypt unshadowed.txt
```


---

#### Single Crack Mode


John also has another mode, called Single Crack mode. In this mode, John uses only the information provided in the username, to try and work out possible passwords heuristically, by slightly changing the letters and numbers contained within the username.

**Word Mangling**

If we take the username: Markus

Some possible passwords could be:

- Markus1, Markus2, Markus3 (etc.)
- MArkus, MARkus, MARKus (etc.)
- Markus!, Markus$, Markus* (etc.)


This technique is called word mangling. John is building it's own dictionary based on the information that it has been fed and uses a set of rules called "mangling rules" which define how it can mutate the word it started with to generate a wordlist based off of relevant factors for the target you're trying to crack. This is exploiting how poor passwords can be based off of information about the username, or the service they're logging into.  

**GECOS**

John's implementation of word mangling also features compatibility with the Gecos fields of the UNIX operating system, and other UNIX-like operating systems such as Linux.
In /etc/passwd and /etc/shadow You can see that each field is seperated by a colon ":". Each one of the fields that these records are split into are called Gecos fields. John can take information stored in those records, such as full name and home directory name to add in to the wordlist it generates when cracking /etc/shadow hashes with single crack mode.

```sh
john --single --format=[format] [path to file]
```


```sh
john --single --format=raw-sha256 hashes.txt
```

If you're cracking hashes in single crack mode, you need to change the file format that you're feeding john for it to understand what data to create a wordlist from. You do this by prepending the hash with the username that the hash belongs to, so according to the above example- we would change the file hashes.txt

**From:**  
1efee03cdcb96d90ad48ccc7b8666033
**To**
mike:1efee03cdcb96d90ad48ccc7b8666033


---

#### Custom Rules

You can define your own sets of rules, which John will use to dynamically create passwords. This is especially useful when you know more information about the password structure of whatever your target is.

Many organisations will require a certain level of password complexity to try and combat dictionary attacks, meaning that if you create an account somewhere, go to create a password and enter:

polopassword

You may receive a prompt telling you that passwords have to contain at least one of the following:

- Capital letter
- Number
- Symbol

This is good! However, we can exploit the fact that most users will be predictable in the location of these symbols. For the above criteria, many users will use something like the following:

Polopassword1!


***Custom rules are defined in the john.conf file, usually located in /etc/john/john.conf***

The first line:

`[List.Rules:THMRules]` - Is used to define the name of your rule, this is what you will use to call your custom rule as a John argument.

We then use a regex style pattern match to define where in the word will be modified, again- we will only cover the basic and most common modifiers here:

`Az` - Takes the word and appends it with the characters you define  

`A0` - Takes the word and prepends it with the characters you define  

`c` - Capitalises the character positionally

Lastly, we then need to define what characters should be appended, prepended or otherwise included, we do this by adding character sets in square brackets `[ ]` in the order they should be used. These directly follow the modifier patterns inside of double quotes `" "`.


`[0-9]` - Will include numbers 0-9  

`[0]` - Will include only the number 0  

`[A-z]` - Will include both upper and lowercase  

`[A-Z]` - Will include only uppercase letters  

`[a-z]` - Will include only lowercase letters  

`[a]` - Will include only a  

`[!£$%@]` - Will include the symbols !£$%@


`[List.Rules:PoloPassword]`

`cAz"[0-9] [!£$%@]"`

In order to:

Capitalise the first  letter - `c`

Append to the end of the word - `Az`

A number in the range 0-9 - `[0-9]`

Followed by a symbol that is one of `[!£$%@]`

```sh
john --wordlist=[path to wordlist] --rule=PoloPassword [path to file]
```


---

#### Cracking Password Protected Zip Files

We can use John to crack the password on password protected Zip files. Again, we're going to be using a separate part of the john suite of tools to convert the zip file into a format that John will understand

**zip2john**

Similarly to the unshadow tool that we used previously, we're going to be using the zip2john tool to convert the zip file into a hash format that John is able to understand, and hopefully crack

```sh
zip2john [options] [zip file] > [output file]
```

`[options]` - Allows you to pass specific checksum options to zip2john, this shouldn't often be necessary  

`[zip file]` - The path to the zip file you wish to get the hash of

`>` - This is the output director, we're using this to send the output from this file to the...  

`[output file]` - This is the file that will store the output from

```sh
zip2john zipfile.zip > zip_hash.txt
```

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt zip_hash.txt
```


---

#### Cracking Password Protected RAR Archives

```sh
rar2john [rar file] > [output file]
```

```sh
rar2john rarfile.rar > rar_hash.txt
```

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt rar_hash.txt
```


---

#### Cracking SSH Keys

Using John to crack the SSH private key password of id_rsa files. Unless configured otherwise, you authenticate your SSH login using a password. However, you can configure key-based authentication, which lets you use your private key, id_rsa, as an authentication key to login to a remote machine over
ssh2john converts the id_rsa private key that you use to login to the SSH session into hash format that john can work with.

```sh
ssh2john [id_rsa private key file] > [output file]
```


`[id_rsa private key file]` - The path to the id_rsa file you wish to get the hash of

`>` - This is the output director, we're using this to send the output from this file to the...  

`[output file]` - This is the file that will store the output from


```sh
ssh2john id_rsa > id_rsa_hash.txt
```

```sh
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa_hash.txt
```






