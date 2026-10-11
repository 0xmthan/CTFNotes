## Active/Passive ARP Reconnaissance

netdiscover is an ARP-based host discovery tool with a live, auto-refreshing TUI. It can actively scan a range or sit **passively** sniffing ARP traffic without sending a single packet — useful when you want to map a network quietly. Like [arp-scan](./arp-scan.md) it works at layer 2, so it finds hosts that ignore ICMP, and it reports MAC addresses and vendors.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install netdiscover
```
### macOS

```bash
# Not in Homebrew core — use arp-scan on macOS, or run netdiscover from the Kali VM
```
---
## Basic Syntax
```bash
sudo netdiscover [-i <iface>] [-r <range> | -p]
```

- Needs `sudo`/root for raw ARP
- With no `-r`/`-l`/`-p`, it auto-scans common LAN ranges on the detected interface

---
## Active vs Passive

```bash
# ACTIVE: send ARP requests across a range (fast, but noisy)
sudo netdiscover -r 192.168.1.0/24

# PASSIVE: send nothing, just listen for ARP on the wire (stealthy)
sudo netdiscover -p
```

- **Passive mode** is the standout feature — zero packets sent, so it won't trip IDS or alert the network; it only sees hosts that happen to talk

---
## Common Options Reference

| Option | Description |
| ------ | ----------- |
| `-i <device>` | Interface to use (e.g. `eth0`) |
| `-r <range>` | Scan a range: `192.168.1.0/24`, `/16`, `/8` |
| `-l <file>` | Scan ranges listed in a file |
| `-p` | Passive mode — sniff only, send nothing |
| `-m <file>` | Scan a list of known MACs / hostnames |
| `-F <filter>` | Custom pcap filter (default `arp`) |
| `-s <ms>` | Sleep time between ARP requests |
| `-c <n>` | Send each ARP request N times (lossy nets) |
| `-f` | Fast mode — skip some addresses, saves time |
| `-P` | Print results in parseable format, then stop |
| `-L` | Like `-P` but keep listening after active scan |
| `-N` | No header (only with `-P`/`-L`) |
| `-r` + `-f` | Common combo for a quick active sweep |

---
## Practical Examples

### Quick Active Sweep
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -f
```

- `-f` (fast mode) skips low-probability addresses — good default for auto-recon

### Passive Mapping (Stealth)
```bash
sudo netdiscover -i eth0 -p
```

- Leave it running; hosts appear as they send ARP. Nothing leaves your NIC.

### Parseable Output for Scripts
```bash
sudo netdiscover -i eth0 -r 192.168.1.0/24 -P -N
```

- `-P` prints clean rows and exits; `-N` drops the header — pipe into `awk`/`grep`

### Scan a Larger Range
```bash
sudo netdiscover -i eth0 -r 10.0.0.0/16 -f
```

---
## Output Interpretation

```
 Currently scanning: 192.168.1.0/24   |   Screen View: Unique Hosts

 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240
 _____________________________________________________________________________
   IP            At MAC Address      Count     Len   MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.1.1     a4:2b:b0:aa:bb:cc      1       60    TP-LINK TECHNOLOGIES
 192.168.1.10    00:0c:29:11:22:33      1       60    VMware, Inc.
 192.168.1.23    b8:27:eb:44:55:66      2      120    Raspberry Pi Foundation
```

| Column | Meaning |
| ------ | ------- |
| `IP` | Discovered host |
| `At MAC Address` | Host's hardware address |
| `Count` | ARP packets seen from it |
| `Len` | Total bytes |
| `MAC Vendor / Hostname` | Vendor from the MAC prefix (OUI) |

- The TUI refreshes live; press `Ctrl+C` to quit

---
## Tips & Best Practices

1. **Passive mode (`-p`) is the reason to pick netdiscover** over arp-scan — it maps a LAN without sending anything
2. **`-f` fast mode** is a good default for active sweeps; drop it if you suspect hosts are being missed
3. **Layer 2 only** — it can't see past the local router; use [Nmap](./Nmap.md)/[masscan](./masscan.md) for remote subnets
4. **Use `-P -N`** when you want output you can pipe into other tools instead of the interactive TUI
5. **Always set `-i`** on multi-interface boxes so you scan the right segment
6. **Great as a first move** on an unknown network — see who's alive, then port-scan the interesting hosts
7. **Vendor column is quick recon** — spot VMs, Pis, routers, and IoT devices by their MAC OUI

---
## Quick Reference

```bash
# Active sweep, fast mode
sudo netdiscover -i eth0 -r 192.168.1.0/24 -f

# Passive (stealth) — send nothing
sudo netdiscover -i eth0 -p

# Auto-scan common ranges
sudo netdiscover -i eth0

# Parseable, no header
sudo netdiscover -i eth0 -r 192.168.1.0/24 -P -N
```

---
## Related Tools

- **[arp-scan](./arp-scan.md)** - One-shot ARP sweep, cleaner for scripting; netdiscover adds passive mode + live TUI
- **[Nmap](./Nmap.md)** - `nmap -sn` for discovery that also crosses routers
- **[masscan](./masscan.md)** - Port-sweep the hosts netdiscover finds
- **[tcpdump](./tcpdump.md)** - Inspect the raw ARP traffic directly

---
## Resources

- [netdiscover GitHub](https://github.com/netdiscover-scanner/netdiscover)
- [netdiscover Man Page](https://manpages.debian.org/testing/netdiscover/netdiscover.8.en.html)
