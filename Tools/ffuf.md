### Web Fuzzing & Directory Brute-Forcing

ffuf (Fuzz Faster U Fool) is a fast web fuzzer for directory discovery, parameter fuzzing, vhost enumeration, and more. Uses the keyword `FUZZ` as a placeholder anywhere in a request.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install ffuf
```
### macOS

```bash
brew install ffuf
```
---
## Basic Syntax
```bash
ffuf -u <URL/FUZZ> -w <wordlist>
```

---
## Common Modes

### Directory / File Fuzzing
```bash
ffuf -u http://$IP/FUZZ -w /usr/share/wordlists/dirb/common.txt
```

- Replace path segment with `FUZZ`
- **Use case**: Discover hidden directories and files

### File Extension Fuzzing
```bash
ffuf -u http://$IP/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.html,.txt,.bak
```

- `-e`: Append extensions to each wordlist entry
- **Use case**: Find files when you know a directory exists

### Subdomain / VHost Fuzzing
```bash
ffuf -u http://FUZZ.target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

```bash
# VHost (when subdomains don't resolve via DNS)
ffuf -u http://$IP -H "Host: FUZZ.target.com" -w subdomains.txt
```

- `-H`: Inject custom header
- **Use case**: Enumerate subdomains or virtual hosts

### GET Parameter Fuzzing
```bash
ffuf -u "http://$IP/index.php?FUZZ=value" -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
```

### GET Parameter Value Fuzzing
```bash
ffuf -u "http://$IP/index.php?id=FUZZ" -w /usr/share/seclists/Fuzzing/integers.txt
```

### POST Body Fuzzing
```bash
ffuf -u http://$IP/login -X POST -d "username=admin&password=FUZZ" -w rockyou.txt -H "Content-Type: application/x-www-form-urlencoded"
```

- `-X POST`: Set request method
- `-d`: POST body data
- **Use case**: Brute-force login forms

---
## Filtering & Matching

By default ffuf shows all responses. Use filters to cut noise.

| Option | Description |
| ------ | ----------- |
| `-mc <codes>` | Match HTTP status codes (e.g. `200,301,302`) |
| `-ms <size>` | Match response size (bytes) |
| `-mw <words>` | Match word count in response |
| `-ml <lines>` | Match line count in response |
| `-mr <regex>` | Match regex in response body |
| `-fc <codes>` | Filter out status codes (e.g. `404`) |
| `-fs <size>` | Filter response size |
| `-fw <words>` | Filter word count |
| `-fl <lines>` | Filter line count |
| `-fr <regex>` | Filter regex in response body |

```bash
# Show only 200 responses
ffuf -u http://$IP/FUZZ -w wordlist.txt -mc 200

# Filter out a noisy response size (find baseline first, then filter it)
ffuf -u http://$IP/FUZZ -w wordlist.txt -fs 1234
```

- **Tip**: Run once without filters to see the baseline response size, then filter that size

---
## Common Options Reference

### Performance

| Option | Description |
| ------ | ----------- |
| `-t <n>` | Number of threads (default: 40) |
| `-rate <n>` | Max requests per second |
| `-timeout <n>` | HTTP timeout in seconds (default: 10) |
| `-p <delay>` | Delay between requests (e.g. `0.1`) |

### Output

| Option            | Description                                                |
| ----------------- | ---------------------------------------------------------- |
| `-o results.json` | Save output to file                                        |
| `-of json`        | Output format: `json`, `csv`, `ejson`, `html`, `md`, `all` |
| `-v`              | Verbose: show full URL and redirect target                 |
| `-s`              | Silent mode (no banner, just results)                      |

### Request Options

| Option | Description |
| ------ | ----------- |
| `-H "Header: val"` | Add custom header |
| `-X <method>` | HTTP method (GET/POST/PUT...) |
| `-d <data>` | POST data body |
| `-b "cookie=val"` | Cookie string |
| `-r` | Follow redirects |
| `-recursion` | Recursively fuzz discovered directories |
| `-recursion-depth <n>` | Max recursion depth |
| `-u <url>` | Target URL with FUZZ keyword |
| `-w <wordlist>` | Wordlist path (use `-` for stdin) |
| `-e <exts>` | File extensions to append |

