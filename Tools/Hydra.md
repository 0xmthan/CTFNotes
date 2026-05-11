## Network Login Brute-Forcer

Hydra is a fast, parallelized login cracker that supports dozens of protocols — SSH, FTP, HTTP, SMB, RDP, MySQL, and more. Used to brute-force credentials when you have a username or wordlist to test.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install hydra
```
### macOS

```bash
brew install hydra
```
---
## Basic Syntax
```bash
hydra -l <user> -P <wordlist> <protocol>://<target>
```

---
## Attack Modes

### Single Username + Wordlist
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://$IP
```

- `-l`: Single known username
- `-P`: Path to password wordlist
- **Use case**: You know the username, brute-force the password

### Username List + Wordlist
```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://$IP
```

- `-L`: Wordlist of usernames to try
- **Use case**: Enumerate both username and password

### Single Username + Single Password
```bash
hydra -l admin -p password123 ssh://$IP
```

- `-p`: Single password to test
- **Use case**: Credential stuffing / validating a specific pair

### Username + Password from Same File
```bash
hydra -C credentials.txt ftp://$IP
```

- `-C`: Colon-separated `user:pass` file
- **Use case**: Testing a known credential dump

---
## Protocol-Specific Examples

### SSH
```bash
hydra -l root -P rockyou.txt ssh://$IP
hydra -l root -P rockyou.txt -s 2222 ssh://$IP   # non-default port
```

### FTP
```bash
hydra -l admin -P rockyou.txt ftp://$IP
```

### HTTP GET (Basic Auth)
```bash
hydra -l admin -P rockyou.txt $IP http-get /protected/
```

### HTTP POST Form
```bash
hydra -l admin -P rockyou.txt $IP http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid credentials"
```

- Format: `/path:POST_body:failure_string`
- `^USER^` and `^PASS^` are placeholders Hydra replaces
- `F=` defines the failure string (what the page says on bad login)
- **Use case**: Brute-force web login forms

### HTTP POST Form (with port)
```bash
hydra -l admin -P rockyou.txt $IP http-post-form "/login:user=^USER^&pass=^PASS^:F=Wrong" -s 8080
```

### RDP
```bash
hydra -l administrator -P rockyou.txt rdp://$IP
```

### SMB
```bash
hydra -l admin -P rockyou.txt smb://$IP
```

### MySQL
```bash
hydra -l root -P rockyou.txt mysql://$IP
```

---
## Common Options Reference

| Option | Description |
| ------ | ----------- |
| `-l <user>` | Single username |
| `-L <file>` | Username wordlist |
| `-p <pass>` | Single password |
| `-P <file>` | Password wordlist |
| `-C <file>` | Colon-separated `user:pass` file |
| `-s <port>` | Non-default port |
| `-t <n>` | Number of parallel tasks (default: 16) |
| `-f` | Stop after first valid pair found |
| `-F` | Stop on first valid pair per host |
| `-v` | Verbose output |
| `-V` | Very verbose (show each attempt) |
| `-d` | Debug mode |
| `-o <file>` | Save output to file |
| `-e nsr` | Try empty pass, login as pass, reversed login |
| `-u` | Loop around users, not passwords |
| `-w <sec>` | Wait time between retries (seconds) |
| `-x min:max:charset` | Password generation mode |

---
## Practical Examples

### Brute-Force SSH with Verbose Output
```bash
hydra -l john -P rockyou.txt -v ssh://$IP
```

### Stop on First Hit
```bash
hydra -l admin -P rockyou.txt -f ssh://$IP
```

### Web Login Form (Full Example)
```bash
hydra -l admin -P rockyou.txt $IP http-post-form \
  "/admin/login:username=^USER^&password=^PASS^:F=Login failed" -V
```

### Try Empty & Common Variations
```bash
hydra -l admin -P rockyou.txt -e nsr ssh://$IP
```

- `-e n`: try empty password
- `-e s`: try username as password
- `-e r`: try reversed username as password

### Custom Port + Output File
```bash
hydra -l root -P rockyou.txt -s 2222 -o found.txt ssh://$IP
```

### Generate Passwords on the Fly (e.g. 4-digit PINs)
```bash
hydra -l admin -x 4:4:1 ssh://$IP
```

- `-x 4:4:1`: min 4 chars, max 4 chars, charset `1` = digits only

---
## Output Interpretation

```
[22][ssh] host: 10.10.10.5   login: john   password: letmein
[STATUS] 1328 tries/min, 1328 tries in 00:01h, 13353021 remaining
```

| Field | Meaning |
| ----- | ------- |
| `[port][protocol]` | Service that was cracked |
| `host` | Target IP |
| `login` | Valid username found |
| `password` | Valid password found |
| `tries/min` | Current attack speed |
| `remaining` | Attempts left in wordlist |

---
## Failure String Tips

Finding the right failure string for HTTP forms:

1. Submit a wrong login manually in a browser
2. Note the error message (e.g. `"Invalid credentials"`, `"Login failed"`)
3. Pass that string to `F=` in the http-post-form module

```bash
# Use curl to confirm the failure string
curl -s -X POST http://$IP/login -d "user=fake&pass=fake" | grep -i "fail\|invalid\|wrong"
```

---
## Tips & Best Practices

1. **Use `-f` on CTFs** — stop after first hit, saves time
2. **Always use `-V` first** on HTTP forms to verify the request format is correct
3. **Find baseline failure string** before running — wrong `F=` means Hydra thinks everything is valid
4. **Lower `-t`** on slow/rate-limited targets: `-t 4` or `-t 1` to avoid lockouts
5. **Try `-e nsr` first** — catches trivial creds (empty, user=pass) without a full wordlist
6. **Check the port** — services often run on non-standard ports; always confirm with Nmap first
7. **Prefer [ffuf](./ffuf.md) for HTTP brute-force** — more flexible filtering; Hydra is better for non-HTTP protocols

---
## Quick Reference

```bash
# SSH brute-force
hydra -l admin -P rockyou.txt ssh://$IP

# FTP brute-force
hydra -l admin -P rockyou.txt ftp://$IP

# HTTP Basic Auth
hydra -l admin -P rockyou.txt $IP http-get /protected/

# HTTP POST form
hydra -l admin -P rockyou.txt $IP http-post-form "/login:user=^USER^&pass=^PASS^:F=Invalid"

# Stop on first hit, verbose
hydra -l admin -P rockyou.txt -f -V ssh://$IP

# Non-default port
hydra -l admin -P rockyou.txt -s 2222 ssh://$IP

# Try empty/reversed creds
hydra -l admin -P rockyou.txt -e nsr ssh://$IP
```

---
## Related Tools

- **[Nmap](./Nmap.md)** - Identify open services/ports before attacking
- **[ffuf](./ffuf.md)** - Better for HTTP form fuzzing with response filtering
- **[John the Ripper](./John.md)** - Crack offline hashes; Hydra is for live services
- **Medusa** - Alternative parallelized login brute-forcer
- **CrackMapExec** - SMB/AD credential testing at scale

---
## Resources

- [Hydra GitHub](https://github.com/vanhauser-thc/thc-hydra)
- [Hydra Man Page](https://manpages.debian.org/unstable/hydra/hydra.1.en.html)
