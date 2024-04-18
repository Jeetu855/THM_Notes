
### du(disk usage)

du is a command in linux (short for disk usage) which helps you identify what files/directories are consuming how much space
**The size here is shown in KB**

Important flags

|   |   |
|---|---|
|Flag|Description|
|-a|Will list files as well with the folder.|
|-h|Will list the file sizes in human readable format(B,MB,KB,GB)|
|-c|Using this flag will print the total size at the end. Jic you want to find the size of directory you were enumerating|
|-d <number>|Flag to specify the depth-ness of a directory you want to view the results for (eg. -d 2)|
|--time|To get the results with time stamp of last modified.|

```sh
du -a /home/
```

will list every file in the /home/ directory with their sizes in KB

---

### grep

The grep filter searches a file for a particular pattern of characters, and displays all lines that contain that pattern. The pattern that is searched in the file is referred to as the regular expression

Syntax: `grep "PATTERN" file.txt` will search the file.txt for the specified "PATTERN" string, if the string is found in the line, the grep will return the whole line containing the "PATTERN" string.


egrep and fgrep are no different from grep(other than 2 flags that can be used with grep to function as both). In simple words, egrep matches the regular expressions in a string, and fgrep searches for a fixed string inside text. Now grep can do both their jobs by using -E and -F flag, respectively.

In other terms, `grep -E` functions same as `egrep` and `grep -F` functions same as `fgrep`.

Important Flags