### Multiple Wordlists (Multi-FUZZ)
```bash
ffuf -u http://$IP/FUZZ1/FUZZ2 -w wordlist1.txt:FUZZ1 -w wordlist2.txt:FUZZ2
```

- Name keywords with `-w wordlist:KEYWORD` syntax
- **Use case**: Fuzz multiple positions simultaneously

---
## Practical Examples

### Basic Directory Scan
```bash
ffuf -u http://$IP/FUZZ -w /usr/share/wordlists/dirb/common.txt -mc 200,301,302,403
```

### Recursive Directory Scan
```bash
ffuf -u http://$IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -recursion -recursion-depth 2
```

### PHP File Discovery
```bash
ffuf -u http://$IP/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php -mc 200
```

### Login Brute-Force (POST)
```bash
ffuf -u http://$IP/login -X POST -d "user=admin&pass=FUZZ" -w rockyou.txt \
  -H "Content-Type: application/x-www-form-urlencoded" -fc 200 -mr "Invalid"
```

- `-fc 200` + `-mr "Invalid"`: filter responses that contain the failure message
- Adjust match/filter based on how the app responds to bad creds

### Find Valid Usernames (Response Size Difference)
```bash
ffuf -u http://$IP/login -X POST -d "user=FUZZ&pass=wrongpass" -w usernames.txt \
  -H "Content-Type: application/x-www-form-urlencoded" -fs <baseline_size>
```

### VHost Enumeration
```bash
ffuf -u http://$IP -H "Host: FUZZ.target.htb" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -fs <baseline_size>
```

### API Endpoint Fuzzing
```bash
ffuf -u http://$IP/api/v1/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt -mc 200,201,204
```

---
## Output Interpretation

```
[Status: 200, Size: 4512, Words: 312, Lines: 87]    admin
[Status: 301, Size: 0, Words: 1, Lines: 1]           uploads
```

| Field | Meaning |
| ----- | ------- |
| `Status` | HTTP response code |
| `Size` | Response body size in bytes |
| `Words` | Word count in response |
| `Lines` | Line count in response |

---
## Tips & Best Practices

1. **Find baseline first** — run a quick scan and note the size of 404 responses, then filter with `-fs`
2. **Use SecLists** — `/usr/share/seclists/` has better wordlists than dirb for most scenarios
3. **Filter > Match** — filtering noise usually easier than matching valid responses
4. **Check status codes** — `301/302` are redirects worth following manually; `403` means the path exists but is forbidden
5. **Tune threads** — lower `-t` on slow targets to avoid false negatives from rate limiting
6. **Use `-v`** to see full redirect chains when `-r` is off
7. **Recursion is noisy** — use `-recursion-depth 1` or 2 max to avoid rabbit holes

---
## Quick Reference

```bash
# Directory fuzzing
ffuf -u http://$IP/FUZZ -w /usr/share/wordlists/dirb/common.txt

# With extensions
ffuf -u http://$IP/FUZZ -w wordlist.txt -e .php,.txt,.html

# Filter 404s by size
ffuf -u http://$IP/FUZZ -w wordlist.txt -fs 1234

# POST login brute-force
ffuf -u http://$IP/login -X POST -d "user=admin&pass=FUZZ" \
  -w rockyou.txt -H "Content-Type: application/x-www-form-urlencoded"

# VHost enumeration
ffuf -u http://$IP -H "Host: FUZZ.target.htb" -w subdomains.txt -fs 1234

# Save output
ffuf -u http://$IP/FUZZ -w wordlist.txt -o results.json -of json
```

---
## Related Tools

- **[Nmap](./Nmap.md)** - Port scan before fuzzing to find open web services
- **Gobuster** - Alternative directory fuzzer (simpler syntax)
- **Feroxbuster** - Recursive fuzzer with auto-filtering
- **Burp Suite** - Manual proxy to craft and replay requests found by ffuf
- **SecLists** - Wordlist collection (`apt install seclists`)

---
## Resources

- [ffuf GitHub](https://github.com/ffuf/ffuf)
- [SecLists GitHub](https://github.com/danielmiessler/SecLists)
- [ffuf Wiki](https://github.com/ffuf/ffuf/wiki)
