# JWT
- Use [JWT-reverse](https://jwt.io/).
- Probe a custom **JWT** -> `<HEADER>.<PAYLOAD>.` when the **signature** is not required.
	```shell
	 # HEADER FIELD
	echo -n '{"alg":"NONE","typ":"JWT"}' | base64 
	 # PAYLOAD FIELD
	echo -n '{"id":2,"iat":1693012735,"exp":1693016335}' | base64
	```
- Probe *common* words in the **signature** field like **secret** (your-256-bit-secret).