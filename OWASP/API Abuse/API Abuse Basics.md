# Definition
Something to do.
# Installing and Preparing Lab
I will use the [OWASP-crAPI](https://github.com/OWASP/crAPI) GitHub project to practice this vulnerability.
First, we must validate the version of *docker-compose*, it should be higher than **1.27.0**.
```shell
docker-compose version
```
![[Pasted image 20230822115826.png]]
___
We should update that version.
```shell
sudo apt remove docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.3/docker-compose-$(uname -s)-$(uname -m)" -o docker-compose
sudo chmod +x docker-compose
./docker-compose version
```
![[Pasted image 20230822120612.png]]
___
To use this *binary* of docker-compose version **2.20.3** in any place, we could move it to **/usr/local/bin/** (PATH).
```shell
sudo mv ./docker-compose /usr/local/bin/
```
___
Well, continuing we should clone the *machine* repository.
```shell
git clone https://github.com/OWASP/crAPI
cd crAPI/deploy/docker
docker-compose pull
```
![[Pasted image 20230822121330.png]]
___
After that, we have to deploy the **containers** of the project.
```shell
docker-compose -f docker-compose.yml --compatibility up -d
```
![[Pasted image 20230822122014.png]]
You can observe that **ERROR**.
___
Well, we should remove all the docker *images* and *containers* and then re-execute the commands.
```shell
docker rm $(docker ps -a -q) --force
docker rmi $(docker images -a -q)
docker volume rm $(docker volume ls -q)
docker network rm $(docker network ls -q)
VERSION=develop docker-compose pull
VERSION=develop docker-compose -f docker-compose.yml --compatibility up -d
```
![[Pasted image 20230822124418.png]]
![[Pasted image 20230822124437.png]]
___
# Contextualizing
Let's start, why don't we register?
![[Pasted image 20230822135124.png]]
___
Then, we could sign in.
![[Pasted image 20230822135804.png]]
Nothing so far.
___
Let's analize the **Login** request.
![[Pasted image 20230822140012.png]]
This is making a *request* to an authentication **API**.
___
So, the content of the *response* of that *request* is assinging a token.
![[Pasted image 20230822140258.png]]
You may notice, according to the `.` in the **token** structure, it is a **JWT**.
![[Pasted image 20230822140544.png]]
This **JWT** is generated to each logged user.
___
Now, in the **dashboard** field, it shows our information according to the **JWT** assigned.
![[Pasted image 20230822141011.png]]
![[Pasted image 20230822141102.png]]
___
# Hacking
## Enumerating With Postman
To sort the enumeration of the web environment, we could use [[Postman]] to improve our effectiveness when working with multiple requests and responses.
First, we have to create a new *Collection* called **crAPI**.
![[Pasted image 20230822144326.png]]
Then, we have to create a new **HTTP request** to the web *login* address and specifying the **request** data, the **POST** method and **JSON** as *Content: Type*.
![[Pasted image 20230822144716.png]]
![[Pasted image 20230822145139.png]]
___
Well, let's do the same with the **dashboard** field to see our information.
![[Pasted image 20230822145644.png]]
But, when we send that request in *postman* it doesn't turn out as we expected.
![[Pasted image 20230822145841.png]]
___
It happens because we are not sending the **JWT** in the request.
But the **JWT** is dynamic. So, we have to work with *variables* in *postman* and specify the token *type* and its *value*.
![[Pasted image 20230822150317.png]]
![[Pasted image 20230822150348.png]]
___
Let's continue by registering the **Shop** field *(We have to click on the SHOP button to see the request in the web)*.
![[Pasted image 20230822150646.png]]
![[Pasted image 20230822150906.png]]
___
Well, let's enumerate when we *buy* a product.
![[Pasted image 20230822161310.png]]
![[Pasted image 20230822161501.png]]
___
## Attacking User Information
Doing the same proccess as above, let's enumerate the *change email* button to try if we can change the **mail** of the current user.
![[Pasted image 20230822161947.png]]
___
To change the *email* we must provide de **Token** sent to our *email*.
The *email* manager is deployed on port **8025**.
![[Pasted image 20230822162157.png]]
![[Pasted image 20230822163151.png]]
___
Then, you may notice we can't do a *brute force* to crack the **Token**.
![[Pasted image 20230822163324.png]]
___
Well, why don't we try out the **forgot password** feature.
![[Pasted image 20230822163444.png]]
![[Pasted image 20230822163510.png]]
___
It is sending a **OPT** token to your *email* to change your password.
![[Pasted image 20230822163651.png]]
___
Well, and the **OPT** value has *four* digists.
![[Pasted image 20230822163746.png]]
___
Can we do a *brute force* to crack that value?
Let's try it using [[ffuz]].
First, how this application is being processed?
![[Pasted image 20230822164215.png]]
![[Pasted image 20230822164316.png]]
We are going to enumerate that in **postman**.
![[Pasted image 20230822164532.png]]
___
Continuing, we use [[ffuz]].
```shell
ffuf -u http://localhost:8888/identity/api/auth/v3/check-otp -w /usr/share/SecLists/Fuzzing/4-digits-0000-9999.txt -X POST -d '{"email":"cxnsxle@cxnsxle.com","otp":"FUZZ","password":"newPassword#1"}' -H 'Content-Type: application/json' -p 1
```
- `-u http://localhost:8888/identity/api/auth/v3/check-otp`: **API** request URL.
- `-w /usr/share/SecLists/Fuzzing/4-digits-0000-9999.txt`: *Wordlist* of **SecLists** with all combinations of **4** digits numbers.
- `-X POST`: **POST** method.
- `-d`: **POST** data with *FUZZ* word to replace it by the wordlist.
- `-H 'Content-Type: application/json'`: Headers needed.
- `-p 1`: Delay of **1** second to avoid overloads.
___
After the **2936** attemp, we can notice in **postman** that the **API** block us due to a lot of *request*.
![[Pasted image 20230822165952.png]]
![[Pasted image 20230822170013.png]]
___
And, what do we do now?
When we were using **postman**, in the **dashboard** field, enumarated earlier, the **HTTP** request was sent to the **V2** of the **API**.
![[Pasted image 20230822211106.png]]
Well, why don't we use that *version* on our *change password* **HTTP** request?
![[Pasted image 20230822211247.png]]
___
Now that we can to do brute force again, let's continue with the version **V2** of the **API**.
```shell
ffuf -u http://localhost:8888/identity/api/auth/v2/check-otp -w /usr/share/SecLists/Fuzzing/4-digits-0000-9999.txt -X POST -d '{"email":"cxnsxle@cxnsxle.com","otp":"FUZZ","password":"newPassword#1"}' -H 'Content-Type: application/json' -p 1 -mc 200
```
- `-mc 200`: Only shows reponses with status code **200**.
___
But we are going to generate another **OTP**, since the previous one has already expired.
![[Pasted image 20230822213314.png]]
___
Then, let's crack it.
![[Pasted image 20230822213438.png]]
Good job, now the new *password* of that users should be **newPassword#1**. Let's use it.
![[Pasted image 20230822213625.png]]
___
## Attacking the Money
We go back to the **Shop** field.
### Mass Assignment Attack
You can remember we buy a *Seat* twice. Therefore, we should have **$80**.
![[Pasted image 20230822214138.png]]
___
When you encounter these cases, you should try out to change the **HTTP** request. For instance, change **GET** by **POST** and analize the response.
But, the web does not always allow **any** method. How do we know which methods are allowed?
We can try to use another *brute force* with [[ffuf]].
```shell
ffuf -u http://localhost:8888/workshop/api/shop/products -w /usr/share/SecLists/Fuzzing/http-request-methods.txt -X FUZZ -p 1
```
- `-X FUZZ`: Replace **FUZZ** by each word in the *wordlist*.
![[Pasted image 20230822214808.png]]
There are a lot of respones with status code **405**.
![[Pasted image 20230822215038.png]]
___
Let's avoid them.
```shell
ffuf -u http://localhost:8888/workshop/api/shop/products -w /usr/share/SecLists/Fuzzing/http-request-methods.txt -X FUZZ -p 1 -mc 401,200
```
- `-mc 401,200`: Only shows responses with status code **401** and **200**.
![[Pasted image 20230822215205.png]]
You may notice there are **four** allowed methods.
___
Another way to find the allowed methods is sending a **OPTIONS** http request and analizing the response *headers*.
![[Pasted image 20230822215442.png]]
___
First, we are going to probe the **POST** method.
Curiously, this method asks for three parameters **name**, **price** and **image_url**. Is it to create a new product?
![[Pasted image 20230822215831.png]]
___
We are going to try to create a new product, *but with a negative price*.
Curiously, the response was **OK**.
![[Pasted image 20230822220242.png]]
![[Pasted image 20230822220321.png]]
___
So, what happens if I click the **Buy** button?
![[Pasted image 20230822220429.png]]
My money has been incremented. Nice.
___
### Attacking the Coupon Field
Another way to attack this web is in the **Add Coupons** field.
Let's get analizing the **request** behind of that button and enumare it with **postman**.
![[Pasted image 20230822221049.png]]
![[Pasted image 20230822221106.png]]
![[Pasted image 20230822221344.png]]
In this case as the coupon *value* is invalid, it does not show us nothing.
___
As this web uses a **json** format to send data and also in this case of validating coupon, you could think the web uses a databse to verify the coupon *value*.
Let's try a [[NoSQLI Basics]] to see if the web responds with anything.
This payloads says *the coupon code does not equal to 123*.
```json
{
	"coupon_code":{
		"$ne":"123"
	}
}
```
![[Pasted image 20230822222009.png]]
![[Pasted image 20230822222153.png]]
Nice job, we were able to bypass the *coupon* validation using a [[NoSQLI Basics]].
___