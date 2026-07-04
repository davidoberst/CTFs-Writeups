first we start with a nmap scan : 

[davidoberst@archlinux ~]$ sudo nmap -sV -O 10.67.189.47
[sudo] password for davidoberst: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-04 18:04 -0500
Nmap scan report for cyberlens.thm (10.67.189.47)
Host is up (0.072s latency).
Not shown: 994 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Apache httpd 2.4.57 ((Win64))
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).

Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows


una vez entre al puerto 80, encontre un sitio web, un sitio normal, con tematica de esteganogradia, un formulario de contacto y un "extractor de metadata, antes de revisatr los parametros mejor, quise hacer un fuzzxing para mpaear mejor la estructura dl sitio. 

al usar el comando 

[davidoberst@archlinux Web-Content]$ gobuster dir -u http://10.67.189.47 -w common.txt

pude encontrar lo siguiente : 
Images               (Status: 301) [Size: 235] [--> http://10.67.189.47/Images/]
css                  (Status: 301) [Size: 232] [--> http://10.67.189.47/css/]
images               (Status: 301) [Size: 235] [--> http://10.67.189.47/images/]
js                   (Status: 301) [Size: 231] [--> http://10.67.189.47/js/]
