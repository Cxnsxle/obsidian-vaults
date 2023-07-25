# Definition
Something to do!!
## Installing and Preparing lab
I will be using [PENTESTER LAB: PADDING ORACLE](https://www.vulnhub.com/entry/pentester-lab-padding-oracle,174/) VulnHub machine to practice this vulnerability.
### Downlad ISO Image
First, we have to download the ISO image from VulnHub source.
### Install and Configure Machine
Second, we are going to install this machine in **VMware**.
![[Pasted image 20230720183255.png]]
___
After that, we have to follow the secuence: **Next -> Next -> (Change name and path machine) Next -> (Select single file) Next -> Customize Hardware**.
Then, we must add this machine to our *Network*.
I have this network on VMware and my *Attacker* machine uses that.
![[Pasted image 20230720184103.png]]
![[Pasted image 20230720184233.png]]
___
Then, **Close -> Finish**.
Now you can start the machine and from *Attacker* machine you can see that.
```shell
sudo arp-scan -I ens33 --localnet --ignoredups
```
![[Pasted image 20230720184653.png]]
___
From web browser.
![[Pasted image 20230720184726.png]]
___
# Attacking
## Understanding
Let's start to scan the machine by using *Nmap*.
```shell
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.200.132
```
- `-p-`: All ports (65535).
- `--open`: Show open ports.
- `-sS`: Half-open scanning technique.
- `--min-rate 5000`: min rate of sent packets is 5000.
- `-vvv`: Triple verbose.
- `-n`: No DNS resolution.
- `-Pn`: No Host discovery, the IP sent is taken as valid or existing.
- `192.168.200.132`: Target's IP.
![[Pasted image 20230720185047.png]]
___
## Contextualizing
First of all, we need to know how *Padding* works in the **CBC** cipher. Therefore, you should read the explanation made by [PentesterLab](https://pentesterlab.com/exercises/padding_oracle/course).
So, this encryption technique uses the XOR operation. Great, with this specific operation you can swap the order of the operands as explained in the following image.
![[Pasted image 20230725163038.png]]
___
After that, to get a better understanding of **CBC** cipher, also you might see this simple explanation from [Wikipedia](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation).
![[Pasted image 20230725162620.png]]
___
Then, the padding *PSKC7* does the padding according to the number of spaces to complete its block.
For instance, if we have *two* spaces to be completed, we must complete those espaces with **\x02**. In another way, if we have *five* spaces to be completed, we should use **\x05**.
![[Pasted image 20230725170616.png]]
___
Good, now you can see that we have **TWO** encrypted blocks and we want to descipher the *second* block.
![[Pasted image 20230725163445.png]]
To achieve it you should know that the server, in the *padding* context, does the following.
![[Pasted image 20230725165714.png]]
Thus, you can take advantage of that *padding validation* to try to obtain the **PLAIN TEXT** of each block.
___
So, how do we do that?
Leveraging the *XOR* property to swap operands. You could variate the *last block piece* of the **encrypted block 1**. And why would we do that?
As we can *swap* operands, the server will respond us the same when we send the following.
![[Pasted image 20230725172926.png]]
Thus, we could variate **E2** with some *padding* value to know the **PLAIN TEXT** value of *C4*.
___
**THIS IS A PRACTICAL EXPLANATION OF THIS METHOD, NOW YOU CAN GET A MUCH BETTER UNDERSTANDING OF PentesterLab'S DETAILED EXPLANATION**.
___
## Attacking
Now that we know how *CBC* encryption and *padding* work. Let's start by registering.
![[Pasted image 20230725173908.png]]
___
![[Pasted image 20230725173934.png]]
___
Now you can see that your *cookie session* has been encrypted and this lab is using *CBC padding* technique.
![[Pasted image 20230725174252.png]]
___
### Attack Using padbuster
So, with this *CBC encrypted* string we could use **padbuster** tool, that automatize us, to find out the **PLAIN TEXT** of our *cookie*.
```shell
padbuster http://192.168.200.132/index.php 6MB1N19F5oTSMPS5TCZWYHDKkZ8jj57b 8 -cookies 'auth=6MB1N19F5oTSMPS5TCZWYHDKkZ8jj57b'
```
- `http://192.168.200.132/index.php`: web link.
- `6MB1N19F5oTSMPS5TCZWYHDKkZ8jj57b`: CBC encrypted string.
- `8`: block size, it should be *multiple of 8*
- `-cookies 'auth=6MB1N19F5oTSMPS5TCZWYHDKkZ8jj57b'`: specify that the web is encrypting the *auth* cookie.
![[Pasted image 20230725175104.png]]
We can see that the *CBC encrypted* string desciphered is **user=cxnsxle**.
___
Now, let's try *encryp* the **user=admin** plain text to see whether we can be *admin*.
```shell
adbuster http://192.168.200.132/index.php 6MB1N19F5oTSMPS5TCZWYHDKkZ8jj57b 8 -cookies 'auth=6MB1N19F5oTSMPS5TCZWYHDKkZ8jj57b' -plaintext 'user=admin'
```
- `-plaintext 'user=admin'`: To generate *CBC encrypted* string of *user=admin*.
![[Pasted image 20230725180045.png]]
___
Then, we could use this string in the *cookie auth* field to become *admin*.
![[Pasted image 20230725180230.png]]
___
### Attack Using Bit Flipper
Another way to get the *CBC encrypted* string of *admin* cookie is having the *CBC encrypted* string of another user similar to *admin* like **cdmin**.
Then, now that we know that *CBC encryption* uses **bytes**, we could use a **Bit Flipper** to variate a few bits until find out the *admin CBC encrypted* string.
We will use **Bit Flipper** tool of *BurpSuite* to achieve that. 
First, we are going to register the *cdmin* account and then obtain his *CBC encrypted* string cookie.
![[Pasted image 20230725181350.png]]
His cookie is **aiEaTxWaYZkygmUg%2BR15FZU1DKg4PWsc**.
___
After that, we will use BurpSuite to do a *Bit Flipper* attack in the *Intruder* field.
Then, intercept the web and go to the Intruder field by pressing *CTRL+i*. You should see something like this.
![[Pasted image 20230725181858.png]]
___
After a while the attack started, we were able to obtain a *CBC encrypted* string of *admin* cookie.
![[Pasted image 20230725182209.png]]
The string is **aiEaTxUaYZkygmUg%2BR15FZU1DKg4PWsc**.
___
Finally we can use this *string* to become **admin** again.
![[Pasted image 20230725182402.png]]
___