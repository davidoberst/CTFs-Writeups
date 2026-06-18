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

no other items founded 
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.64.132.198
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htpasswd            (Status: 403) [Size: 278]
.htaccess            (Status: 403) [Size: 278]
assets               (Status: 301) [Size: 315] [--> http://10.64.132.198/assets/]
server-status        (Status: 403) [Size: 278]
Progress: 20481 / 20481 (100.00%)
===============================================================
Finished
===============================================================



it shows the webiste of U.A High school, i found the about us or contact me page,which contains a form, whit name, email, subjetct, message, i will intercept the requstr wohit butrpsuite.

when i intercept the form, i recieve this :
POST /contact.html HTTP/1.1
Host: 10.64.132.198
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 67
Origin: http://10.64.132.198
Connection: keep-alive
Referer: http://10.64.132.198/contact.html
Cookie: PHPSESSID=f0v3sh7rpase8am97er21s47t7
Upgrade-Insecure-Requests: 1
Priority: u=0, i

name=Juan&email=juan%40gfmail.com&subject=dedefde&message=dejdejdje


luego de vierificar e investigar creo que la vulnerabilidad puede estar en PHPSESSID, como el setvidor se corre en UBuntu,hare un Session POsinig.

s