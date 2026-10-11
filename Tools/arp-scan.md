## Layer-2 Host Discovery

arp-scan discovers live hosts on the **local network segment** by sending ARP requests and listening for replies. Because ARP operates at layer 2, it finds hosts even when they're configured to ignore ICMP (ping) — making it more reliable than a ping sweep for mapping a LAN. It also reports each host's MAC address and vendor.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install arp-scan
```
### macOS

```bash
brew install arp-scan
```
### Verify
```bash
arp-scan --version
```
---
## Basic Syntax
```bash
sudo arp-scan [options] <targets | --localnet>
```

- Needs `sudo`/root to send raw ARP frames
- Only works on the **local broadcast domain** — ARP doesn't cross routers, so it can't scan remote subnets

---
## Common Options Reference

| Option | Description |
| ------ | ----------- |
| `--localnet` / `-l` | Scan the whole subnet of your interface (from its IP + netmask) |
| `-I <iface>` / `--interface` | Interface to use (e.g. `eth0`, `wlan0`) |
| `--file=<f>` / `-f` | Read target IPs/hostnames from a file |
| `-g` | Don't display duplicate packets |
| `-t <ms>` | Timeout per host (default 500ms) |
| `-r <n>` / `--retry` | Retry count per host (default 2) |
| `-B <rate>` / `--bandwidth` | Output bandwidth / send rate control |
| `--vendor=<file>` | Use a custom MAC-to-vendor mapping file |
| `-q` / `--quiet` | Minimal output |
| `-x` / `--plain` | No header/footer — just results (scriptable) |

---
## Practical Examples

### Scan Your Whole Local Subnet
```bash
sudo arp-scan --localnet
```

- Easiest command — auto-detects your subnet from the interface

### Scan on a Specific Interface
```bash
sudo arp-scan -I eth0 --localnet
```

- Needed when you have multiple interfaces (e.g. `eth0` + `wlan0` + `tun0`)

### Scan an Explicit Range
```bash
sudo arp-scan 192.168.1.0/24
```

### Script-Friendly Output (IPs only)
```bash
sudo arp-scan --localnet -x | awk '{print $1}'
```

### Identify Devices by Vendor
```bash
sudo arp-scan --localnet | grep -iE "raspberry|cisco|vmware|apple"
```

- The MAC vendor often reveals what a host is (router, printer, IoT, VM)

---
## Output Interpretation

```
Interface: eth0, type: EN10MB, MAC: de:84:54:82:39:cb, IPv4: 192.168.1.50
Starting arp-scan 1.10.0 with 256 hosts
192.168.1.1     a4:2b:b0:aa:bb:cc       TP-LINK TECHNOLOGIES CO.,LTD.
192.168.1.10    00:0c:29:11:22:33       VMware, Inc.
192.168.1.23    b8:27:eb:44:55:66       Raspberry Pi Foundation

3 packets received by filter, 0 packets dropped
Ending arp-scan: 256 hosts scanned ... 3 responded
```

| Field | Meaning |
| ----- | ------- |
| `192.168.1.1` | Live host IP |
| `a4:2b:b0:...` | That host's MAC address |
| `TP-LINK ...` | Vendor looked up from the MAC prefix (OUI) |
| `3 responded` | Number of live hosts found |

---
## Tips & Best Practices

1. **Use arp-scan when ping sweeps fail** — hosts can drop ICMP but *must* answer ARP to be on the LAN
2. **`--localnet` is the go-to** — no need to figure out the CIDR yourself
3. **Always pass `-I`** if you have more than one interface, or it may scan the wrong segment
4. **Can't cross routers** — it only sees your own broadcast domain; use [Nmap](./Nmap.md)/[masscan](./masscan.md) for remote subnets
5. **MAC vendor = free recon** — spot VMs (`VMware`/`VirtualBox`), Pis (`b8:27:eb`), and network gear at a glance
6. **Pair with a full port scan** — arp-scan tells you *what's alive*, then scan those IPs for open ports
7. **On the OrbStack Kali box**, scan `eth0` to enumerate the OrbStack virtual network

---
## Quick Reference

```bash
# Scan local subnet (auto-detect)
sudo arp-scan --localnet

# Specific interface
sudo arp-scan -I eth0 --localnet

# Explicit range
sudo arp-scan 192.168.1.0/24

# Just the live IPs
sudo arp-scan --localnet -x | awk '{print $1}'

# Find a vendor
sudo arp-scan --localnet | grep -i raspberry
```

---
## Related Tools

- **[netdiscover](./netdiscover.md)** - Similar ARP-based discovery with passive mode and a live TUI
- **[Nmap](./Nmap.md)** - `nmap -sn` does ARP discovery too, and scans remote subnets arp-scan can't reach
- **[masscan](./masscan.md)** - After finding hosts, sweep them for open ports fast
- **[tcpdump](./tcpdump.md)** - Watch the ARP traffic itself for troubleshooting

---
## Resources

- [arp-scan GitHub](https://github.com/royhills/arp-scan)
- [arp-scan Man Page](https://linux.die.net/man/1/arp-scan)
