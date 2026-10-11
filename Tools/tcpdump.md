## Command-Line Packet Capture

tcpdump is a CLI packet sniffer and analyzer built on libpcap. In CTFs it's used to capture traffic on an interface, confirm whether a target is actually talking back (reverse shells, ping sweeps, SSRF callbacks), and pull data out of `.pcap` files. It uses the same BPF filter syntax as Wireshark, so filters are portable between the two.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install tcpdump
```
### macOS

```bash
# Pre-installed on macOS — use tcpdump directly
tcpdump --version
```
---
## Basic Syntax
```bash
sudo tcpdump [options] [BPF filter expression]
```

- Needs `sudo`/root to put the interface in promiscuous mode
- Press `Ctrl+C` to stop; it prints a packet count on exit

---
## Picking an Interface

```bash
# List capturable interfaces
sudo tcpdump -D
# or
ip -br addr

# Capture on a specific interface
sudo tcpdump -i eth0

# Capture on ALL interfaces
sudo tcpdump -i any
```

- `tun0` is usually your [OpenVPN](./OpenVPN.md) interface on HTB/THM — capture there to see target traffic

---
## Common Options Reference

| Option | Description |
| ------ | ----------- |
| `-i <iface>` | Interface to capture on (`any` = all) |
| `-n` | Don't resolve hostnames (faster, cleaner) |
| `-nn` | Don't resolve hostnames **or** port names |
| `-v` / `-vv` / `-vvv` | Increasing verbosity (TTL, options, etc.) |
| `-c <n>` | Stop after capturing N packets |
| `-w <file.pcap>` | Write raw packets to file (for Wireshark) |
| `-r <file.pcap>` | Read packets from a saved file |
| `-s <bytes>` | Snap length — bytes per packet (`-s 0` = full packet) |
| `-A` | Print packet payload as ASCII |
| `-X` | Print payload as hex **and** ASCII |
| `-e` | Show link-layer (Ethernet MAC) header |
| `-t` / `-tttt` | Suppress / use human-readable timestamps |
| `-l` | Line-buffer output (needed when piping to grep) |
| `-G <sec>` + `-w` | Rotate capture file every N seconds |

---
## BPF Filter Expressions

Filters go at the end of the command. Combine with `and` / `or` / `not`.

### Host / Network
```bash
sudo tcpdump -n host 10.10.10.5           # to or from this host
sudo tcpdump -n src host 10.10.10.5       # from this host only
sudo tcpdump -n dst host 10.10.10.5       # to this host only
sudo tcpdump -n net 192.168.1.0/24        # whole subnet
```

### Port / Protocol
```bash
sudo tcpdump -n port 80                   # HTTP either direction
sudo tcpdump -n src port 4444             # reverse shell callback
sudo tcpdump -n tcp port 443
sudo tcpdump -n udp port 53               # DNS
sudo tcpdump -n icmp                      # pings
sudo tcpdump -n portrange 8000-9000
```

### Combined
```bash
# HTTP or HTTPS to one host, no DNS noise
sudo tcpdump -n host 10.10.10.5 and \(port 80 or port 443\)

# Everything except SSH (so your own session doesn't flood output)
sudo tcpdump -n not port 22
```

- Escape parentheses with `\(` `\)` or wrap the whole filter in quotes

---
## Practical Examples

### Confirm a Reverse Shell / Callback Is Reaching You
```bash
# Watch your listener port for the victim connecting back
sudo tcpdump -i tun0 -n tcp port 4444
```

- If you see a SYN from the target but no shell, the payload fired but the connection is being dropped

### Watch for an Incoming Ping (SSRF / blind RCE test)
```bash
sudo tcpdump -i tun0 -n icmp
# then trigger: ping -c1 YOUR_IP on the target
```

### Capture to a File for Wireshark
```bash
sudo tcpdump -i eth0 -n -w capture.pcap
# ...reproduce the traffic, then Ctrl+C...
wireshark capture.pcap
```

### Read & Filter an Existing Capture
```bash
# Show only HTTP traffic from a saved file
tcpdump -nr capture.pcap port 80

