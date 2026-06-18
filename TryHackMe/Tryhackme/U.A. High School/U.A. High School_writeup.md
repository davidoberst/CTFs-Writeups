First i will do a nmap scan to target 

[davidoberst@archlinux ~]$ sudo nmap -Pn -sV -O 10.65.145.2

we have : 
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

i open the webpage 
http://10.65.184.34/ 
im gonna do fuzzing
[davidoberst@archlinux Web-Content]$ gobuster dir -u http://10.65.184.34/ -w common.txt



it shows the webiste of U.A High school, i found the about us or contact me page,which contains a form, whit name, email, subjetct, message, i will intercept the requstr wohit butrpsuite.

