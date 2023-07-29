# Definition
Something to do!!
# Installing and Preparing lab
I will use my own machine to practice this vulnerability.
```bash
cd /var/www/html
sudo service apache2 start
```
___
Then, we are going to create the main **index.php** at **/var/www/html/**.
This is simply a *PHP* *Form* to validate **admin** credentials, with a remarkable *strcmp* function that expects to compare strings.
```php
<html>
	<font color="red"><center><h1>Secure Login Page</h1></center></font>
	<hr>
	<body style="background-color:powderblue;">
		<center><form method="POST" name="<?php basename($_SERVER['PHP_SELF']); ?>">
					Username: <input type="text" name="username" id="username" size="30">
					&nbsp;
					Password: <input type="password" name="password" id="password" size="30">
					<input type="submit" value="Login">
					<hr>
				</form>
		</center>

		<?php 
			$USER = "admin";
			$PASSWORD = "adm1n!$!@@#!adminS_!@#";

			// Validate empty fields
			if (isset($_POST['username']) && isset($_POST['password'])) {
				// Validate username
				if ($_POST['username'] == $USER) {
					// Validate password --> (Type Juggling vulnerable)
					if (strcmp($_POST['password'], $PASSWORD) == 0) {
						echo "[+] Welcome: admin";
					} else {
						echo "[!] Password incorrect";
					}
				} else {
					echo "[!] Username incorrect";
				}
			}
		?>
	</body>
</html>
```
___
# Attacking
## Contextualizing
Let's use **BurpSuite** to find out what is happening behind the POST request.
![[Pasted image 20230729164354.png]]
___
So what is the problem when we use **strcmp** to compare strings?
This happens when we send an **ARRAY** which validates it by skipping validation.
![[Pasted image 20230729164738.png]]
___
## Type Juggling in `==` Comparation 
Another way to bypass password validation is when we use `==` comparation to validate passwords.
To explain this case let's modify our **index.php** at **/var/www/html/**.
This code uses a **MD5** validation to verify the correct password.
```php
<html>
	<font color="red"><center><h1>Secure Login Page</h1></center></font>
	<hr>
	<body style="background-color:powderblue;">
		<center><form method="POST" name="<?php basename($_SERVER['PHP_SELF']); ?>">
					Username: <input type="text" name="username" id="username" size="30">
					&nbsp;
					Password: <input type="password" name="password" id="password" size="30">
					<input type="submit" value="Login">
					<hr>
				</form>
		</center>

		<?php 
			$USER = "admin";
			// password hash
			$PASSWORD = "0e8961261230981231269013";

			// Validate empty fields
			if (!empty($_POST['username']) && !empty($_POST['password'])) {
				// Apply a md5 to password input
				$password_input = md5($_POST['password']);
				// Validate username and password (Type Juggling vulnerable)
				if ($_POST['username'] == $USER && $password_input == $PASSWORD) {
					echo "[+] Welcome: admin";
				} else {
					echo "[!] Username or password incorrect";
				}
			}
		?>
	</body>
</html>
```
___
So, the problem with `==` validation is that it resolves the hash as a *mathematical operation*.
For example, if the hash is `0e12123456789` then the `==` comparison will resolve it as `0^12123456789=0`.
![[Pasted image 20230729171103.png]]
___
So, how do we bypass the **MD5** password validation?
First of all, we must find some value that if after being applied a **MD5** will give us **0e...**.
Making a web search, we did find this [website](https://www.hackplayers.com/2018/03/hashes-magicos-en-php-type-jugling.html) that shows us some *magic* hashes.
Now, using this string **aabg7XSs**, let's try out in the form.
![[Pasted image 20230729171858.png]]
___