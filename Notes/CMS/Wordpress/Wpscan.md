----
- Tags: #web #wordpress #reconnaissance #tools
----
Wpscan is commonly used to enumerate web sites defined by [[WordPress]] CMS.
Its use mode is very straightforward by doing in Bash:
```bash
wpscan --url <URL>
```
To enumarate possible valid user, you can use:
```bash
wpscan --url <URL> --enumerate u
```
You can use brute force to find out valid credentials into the authentication panels:
```bash
wpscan --url <URL> -U <user> -P <word list>
```
- This procedure is also performed manually by using a Bash code:
- 
## References
[^1] Wpscan GitHub project:: [wpscan](https://github.com/wpscanteam/wpscan)
[^1] Manual procedure abusing xmlrpc.php file [xmlrpc](https://github.com/wpscanteam/wpscan)