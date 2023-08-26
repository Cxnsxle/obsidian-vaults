# SQL Truncation
When there is **input** fields size are limited. For example, just **13** characters.
- Use an existing **mail** or **username**, so when you register that is already exists.
	```txt
	(13)                                 (28)
	 mail@mail.com --------> mail@mail.com       anything
	```
	The database will truncate all the *after* the limit and then replace the original password.