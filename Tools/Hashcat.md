## GPU-Accelerated Password Cracking

Hashcat uses your GPU to crack hashed passwords via dictionary attacks, brute-force, rule-based attacks...

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install hashcat
```
### macOS

```bash
brew install hashcat
```
---
## Basic Syntax
```bash
hashcat -m <hash_type> -a <attack_mode> <hashfile> <wordlist/mask>
```

---
## Attack Modes

| Mode | Flag | Description |
| ---- | ---- | ----------- |
| Dictionary | `-a 0` | Try each word from a wordlist |
| Combinator | `-a 1` | Combine words from two wordlists |
| Brute-Force | `-a 3` | Try all combinations using a mask |
| Hybrid Dict+Mask | `-a 6` | Wordlist words with appended mask |
| Hybrid Mask+Dict | `-a 7` | Mask prepended to wordlist words |

---
## Common Hash Types

| Hash | `-m` Value | Example Hash |
| ---- | ---------- | ------------ |
| MD5 | `0` | `5f4dcc3b5aa765d61d8327deb882cf99` |
| SHA-1 | `100` | `5baa61e4c9b93f3f0682250b6cf8331b7ee68fd8` |
| SHA-256 | `1400` | `5e884898da28...` |
| SHA-512 | `1700` | `b109f3bbbc24...` |
| bcrypt | `3200` | `$2a$05$...` |
| NTLM | `1000` | `b4b9b02e6f09...` |
| NetNTLMv2 | `5600` | `admin::WORKGROUP:...` |
| WPA/WPA2 | `22000` | `.hc22000` capture file |
| MD5crypt | `500` | `$1$salt$hash` |
| SHA-512crypt | `1800` | `$6$salt$hash` |

- Run `hashcat --example-hashes` to see all supported formats with examples
- Identify an unknown hash with `hashid` or `hash-identifier`

---

## Dictionary Attack (`-a 0`)

```bash
hashcat -m <hash_type> -a 0 hash.txt /path/to/rockyou.txt
```

- `-m`: Hash type (see table above)
- `-a 0`: Dictionary attack mode
- `hash.txt`: File containing the hash(es) to crack
- **Use case**: Most common CTF scenario — always try this first

### With Rules

```bash
hashcat -m 0 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

- `-r`: Apply transformation rules (capitalize, add numbers, leet speak, etc.)
- **Popular rule files**: `best64.rule`, `rockyou-30000.rule`, `d3ad0ne.rule`
- **Use case**: When a plain wordlist fails — rules mutate each word

---
## Brute-Force / Mask Attack (`-a 3`)

```bash
hashcat -m <hash_type> -a 3 hash.txt ?a?a?a?a?a?a
```
### Mask Charset Reference

| Mask | Characters |
| ---- | ---------- |
| `?l` | Lowercase letters (`a-z`) |
| `?u` | Uppercase letters (`A-Z`) |
| `?d` | Digits (`0-9`) |
| `?s` | Special characters |
| `?a` | All of the above combined |
| `?b` | All bytes (0x00-0xFF) |

- Each `?x` in the mask = one character position
- **Example**: `?u?l?l?l?d?d` = one uppercase + three lowercase + two digits
- **Use case**: When you know the password format or length

### Increment Mode (Try all lengths up to N)

```bash
hashcat -m 0 -a 3 hash.txt --increment --increment-min 4 --increment-max 8 ?a?a?a?a?a?a?a?a
```

- `--increment`: Try masks of increasing length
- `--increment-min` / `--increment-max`: Length range to try
---
## Combinator Attack (`-a 1`)

```bash
hashcat -m 0 -a 1 hash.txt wordlist1.txt wordlist2.txt
```

- Concatenates every word from list1 with every word from list2
- **Use case**: When password is two words joined (e.g., `sunflower2024`)

---
## Hybrid Attacks (`-a 6` / `-a 7`)
### Dictionary + Mask (append)
```bash
hashcat -m 0 -a 6 hash.txt rockyou.txt ?d?d?d?d
```

- Takes words from wordlist and appends the mask
- **Example output**: `password1234`, `dragon2025`

### Mask + Dictionary (prepend)
```bash
hashcat -m 0 -a 7 hash.txt ?d?d?d?d rockyou.txt
```

- Prepends the mask to each word
- **Example output**: `2024password`, `1337dragon`
---
## WPA/WPA2 Cracking

```bash
# Convert .cap to hashcat format first
hcxpcapngtool -o hash.hc22000 capture.cap

# Crack with dictionary
hashcat -m 22000 hash.hc22000 rockyou.txt
```

- Requires `.hc22000` format (replaces old `-m 2500`)
- Pair with airmon-ng to capture the handshake first
- **Use case**: Cracking captured WiFi handshakes

---
## Common Options Reference
### Performance

