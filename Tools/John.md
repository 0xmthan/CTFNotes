## CPU-Based Password Cracking

John the Ripper (JtR) is a CPU-based password cracker that auto-detects hash types and handles a wide range of formats including Unix shadows, Windows hashes, ZIP/SSH/GPG encrypted files...

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install john
```

```bash
# Jumbo version (more formats, recommended)
sudo apt install john-data
```
### macOS

```bash
brew install john-jumbo
```
---
## Basic Syntax
```bash
john [options] <hashfile>
```

---
## Hash Identification & Preparation

```bash
# Auto-detect and crack
john hash.txt

# Identify hash type manually
john --list=formats | grep -i md5
hash-identifier
```

- JtR auto-detects many formats — try it without `--format` first
- If auto-detect fails, specify `--format=<type>`

---
## Attack Modes

### Wordlist Attack
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

- **Use case**: Most common CTF scenario — try this first

### Wordlist + Rules
```bash
john --wordlist=rockyou.txt --rules hash.txt
```

- `--rules`: Apply mangling rules (capitalize, append digits, leet speak, etc.)
- **Use case**: When plain wordlist fails

### Incremental (Brute-Force)
```bash
john --incremental hash.txt
```

- Tries all character combinations in increasing length
- **Use case**: Last resort when wordlist + rules fail; slow

### Single Crack Mode
```bash
john --single hash.txt
```

- Uses login name, GECOS fields, and simple derivations as candidates
- **Use case**: Quick pass when you have username context (e.g., `/etc/shadow`)

---
## Specifying Hash Format
```bash
john --format=<type> hash.txt
```

### Common Formats

| Format Flag | Hash Type |
| ----------- | --------- |
| `--format=raw-md5` | MD5 |
| `--format=raw-sha1` | SHA-1 |
| `--format=raw-sha256` | SHA-256 |
| `--format=bcrypt` | bcrypt `$2a$` |
| `--format=NT` | NTLM (Windows) |
| `--format=sha512crypt` | Linux shadow `$6$` |
| `--format=md5crypt` | Linux shadow `$1$` |
| `--format=ssh` | SSH private key passphrase |
| `--format=zip` | ZIP archive password |
| `--format=rar` | RAR archive password |
| `--format=pdf` | PDF password |
| `--format=pgp` | GPG/PGP key passphrase |

```bash
# List all supported formats
john --list=formats
```

---
## File-to-Hash Conversion (ssh2john, zip2john, etc.)

JtR ships with helper scripts that extract crackable hashes from protected files.

```bash
# SSH private key
ssh2john id_rsa > id_rsa.hash
john --wordlist=rockyou.txt id_rsa.hash

# ZIP file
zip2john secret.zip > zip.hash
john --wordlist=rockyou.txt zip.hash

# RAR file
rar2john secret.rar > rar.hash
john --wordlist=rockyou.txt rar.hash

# PDF file
pdf2john secret.pdf > pdf.hash
john --wordlist=rockyou.txt pdf.hash

# GPG key
gpg2john key.gpg > gpg.hash
john --wordlist=rockyou.txt gpg.hash

# /etc/shadow (combine with /etc/passwd first)
unshadow /etc/passwd /etc/shadow > combined.txt
john --wordlist=rockyou.txt combined.txt
```

- Helper scripts live in `/usr/share/john/` or `/opt/john/run/`
- On some systems: `python3 /usr/share/john/ssh2john.py id_rsa > hash`

---
## Showing & Managing Results

```bash
# Show cracked passwords (read from potfile)
john --show hash.txt

# Show with format specified
john --show --format=raw-md5 hash.txt

# Check session progress
john --status

# Resume a session
john --restore

