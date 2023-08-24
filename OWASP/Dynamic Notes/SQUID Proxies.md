# SQUID Proxies
- First *Nmap* scan `sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn <IP> -oG allPorts`.
- Second *Nmap* scan `sudo nmap -sCV -p22,3128 <IP> -oN target`.
- Is the port **80** filtered in the nmap scan? `nmap -p80 <IP> -T5 -v -n -Pn` **open** or **closed**?.
- There is a **proxy**, in the victim server, to filter the access to the web? Scan the machine through the **proxy** found.
	```shell
	curl http://<IP> --proxy http://<IP>:<PROXY>
	```
- Brute force *directory* scan.
	```shell
	gobuster dir -u http://<IP> --proxy http://<IP>:<PROXY> -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20
	```
- Use this automate **Python** script.
	```python
	#!/usr/bin/python3
	import sys
	import signal
	import requests
	
	# function to manage CTRL+c
	def def_handler(sig, frame):
	    print("\n[!] Exiting...\n")
	    sys.exti(1)
	
	# function to find out every port behind a Proxy
	def portDiscovery():
	    common_tcp_ports = {20, 21, 22, 23, 25, 53, 80, 110, 115, 119, 135, 143, 161, 194, 443, 445, 465, 514, 993, 995, 1433, 1521, 1723, 3306, 3389, 5060, 5222, 5432, 5900, 6379, 6666, 8080, 8443, 9090, 9100, 9933, 10000, 12345, 14334, 16000, 16992, 20000, 21025, 22222, 27017, 30718, 32764, 32887, 49152, 49153, 50000}
	
	    for port in common_tcp_ports:
	        r = requests.get(main_url + ':' + str(port), proxies=squid_proxy)
	        if (r.status_code != 503):
	            print("Port:", port, "opened")
	
	# CTRL+c
	signal.signal(signal.SIGINT, def_handler)
	
	# global variables
	main_url = "http://127.0.0.1"
	squid_proxy = {"http": "http://192.168.200.134:3128"}
	
	if __name__ == "__main__":
	    portDiscovery();
	```
- Try to change the main **IP** to **127.0.0.1** using that internal proxy.