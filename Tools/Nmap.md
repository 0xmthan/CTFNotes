## Network Mapping & Port Scanning

Nmap is a powerful open-source tool for network discovery and security auditing. It allows you to discover hosts and services on a network, identify open ports, detect operating systems, and gather information about network infrastructure.

---

## Basic Syntax

```zsh
nmap [Scan Type] [Options] [Target]
```

---

## Target Specifications

|Format|Description|
|---|---|
|`$IP`|Single IP address|
|`192.168.1.1-254`|IP range|
|`192.168.45.0/24`|CIDR notation (subnet)|
|`example.com`|Domain name|
|`-iL targets.txt`|Read targets from file|
|`--exclude 10.10.86.5`|Exclude specific host(s)|

---

## Scan Types

### Basic Service & OS Detection

```zsh
nmap -sC -sV $IP
```

- `-sC`: Run default NSE (Nmap Scripting Engine) scripts
- `-sV`: Probe open ports to determine service/version info
- **Use case**: Quick reconnaissance on a single target

### Ping Sweep (Host Discovery)

```zsh
sudo nmap -sP 192.168.45.0/24
```

- `-sP`: Ping scan only (no port scan)
- **Use case**: Identify live hosts on a network
- **Note**: Requires sudo for ICMP ping; use `-sn` for newer versions of nmap

### Aggressive Full Scan

```zsh
nmap -T4 -A -p-
```

- `-T4`: Timing template (aggressive, faster scan)
- `-A`: Enable OS detection, version detection, script scanning, and traceroute
- `-p-`: Scan all 65535 ports
- **Use case**: Comprehensive enumeration
- **Warning**: Slow and aggressive; may trigger IDS/IPS

### Extended Port Range Scan

```zsh
nmap -p-15000 -vv TARGET_IP -oG initial-scans
```

- `-p-15000`: Scan ports 1-15000
- `-vv`: Very verbose output
- `-oG initial-scans`: Output in greppable format
- **Use case**: Thorough scan of common ports with detailed output

---

## Common Options

### Port Selection

|Option|Description|
|---|---|
|`-p 80`|Scan specific port|
|`-p 80,443,8080`|Scan multiple ports|
|`-p 80-443`|Scan port range|
|`-p-`|Scan all ports (1-65535)|
|`-p U:53`|UDP port 53|
|`--top-ports 100`|Scan 100 most common ports|

### Scan Types

|Option|Description|
|---|---|
|`-sS`|TCP SYN scan (stealth, requires sudo)|
|`-sT`|TCP connect scan|
|`-sU`|UDP scan|
|`-sA`|TCP ACK scan (firewall mapping)|
|`-sn`|Ping sweep (no port scan)|
|`-Pn`|Skip ping; assume all hosts alive|

### Version & OS Detection

|Option|Description|
|---|---|
|`-sV`|Service/version detection|
|`-sC`|Run default NSE scripts|
|`-O`|OS detection (requires sudo, unreliable)|
|`-A`|Aggressive: OS, version, script, traceroute|

### Timing & Performance

|Option|Description|
|---|---|
|`-T0`|Paranoid (very slow)|
|`-T1`|Sneaky (slow)|
|`-T2`|Polite (normal)|
|`-T3`|Normal (default)|
|`-T4`|Aggressive (fast)|
|`-T5`|Insane (very fast, unreliable)|
|`--max-retries 1`|Reduce retry attempts|
|`--max-rtt-timeout 100ms`|Set timeout|

### Output Options

|Option|Description|
|---|---|
|`-v`|Verbose output|
|`-vv`|Very verbose output|
|`-oN file.txt`|Normal output|
|`-oX file.xml`|XML output|
|`-oG file.txt`|Greppable output|
|`-oA basename`|Output in all formats|

### Other Useful Options

|Option|Description|
|---|---|
|`--script scriptname`|Run specific NSE script|
|`--script vuln`|Run vulnerability scanning scripts|
|`-6`|IPv6 scanning|
|`--reason`|Show why port is in given state|
|`--packet-trace`|Show all packets sent/received|

---

## Practical Examples

### Discover Live Hosts on Network

```zsh
sudo nmap -sn 192.168.1.0/24
```

### Quick Service Discovery

```zsh
nmap -sC -sV $IP
```

### Stealth SYN Scan (SYN stealth)

```zsh
sudo nmap -sS -p- -T2 $IP
```

### Comprehensive Enumeration

```zsh
nmap -T4 -A -p- -v $IP -oA results
```

### Scan with Specific Script

```zsh
nmap --script smb-enum-shares -p 445 $IP
```

### Check for Vulnerabilities

```zsh
nmap --script vuln -p- $IP
```

### Firewall Evasion (Fragment packets)

```zsh
nmap -f $IP
```

### Aggressive Scan with Output Formats

```zsh
nmap -T4 -A -p- -vv $IP -oN scan.txt -oX scan.xml -oG scan.grep
```

---

## Output Interpretation

### Port States

|State|Meaning|
|---|---|
|**open**|Service actively accepting connections|
|**closed**|Port accessible but no service listening|
|**filtered**|Firewall/filter blocking probe; unknown state|
|**unfiltered**|Port accessible but state undetermined|
|**open\|filtered**|Likely open but filtering makes certainty impossible|
|**closed\|filtered**|Likely closed but filtering makes certainty impossible|

### Service Information

```
80/tcp   open     http        Apache httpd 2.4.41 (Ubuntu)
443/tcp  open     ssl/http    Apache httpd 2.4.41 (Ubuntu)
22/tcp   open     ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
```

---

## Tips & Best Practices

1. **Start with service discovery** (`-sC -sV`) before aggressive scans
2. **Use timing templates** (`-T`) based on network stability:
    - `-T2` for stealth/IDS evasion
    - `-T4` for trusted internal networks
3. **Filter output** with grep for easy parsing:
    
    ```zsh
    nmap -p- $IP | grep "open"
    ```
    
4. **Save results** in multiple formats (`-oA`) for later analysis
5. **Combine tools**: Use nmap with other tools like:
    - `masscan` for faster initial scans
    - `gobuster` for web enumeration
    - `hydra` for credential testing
6. **Review NSE scripts** before running unknown scripts
7. **Use sudo** for advanced scan types (SYN, OS detection, ICMP)

---

## Common Scan Workflows

### Reconnaissance Phase

```zsh
# 1. Discover live hosts
sudo nmap -sn 192.168.1.0/24 -oG live-hosts.txt

# 2. Quick service enumeration
nmap -sC -sV -p- 10.10.1.5 -oA service-enum
```

### Exploitation Phase

```zsh
# 3. Detailed vulnerability scan
nmap --script vuln -p- 10.10.1.5 -oX vuln-report.xml

# 4. Specific service enumeration
nmap --script smb-enum* -p 445 10.10.1.5
```

### Validation Phase

```zsh
# 5. Verify findings
nmap -sC -sV -p 22,80,443,445 10.10.1.5
```

---

## Security Considerations
- Nmap scans can be detected by IDS/IPS systems
- Aggressive scans may disrupt network services
- Different scan types have different detection profiles
- Timing options can help evade detection but reduce reliability
- Always document your authorization and findings

---

## Resources

- [Official Nmap Documentation](https://nmap.org/book/)
- [NSE Script Reference](https://nmap.org/nsedoc/)
- [Nmap Usage Examples](https://nmap.org/man/man1/nmap.1.html)