|   |   |
|---|---|
|Flags|Description|
|-R|Does a recursive grep search for the files inside the folders(if found in the specified path for pattern search; else grep won't traverse diretory for searching the pattern you specify)|
|-h|If you're grepping recursively in a directory, this flag disables the prefixing of filenames in the results.|
|-c|This flag won't list you the pattern only list an integer value, that how many times the pattern was found in the file/folder.|
|-i|I prefer to use this flag most of the time, this is what specifies grep to search for the PATTERN while IGNORING the case|
|-l|will only list the filename instead of pattern found in it.|
|-n|It will list the lines with their line number in the file containing the pattern.|
|-v|This flag prints all the lines that are NOT containing the pattern|
|-E|This flag we already read above... will consider the PATTERN as a regular expression to find the matching strings.|
|-e|The official documentation says, it can be used to specify multiple patterns and if any string matches with the pattern(s) it will list it.|

-e flag can be used to specify multiple patterns, with multiple use of -e flag( grep -e PATTERN1 -e PATTERN2 -e PATTERN3 file.txt), whereas, -E can be used to specify one single pattern(You can't use -E multiple times within a single grep statement).

Other point that you can use to understand the difference is, -e works on the BREs(Basic Regular Expressions) and -E works on EREs (Extended Regular Expressions).

- BREs tend to match a single pattern in a file (Simplest examples can be direct words like "sun", "comic")
- EREs tend to match 2 or more patterns in a file (To select a no of words like (sun sunyon sandston) the pattern could be "^s.*n$").


### STROPS(STRing OPerationS)

##### tr

Translate command(`tr`) can help you in number of ways, ranging from changing character cases in a string to replacing characters in a string.

```sh
tr [flags] [source]/[find]/[select] [destination]/[replace]/[change]
```

|   |   |
|---|---|
|Flags|Description|
|-d|To delete a given set of characters|
|-t|To concat source set with destination set(destination set comes first; t stands for truncate)|
|-s|To replace the source set with the destination set(s stands for squeeze)|
|-c|This is the REVERSE card in this game, for eg. If you specify -c with -d to delete a set of characters then it will delete the rest of the characters leaving the source set which we specified (c stands for complement; as in doing reverse of something)|

You must have noticed the word "set" while reading the flags. Well that's true... tr command works in sets of character.

If you want to convert every alphabetic character to upper case.

```sh
cat file.txt | tr -s '[a-z]' '[A-Z]'
```

OR
```sh
cat file.txt | tr -s '[:lower:]' '[:upper:]
```


##### awk

**Awk is a scripting language used for manipulating data and generating reports.The awk command programming language requires no compiling, and allows the user to use variables, numeric functions, string functions, and logical operators.**

```sh
awk [flags] [select pattern/find(sort)/commands] [input file]
```


If the commands you wrote are in a script you can execute the script commands by using the `-f` flag and specifying the name of the script file. 

```sh
awk -f script.awk input.txt
```

To simply print a file with awk.

```sh
awk '{print}' file.txt
```

To search for a pattern inside a file you enclose the pattern in forward slashes `/pattern/` . For instance, if I want to know who all plays CTF competitions the command should be like: 

```sh
awk '/ctf/' file.txt
```

###### Built-In variables in AWK

Built-in variables include field variables ($1, $2, $3 .. $n). These field variables are used to specify a piece of data (data separated by a delimeter defaulting to space). If I run `awk '{print $1 $3}' file.txt` it will list me the words that are at 1st and 3rd fields.

```sh
awk '{print $1 $3}' file.txt
```

You can see, it joined the words together because we didn't specify the output delimeter

```sh
awk '{print $1,$3}' file.txt
```

***The \$0 variable points to the whole line.  Also, make sure to use single quotes('') to specify patterns, awk treats double quotes("") as a raw string. To use double quotes make sure that you escape the ($) sign(s) with a backslash (\) each, to make it work properly.***

****NR:** (Number Record) is the variable that keeps count of the rows after each line's execution... You can use NR command to number the lines (`awk '{print NR,$0}' file.txt`). Note that awk considers rows as records.**

```sh
awk '{print NR,$0}' file.txt
```

**FS:** (Field Separator) is the variable to set in case you want to define the field for input stream. The field separation (defaut to space) that we talked above and can be altered to whatever you want while specifying the pattern. FS can be defined to another character(s)(yea, can be plural) at the BEGIN{command}.

```sh
awk "BEGIN {FS='o'} {print $1,$3} END{print 'Total Rows=',NR}
```

**RS**: (Record Separator): By default it separate rows with '\n', you can specify something else too.

```sh
awk 'BEGIN{RS="o"} {print $0}' file.txt
```

The record separator in this case is alphabet 'o'

OFS: (Output Field Separator): It is to specify a delimeter while outputing... 

```sh
awk 'BEGIN{OFS=":"} {print $0}' file.txt
```

```sh
awk 'BEGIN{OFS=":"} {print $1,$3}' file.txt
```

I used OFS in both the commands, you can see that only in 2nd one the delimiter was used. Note that the output field separator will separate fields using (:) only when the fields are defined with the print statement. With $0 I didn't had anything else, if it were to be $0,$0 then the lines would be joining their reflection(non-laterally) with a colon(:).

**ORS:** (Output Record Separator)

```sh
awk 'BEGIN{ORS="\n\n"} {print $0}' file.txt
```

Important Flags

|   |   |
|---|---|
|Flags|Description|
|-F|With this flag you can specify FIELD SEPARATOR (FS), and thus don't need to use the BEGIN rule|
|-v|Can be used to specify variables(like we did in BEGIN{OFS=":"}|
|-D|You can debug your .awk scripts specifying this flag(`awk -D script.awk`)|
|-o|To specify the output file (if no name is given after the flag, the output is defaulted to awkprof.out)|


##### sed(Stream EDitor)

sed(Stream EDitor) is a tool that can perform a number of string operations. Listing a few, could be: FIND AND REPLACE, searching, insertion, deletion.

```sh
sed [flags] [pattern/script] [input file]
```

Important Flags

|   |   |
|---|---|
|Flags|Description|
|-e|To add a script/command that needs to be executed with the pattern/script(on searching for pattern)|
|-f|Specify the file containing string pattern|
|-E|Use extended regular expressions|
|-n|Suppress the automatic printing or pattern spacing|


Modes/Commands  

|   |   |
|---|---|
|Commands|Description|
|s|(Most used)Substitute mode (find and replace mode)|
|y|Works same as substitution; the only difference is, it works on individual bytes in the string provided(this mode takes no arguments/conditions)|

Args

|   |   |
|---|---|
|Flags/Args|Description|
|/g|globally(any pattern change will be affected globally, i.e. throughout the text; generally works with s mode)|
|/i|To make the pattern search case-insensitive(can be combined with other flags)|
|/d|To delete the pattern found(Deletes the whole line; takes no parameter like conditions/modes/to-be-replaced string)|
|/p|prints the matching pattern(a duplicate will occur in output if not suppressed with -n flag.)|
|/1,/2,/3../n|To perform an operation on an nth occurrence in a line(works with s mode)|

### xargs 

xargs, a very simple command to use when it comes to make passed string a command's argument, technically, positional argument. The official documentation says, xargs is a command line tool used to build and execute command from the standard input.

Important flags  

|   |   |
|---|---|
|Flags|Description|
|-0|Will terminate the arguments with null character (helps to handle spaces in the argument)|
|-a file|This option allows xargs to read item from a file|
|-d delimiter|To specify the delimiter to be used when differentiating arguments in stdin|
|-L int|Specifies max number non-blank inputs per command line|
|-s int|Consider this as a buffer size that you allocate while running xargs, it sets the max-chars for the command, which includes it's initial arguments and terminating nulls as well.(You won't be using this most of the times but it's good to know). Default size is around 128kB (if not specified).|
|-x|This flag will exit the command execution if the size specified is exceeded.(For security purposes.)|
|-E str|This is to specify the end-of-file string (You can use this in case you are reading arguments from a file)|
|-I str|(Capital i) Used to replace str occurrence in arguments with the one passed via stdin(More like creating a variable to use later)|
|-p|prompt the user before running any command as a token of confirmation.|
|-r|If the standard input is blank (i.e. no arguments passed) then it won't run the command.|
|-n int|This specifies the limit of max-args to be taken from command input at once. After the max-args limit is reached, it will pass the rest arguments into a new command line with the same flags issued to the previously ran command. (More like a looping)|
|-t|verbose; (Print the command before running it).Note: This won't ask for a prompt|

### xargs 

xargs, a very simple command to use when it comes to make passed string a command's argument, technically, positional argument. The official documentation says, xargs is a command line tool used to build and execute command from the standard input.

Important flags  

|   |   |
|---|---|
|Flags|Description|
|-0|Will terminate the arguments with null character (helps to handle spaces in the argument)|
|-a file|This option allows xargs to read item from a file|
|-d delimiter|To specify the delimiter to be used when differentiating arguments in stdin|
|-L int|Specifies max number non-blank inputs per command line|
|-s int|Consider this as a buffer size that you allocate while running xargs, it sets the max-chars for the command, which includes it's initial arguments and terminating nulls as well.(You won't be using this most of the times but it's good to know). Default size is around 128kB (if not specified).|
|-x|This flag will exit the command execution if the size specified is exceeded.(For security purposes.)|
|-E str|This is to specify the end-of-file string (You can use this in case you are reading arguments from a file)|
|-I str|(Capital i) Used to replace str occurrence in arguments with the one passed via stdin(More like creating a variable to use later)|
|-p|prompt the user before running any command as a token of confirmation.|
|-r|If the standard input is blank (i.e. no arguments passed) then it won't run the command.|
|-n int|This specifies the limit of max-args to be taken from command input at once. After the max-args limit is reached, it will pass the rest arguments into a new command line with the same flags issued to the previously ran command. (More like a looping)|
|-t|verbose; (Print the command before running it).Note: This won't ask for a prompt|

### xargs 

xargs, a very simple command to use when it comes to make passed string a command's argument, technically, positional argument. The official documentation says, xargs is a command line tool used to build and execute command from the standard input.

Important flags  

|   |   |
|---|---|
|Flags|Description|
|-0|Will terminate the arguments with null character (helps to handle spaces in the argument)|
|-a file|This option allows xargs to read item from a file|
|-d delimiter|To specify the delimiter to be used when differentiating arguments in stdin|
|-L int|Specifies max number non-blank inputs per command line|
|-s int|Consider this as a buffer size that you allocate while running xargs, it sets the max-chars for the command, which includes it's initial arguments and terminating nulls as well.(You won't be using this most of the times but it's good to know). Default size is around 128kB (if not specified).|
|-x|This flag will exit the command execution if the size specified is exceeded.(For security purposes.)|
|-E str|This is to specify the end-of-file string (You can use this in case you are reading arguments from a file)|
|-I str|(Capital i) Used to replace str occurrence in arguments with the one passed via stdin(More like creating a variable to use later)|
|-p|prompt the user before running any command as a token of confirmation.|
|-r|If the standard input is blank (i.e. no arguments passed) then it won't run the command.|
|-n int|This specifies the limit of max-args to be taken from command input at once. After the max-args limit is reached, it will pass the rest arguments into a new command line with the same flags issued to the previously ran command. (More like a looping)|
|-t|verbose; (Print the command before running it).Note: This won't ask for a prompt|

