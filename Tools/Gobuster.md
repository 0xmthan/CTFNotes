## Directory & DNS Brute-Forcer

Gobuster is a fast, Go-based tool for brute-forcing directories, files, DNS subdomains, and virtual hosts. Unlike ffuf it doesn't use a placeholder keyword — you just point it at a target with a wordlist and pick a mode.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install gobuster
```
### macOS

```bash
brew install gobuster
```
---
## Basic Syntax
```bash
gobuster <mode> -u <URL> -w <wordlist> [options]
```

---
## Modes

### dir — Directory & File Brute-Force
```bash
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/common.txt
```

- Discovers hidden paths and files on a web server
- **Use case**: Map out a web app's structure before manual enumeration

### dns — Subdomain Enumeration
```bash
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

- Resolves each wordlist entry as a subdomain against the target domain
- **Use case**: Find subdomains that don't appear in public DNS records

### vhost — Virtual Host Enumeration
```bash
gobuster vhost -u http://$IP -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

- Sends requests with a fuzzed `Host:` header; flags responses that differ from the baseline
- **Use case**: Find vhosts on a shared server when subdomains don't resolve via DNS (common in CTFs like HackTheBox)

---
## Common Options Reference

### dir Mode

| Option | Description |
| ------ | ----------- |
| `-u <url>` | Target URL |
| `-w <file>` | Wordlist |
| `-x <exts>` | File extensions to append (e.g. `php,html,txt`) |
| `-b <codes>` | Blacklist status codes (default: `404`) |
| `-s <codes>` | Whitelist status codes to show (e.g. `200,301,302`) |
| `-k` | Skip TLS certificate verification |
| `-c <cookie>` | Cookie string for authenticated requests |
| `-H "Header: val"` | Add custom HTTP header |
| `-U / -P` | Username / password for HTTP Basic Auth |
| `-r` | Follow redirects |
| `-t <n>` | Threads (default: 10) |
| `-o <file>` | Save output to file |
| `-q` | Quiet — no banner, only results |
| `-v` | Verbose — show all responses including errors |
| `--no-error` | Suppress error messages |
| `--timeout <dur>` | HTTP timeout (e.g. `10s`) |

### dns Mode

| Option | Description |
| ------ | ----------- |
| `-d <domain>` | Target domain |
| `-w <file>` | Wordlist of subdomain names |
| `-r <resolver>` | Use a custom DNS resolver (e.g. `8.8.8.8`) |
| `--show-ips` | Print resolved IP addresses alongside subdomains |
| `-i` | Show IP addresses (alias for `--show-ips`) |

### vhost Mode

| Option | Description |
| ------ | ----------- |
| `-u <url>` | Target base URL |
| `-w <file>` | Wordlist of vhost names |
| `--append-domain` | Append the base domain to each word (e.g. `FUZZ.target.com`) |
| `--exclude-length <n>` | Filter responses by body length (useful to drop baseline size) |

---
## Practical Examples

### Basic Directory Scan
```bash
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/common.txt
```

### Directory Scan with Extensions
```bash
gobuster dir -u http://$IP \
  -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt \
  -x php,html,txt,bak
```

### Show Only Useful Status Codes
```bash
gobuster dir -u http://$IP -w wordlist.txt -s 200,301,302,403
```

### Authenticated Scan (Cookie)
```bash
gobuster dir -u http://$IP/admin -w wordlist.txt -c "PHPSESSID=abc123"
```

### HTTPS Target (Skip TLS Check)
```bash
gobuster dir -u https://$IP -w wordlist.txt -k
```

### DNS Subdomain Enumeration
```bash
gobuster dns -d target.htb \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --show-ips
```

### VHost Enumeration (HTB Style)
```bash
gobuster vhost -u http://$IP -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain -t 50
```

- `--append-domain`: constructs `FUZZ.target.htb` — needed when the base domain isn't just an IP
- Filter noise with `--exclude-length <baseline_size>` after a test run

### Save Results to File
```bash
gobuster dir -u http://$IP -w wordlist.txt -o gobuster_results.txt
```

---
## Output Interpretation

```
/admin                (Status: 301) [Size: 312] [--> http://10.10.10.5/admin/]
/login.php            (Status: 200) [Size: 1842]
/uploads              (Status: 403) [Size: 276]
```

| Field | Meaning |
| ----- | ------- |
| `/path` | Discovered endpoint |
| `Status` | HTTP response code |
| `Size` | Response body size in bytes |
| `-->` | Redirect target (when following 301/302) |

---
## Tips & Best Practices

1. **Filter 404s by size, not code** — some apps return 200 for all paths; use `-x` + compare sizes manually or switch to [ffuf](./ffuf.md) for size-based filtering
2. **Always try `-x php,html,txt`** on web targets — bare directory scans miss files
3. **For vhost on HTB**, find the base domain first (`/etc/hosts` entry or page source), then use `--append-domain`
4. **Use SecLists over dirb** — `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt` is far more comprehensive
5. **Bump threads with `-t 50`** on stable targets; drop to `-t 5` if you're getting timeouts or dropped connections
6. **`-q` is useful for piping** output into `grep` or saving clean logs
7. **dns mode needs real domain names** — it won't work against bare IPs; add the target to `/etc/hosts` first if needed

---
## Quick Reference

```bash
# Directory brute-force
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/common.txt

# With extensions
gobuster dir -u http://$IP -w wordlist.txt -x php,html,txt

# Only show hits
gobuster dir -u http://$IP -w wordlist.txt -s 200,301,302,403

# Authenticated (cookie)
gobuster dir -u http://$IP -w wordlist.txt -c "session=abc123"

# DNS subdomain enum
gobuster dns -d target.htb -w subdomains.txt --show-ips

# VHost enum (HTB)
gobuster vhost -u http://$IP -w subdomains.txt --append-domain --exclude-length 1234

# Save output
gobuster dir -u http://$IP -w wordlist.txt -o out.txt
```

---
## Related Tools

- **[Nmap](./Nmap.md)** - Identify open web ports before brute-forcing
- **[ffuf](./ffuf.md)** - More flexible fuzzer with response filtering; better for parameter/POST fuzzing
- **[Hydra](./Hydra.md)** - Brute-force credentials once a login page is found
- **Feroxbuster** - Recursive directory fuzzer with auto-filtering; good alternative to Gobuster dir
- **SecLists** - Wordlist collection (`apt install seclists`)

---
## Resources

- [Gobuster GitHub](https://github.com/OJ/gobuster)
- [SecLists GitHub](https://github.com/danielmiessler/SecLists)
