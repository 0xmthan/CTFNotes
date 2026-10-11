## Internet-Scale Port Scanner

masscan is an extremely fast port scanner — it can scan large ranges in a fraction of the time Nmap takes by using its own asynchronous TCP/IP stack and transmitting probes without waiting for replies. In CTFs the usual workflow is: masscan to find open ports fast, then hand those ports to [Nmap](./Nmap.md) for service/version detection.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install masscan
```
### macOS

```bash
brew install masscan
```
### Verify
```bash
masscan --version
```
---
## Basic Syntax
```bash
sudo masscan <targets> -p<ports> [options]
```

- Needs `sudo`/root — it crafts raw packets
- `--rate` controls packets/sec; **this is the knob that matters most**

---
## Common Options Reference

| Option | Description |
| ------ | ----------- |
| `-p <ports>` | Ports: `-p80`, `-p1-65535`, `-p80,443,8080`, `-pU:53` (UDP) |
| `-p-` | All 65535 TCP ports |
| `--rate <n>` | Packets per second (default is slow; bump to 1000–10000) |
| `--range <cidr>` / `<ip>` | Targets (also positional) |
| `-iL <file>` | Read targets from a file |
| `--exclude <ip>` | Skip host(s) |
| `--excludefile <file>` | Skip a list of hosts |
| `-e <iface>` | Interface to use (e.g. `tun0` on VPN) |
| `--router-ip <ip>` | Set gateway manually (needed on some VPNs) |
| `-oL / -oJ / -oX <file>` | Output: list / JSON / XML |
| `-oG <file>` | Greppable output |
| `--open-only` | Report only open ports |
| `--banners` | Grab basic banners (slower) |
| `--ping` | ICMP echo (host discovery) |
| `--resume paused.conf` | Resume an interrupted scan |

---
## Practical Examples

### Fast Full-Port Sweep of One Host
```bash
sudo masscan 10.10.10.5 -p1-65535 --rate 1000
```

- Finishes in seconds vs minutes for `nmap -p-`

### Scan a Subnet for Common Web Ports
```bash
sudo masscan 192.168.1.0/24 -p80,443,8080,8443 --rate 1000 --open-only
```

### Scan over a VPN Interface (HTB/THM)
```bash
sudo masscan 10.10.10.5 -p1-65535 --rate 1000 -e tun0
```

- On VPNs masscan sometimes can't auto-detect the gateway — add `--router-ip <vpn_gateway>` if it hangs

### Output Straight into an Nmap Follow-up
```bash
# 1. masscan finds open ports fast
sudo masscan 10.10.10.5 -p1-65535 --rate 1000 -oG masscan.gnmap

# 2. extract the open ports
ports=$(grep -oP '\d+/open' masscan.gnmap | cut -d/ -f1 | paste -sd,)

# 3. deep scan just those ports with Nmap
nmap -sC -sV -p$ports 10.10.10.5 -oA nmap-deep
```

### UDP Scan
```bash
sudo masscan 10.10.10.5 -pU:53,161,123 --rate 1000
```

### Save as JSON for Parsing
```bash
sudo masscan 10.10.10.5 -p- --rate 1000 -oJ scan.json
```

---
## Output Interpretation

```
Discovered open port 22/tcp on 10.10.10.5
Discovered open port 80/tcp on 10.10.10.5
Discovered open port 3306/tcp on 10.10.10.5
```

| Field | Meaning |
| ----- | ------- |
| `Discovered open port` | A probe got a SYN-ACK back |
| `22/tcp` | Port number and protocol |
| `on 10.10.10.5` | Which host responded |

- masscan reports **open** ports only — it does **not** identify the service (that's Nmap's job)

---
## Tips & Best Practices

1. **masscan finds ports, Nmap identifies them** — always follow a masscan sweep with `nmap -sC -sV -p<ports>`
2. **Tune `--rate` to the network** — `1000` is safe on CTF VPNs; `10000`+ can drop packets or saturate a slow `tun0`, causing false negatives
3. **Too high a rate = missed ports** — if results look thin, re-run slower (`--rate 300`)
4. **Use `-e tun0`** on HTB/THM so probes go out the VPN, not your real interface
5. **If it hangs on a VPN**, set `--router-ip` to the VPN gateway manually
6. **`--open-only` keeps output clean** when scanning ranges
7. **Don't point it at the public internet** without authorization — the whole design is for scale, and it's noisy/illegal against systems you don't own

---
## Quick Reference

```bash
# Full TCP port sweep, one host
sudo masscan 10.10.10.5 -p1-65535 --rate 1000

# Web ports across a subnet, open only
sudo masscan 192.168.1.0/24 -p80,443,8080 --rate 1000 --open-only

# Over a VPN interface
sudo masscan 10.10.10.5 -p- --rate 1000 -e tun0

# Greppable output -> Nmap follow-up
sudo masscan 10.10.10.5 -p- --rate 1000 -oG masscan.gnmap
ports=$(grep -oP '\d+/open' masscan.gnmap | cut -d/ -f1 | paste -sd,)
nmap -sC -sV -p$ports 10.10.10.5

# UDP
sudo masscan 10.10.10.5 -pU:53,161 --rate 1000
```

---
## Related Tools

- **[Nmap](./Nmap.md)** - The essential follow-up: service/version/OS detection on the ports masscan finds
- **[netcat](./netcat.md)** - Quick manual check or banner grab on a single discovered port
- **[arp-scan](./arp-scan.md)** / **[netdiscover](./netdiscover.md)** - Layer-2 host discovery before you scan ports
- **RustScan** - Similar fast-scan-then-Nmap workflow in one tool

---
## Resources

- [masscan GitHub](https://github.com/robertdavidgraham/masscan)
- [masscan README / Usage](https://github.com/robertdavidgraham/masscan#usage)