| Option                      | Description                           |
| --------------------------- | ------------------------------------- |
| `-w 1`                      | Workload profile: Low                 |
| `-w 2`                      | Workload profile: Default             |
| `-w 3`                      | Workload profile: High                |
| `-w 4`                      | Workload profile: Nightmare (max GPU) |
| `--force`                   | Override warnings (use with caution)  |
| `--opencl-device-types 1,2` | Use CPU + GPU                         |

### Output & Session

| Option                  | Description                              |
| ----------------------- | ---------------------------------------- |
| `-o cracked.txt`        | Write cracked passwords to file          |
| `--outfile-format 2`    | Output format (hash:plain)               |
| `--session <name>`      | Name this session for resuming           |
| `--restore`             | Resume a previous session                |
| `--show`                | Show already-cracked hashes from potfile |
| `--potfile-path <file>` | Custom potfile location                  |

### Other Useful Options

| Option | Description |
| ------ | ----------- |
| `--username` | Hash file includes usernames (`user:hash` format) |
| `--remove` | Remove cracked hashes from hash file |
| `-O` | Optimized kernel (faster, limits max password length) |
| `--status` | Enable automatic status updates |
| `--runtime <sec>` | Stop after N seconds |

---

## Practical Examples

### Crack a Single MD5 Hash

```bash
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hash.txt
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

### Crack NTLM (Windows) Hash

```bash
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

### Crack SHA-1 with Brute-Force (6 digits)

```bash
hashcat -m 100 -a 3 hash.txt ?d?d?d?d?d?d
```

### Crack bcrypt with Dictionary

```bash
hashcat -m 3200 -a 0 hash.txt rockyou.txt -w 3
```

- **Note**: bcrypt is slow by design — GPU acceleration matters most here

### Resume a Session

```bash
hashcat --session mysession --restore
```

### Show Already-Cracked Hashes

```bash
hashcat -m 0 hash.txt --show
```

---

## Output Interpretation

```
5f4dcc3b5aa765d61d8327deb882cf99:password

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Time.Started.....: Mon Apr 14 12:00:00 2026
Time.Estimated...: Cracked
Guess.Base.......: File (rockyou.txt)
Speed.#1.........: 1234.5 MH/s
Recovered........: 1/1 (100.00%) Digests
```

| Field | Meaning |
| ----- | ------- |
| `Status: Cracked` | Password found |
| `Status: Exhausted` | Wordlist/mask finished, not found |
| `Status: Running` | Still in progress |
| `Speed` | Hashes attempted per second |
| `Recovered` | How many hashes cracked out of total |

---

## Tips & Best Practices

1. **Always try rockyou.txt first** with a few rules before brute-forcing
2. **Use `-O` (optimized kernel)** for a speed boost when passwords are under 32 chars
3. **Check the potfile** before cracking — hashcat caches previous results:
    ```bash
    hashcat -m 0 hash.txt --show
    ```
4. **Identify hash type** before picking `-m`:
    ```bash
    hashid hash.txt
    # or
    hash-identifier
    ```
5. **Layer your approach** — escalate from fast to slow:
    - Dictionary (`-a 0`) → Dictionary + Rules → Hybrid → Brute-Force (`-a 3`)
6. **Save sessions** for long cracks so you can resume:
    ```bash
    hashcat --session ctf1 -m 0 -a 0 hash.txt rockyou.txt
    ```
7. **Monitor GPU temperature** during long sessions with `watch -n 1 nvidia-smi`

---

## Quick Reference

```bash
# Dictionary attack
hashcat -m 0 -a 0 hash.txt rockyou.txt

# Dictionary + rules
hashcat -m 0 -a 0 hash.txt rockyou.txt -r best64.rule

# Brute-force 6-char (lowercase + digits)
hashcat -m 0 -a 3 hash.txt ?l?l?l?l?d?d

# Hybrid (word + 4 digits)
hashcat -m 0 -a 6 hash.txt rockyou.txt ?d?d?d?d

# Show cracked results
hashcat -m 0 hash.txt --show

# WPA/WPA2
hashcat -m 22000 handshake.hc22000 rockyou.txt
```

---

## Related Tools

- **[Airmon-ng / Aircrack-ng](./Airmon-ng.md)** - Capture WPA handshakes to feed into hashcat
- **John the Ripper** - Alternative CPU-based cracker, useful for SSH keys
- **hcxtools** - Convert `.cap` files to hashcat-compatible format
- **CrackStation** - Online lookup for common hashes

---

## Resources

- [Official Hashcat Documentation](https://hashcat.net/wiki/)
- [Hashcat Example Hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)
- [Hashcat Rule Files](https://github.com/hashcat/hashcat/tree/master/rules)
- [Hashes.com (Online Lookup)](https://hashes.com/en/decrypt/hash)