# Dump payloads as ASCII to grep for creds/flags
tcpdump -nr capture.pcap -A | grep -iE "pass|user|flag|token"
```

### Grab Cleartext Credentials Off the Wire
```bash
sudo tcpdump -i any -nA -s0 port 21 or port 23 or port 80
```

- FTP / Telnet / HTTP Basic send creds in cleartext — `-A` reveals them

### Rotating Capture (long-running)
```bash
sudo tcpdump -i eth0 -n -w cap-%H%M%S.pcap -G 300
```

- New file every 5 minutes; useful when leaving a capture running

---
## Output Interpretation

```
14:02:11.204501 IP 10.10.10.5.52344 > 10.17.72.21.4444: Flags [S], seq 0, win 64240
14:02:11.204600 IP 10.17.72.21.4444 > 10.10.10.5.52344: Flags [S.], seq 0, ack 1
14:02:11.250012 IP 10.10.10.5.52344 > 10.17.72.21.4444: Flags [P.], seq 1:38, length 37
```

| Field | Meaning |
| ----- | ------- |
| `14:02:11.204501` | Timestamp |
| `10.10.10.5.52344 >` | Source IP + port, `>` = direction |
| `10.17.72.21.4444` | Destination IP + port |
| `Flags [S]` | TCP flags: `S`=SYN, `.`=ACK, `P`=PSH, `F`=FIN, `R`=RST |
| `length 37` | Payload bytes |

- `[S]` then `[S.]` then `[.]` = a completed 3-way handshake (the host is talking back)
- Lots of `[S]` with no `[S.]` reply = port filtered/closed or host down

---
## Tips & Best Practices

1. **Always use `-n` (or `-nn`)** — DNS lookups slow capture and clutter output
2. **Exclude your own SSH session** with `not port 22` so your shell traffic doesn't drown the capture
3. **Capture to `.pcap` with `-w`, analyze in Wireshark** — the CLI is for triage, Wireshark for deep inspection
4. **Use `-s0`** when grabbing payloads so packets aren't truncated (`-A`/`-X` need the full packet)
5. **`tun0` is your VPN interface** on HTB/THM — that's where target traffic lives, not `eth0`
6. **Pipe through grep with `-l`** — without line-buffering, grep sees nothing until the buffer flushes
7. **`-c 1` is handy for a one-shot confirmation** (did a single callback arrive?) then it exits

---
## Quick Reference

```bash
# List interfaces
sudo tcpdump -D

# Capture on VPN interface, no DNS
sudo tcpdump -i tun0 -n

# Watch a reverse-shell port
sudo tcpdump -i tun0 -n tcp port 4444

# Watch for incoming pings (SSRF/RCE test)
sudo tcpdump -i tun0 -n icmp

# Capture to file for Wireshark
sudo tcpdump -i eth0 -n -w capture.pcap

# Read a saved capture, show payloads
tcpdump -nr capture.pcap -A

# Grep a capture for creds
tcpdump -nr capture.pcap -A | grep -iE "pass|user|flag"
```

---
## Related Tools

- **[tshark](./tshark.md)** - Wireshark's CLI; richer protocol dissection and field extraction than tcpdump
- **[Nmap](./Nmap.md)** - Active scanning; tcpdump is passive and confirms what the target actually sends back
- **[netcat](./netcat.md)** - Pair with tcpdump to verify reverse-shell callbacks land
- **Wireshark** - GUI analysis of the `.pcap` files tcpdump writes
- **[Airmon-ng / Aircrack-ng](./Airmon-ng.md)** - For wireless captures; tcpdump reads the resulting `.cap` files

---
## Resources

- [tcpdump Official Site](https://www.tcpdump.org/)
- [tcpdump Man Page](https://www.tcpdump.org/manpages/tcpdump.1.html)
- [BPF Filter Reference (pcap-filter)](https://www.tcpdump.org/manpages/pcap-filter.7.html)