# Named session
john --session=ctf1 --wordlist=rockyou.txt hash.txt
john --restore=ctf1
```

- JtR saves cracked passwords to `~/.john/john.pot`
- Always run `--show` after a crack attempt — results persist in potfile

---
## Common Options Reference

| Option | Description |
| ------ | ----------- |
| `--wordlist=<file>` | Path to wordlist |
| `--rules` | Enable word mangling rules |
| `--rules=<ruleset>` | Use specific ruleset (e.g. `Jumbo`) |
| `--format=<type>` | Specify hash format |
| `--incremental` | Brute-force mode |
| `--single` | Single crack mode (uses login info) |
| `--show` | Display cracked passwords |
| `--status` | Show session progress |
| `--session=<name>` | Name session for resuming |
| `--restore[=<name>]` | Resume a stopped session |
| `--fork=<n>` | Use N processes (multi-core) |
| `--list=formats` | List all supported hash formats |
| `--pipe` | Read words from stdin |

---
## Practical Examples

### Crack Linux Shadow File
```bash
unshadow /etc/passwd /etc/shadow > shadow.txt
john --wordlist=rockyou.txt shadow.txt
john --show shadow.txt
```

### Crack NTLM Hash
```bash
echo "aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bd760f388b67351e2b" > ntlm.txt
john --format=NT --wordlist=rockyou.txt ntlm.txt
```

### Crack SSH Private Key
```bash
ssh2john id_rsa > id_rsa.hash
john --wordlist=rockyou.txt id_rsa.hash
john --show id_rsa.hash
```

### Crack ZIP Password
```bash
zip2john protected.zip > zip.hash
john --wordlist=rockyou.txt zip.hash
john --show zip.hash
```

### Crack bcrypt with Rules
```bash
john --format=bcrypt --wordlist=rockyou.txt --rules hash.txt
```

### Multi-Core Crack
```bash
john --fork=4 --wordlist=rockyou.txt hash.txt
```

### Brute-Force Short Password (Incremental)
```bash
john --incremental=digits hash.txt   # digits only
john --incremental=alpha hash.txt    # letters only
john --incremental hash.txt          # all chars
```

---
## Output Interpretation

```
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
password123      (root)
1g 0:00:00:04 DONE (2026-04-17 12:00) 0.2381g/s 3657p/s
```

| Field | Meaning |
| ----- | ------- |
| `Loaded N password hash` | Hashes detected and loaded |
| `plaintext (username)` | Cracked password and its account |
| `Ng 0:00:...` | N cracked, time elapsed, speed |
| `DONE` | Wordlist/mode exhausted |

---
## Tips & Best Practices

1. **Always try `--single` first** — fast and works when username hints at password
2. **Then wordlist → wordlist + rules → incremental** — escalate gradually
3. **Check the potfile** before cracking — results persist between runs:
    ```bash
    john --show hash.txt
    ```
4. **Use `unshadow`** before cracking `/etc/shadow` — JtR needs combined file
5. **Use `--fork=N`** to saturate all CPU cores on long cracks
6. **Jumbo build > stock build** — supports SSH keys, ZIPs, PDFs, and more
7. **Prefer [Hashcat](./Hashcat.md) for GPU** — JtR is CPU-only; use hashcat when speed matters
8. **Identify format if auto-detect fails**:
    ```bash
    john --list=formats | grep -i <type>
    ```

---
## Quick Reference

```bash
# Auto-detect and crack
john --wordlist=rockyou.txt hash.txt

# With rules
john --wordlist=rockyou.txt --rules hash.txt

# Specify format
john --format=raw-md5 --wordlist=rockyou.txt hash.txt

# SSH key
ssh2john id_rsa > id_rsa.hash && john --wordlist=rockyou.txt id_rsa.hash

# ZIP
zip2john file.zip > zip.hash && john --wordlist=rockyou.txt zip.hash

# Linux shadow
unshadow /etc/passwd /etc/shadow > shadow.txt && john --wordlist=rockyou.txt shadow.txt

# Show cracked
john --show hash.txt

# Resume session
john --restore
```

---
## Related Tools

- **[Hashcat](./Hashcat.md)** - GPU-accelerated cracker; faster for large wordlists and brute-force
- **hash-identifier** - Identify unknown hash types
- **SecLists** - Wordlist collection (`apt install seclists`)
- **CrackStation** - Online lookup for common hashes

---
## Resources

- [John the Ripper Official Site](https://www.openwall.com/john/)
- [Jumbo GitHub](https://github.com/openwall/john)
- [John the Ripper Wiki](https://www.openwall.com/john/doc/)
