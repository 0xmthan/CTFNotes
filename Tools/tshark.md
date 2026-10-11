## Wireshark on the Command Line

tshark is the terminal version of Wireshark — same capture engine and protocol dissectors, no GUI. It's the strongest tool for pulling specific fields out of traffic: HTTP credentials, DNS queries, exported files, cleartext tokens. In CTFs it shines for carving data out of a provided `.pcap` without clicking through Wireshark.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install tshark
```

- Say **Yes** when asked "Should non-superusers be able to capture packets?" (or run with sudo)
### macOS

```bash
brew install wireshark   # includes tshark
```
---
## Basic Syntax
```bash
tshark [options] [-f "capture filter"] [-Y "display filter"]
```

- **Capture filter** (`-f`): BPF syntax, applied *while capturing* (like tcpdump)
- **Display filter** (`-Y`): Wireshark syntax, applied *after* capture — far more powerful

---
## Capture vs Read

```bash
# Live capture on an interface
sudo tshark -i eth0

# List interfaces
tshark -D

# Read an existing capture (no sudo needed)
tshark -r capture.pcap
```

---
## Common Options Reference

| Option | Description |
| ------ | ----------- |
| `-i <iface>` | Interface to capture on |
| `-r <file>` | Read from a `.pcap`/`.pcapng` file |
| `-w <file>` | Write captured packets to file |
| `-c <n>` | Stop after N packets |
| `-f "<bpf>"` | Capture filter (BPF, like tcpdump) |
| `-Y "<expr>"` | Display filter (Wireshark syntax) |
| `-T fields` | Output only specific fields |
| `-e <field>` | Field to print (repeatable, use with `-T fields`) |
| `-n` | Disable name resolution |
| `-q` | Quiet — suppress per-packet output (use with stats) |
| `-z <stat>` | Run a statistic (e.g. `-z io,phs`) |
| `--export-objects <proto>,<dir>` | Carve out transferred files (http, smb, tftp...) |
| `-x` | Hex + ASCII dump of each packet |

---
## Display Filters (the `-Y` powerhouse)

Wireshark display filters are more expressive than BPF — they understand protocol fields.

```bash
# HTTP requests only
tshark -r cap.pcap -Y "http.request"

# DNS queries
tshark -r cap.pcap -Y "dns.flags.response == 0"

# Traffic to/from a host
tshark -r cap.pcap -Y "ip.addr == 10.10.10.5"

# A specific TCP stream
tshark -r cap.pcap -Y "tcp.stream == 3"

# Only packets containing a string
tshark -r cap.pcap -Y 'frame contains "password"'

# POST requests carrying form data
tshark -r cap.pcap -Y "http.request.method == POST"
```

Common field operators: `==`, `!=`, `contains`, `matches` (regex), `&&`, `||`, `!`.

---
## Field Extraction (`-T fields`)

The killer feature — print just the columns you want, perfect for scripting.

```bash
# Every URL requested in a capture
tshark -r cap.pcap -Y "http.request" -T fields -e http.host -e http.request.uri

# All DNS query names
tshark -r cap.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name

# Source IP, dest IP, dest port for every SYN
tshark -r cap.pcap -Y "tcp.flags.syn==1 && tcp.flags.ack==0" \
  -T fields -e ip.src -e ip.dst -e tcp.dstport
```

Add `-E header=y` for column headers, `-E separator=,` for CSV output.

---
## Practical Examples

### Pull HTTP Basic-Auth / Form Credentials
```bash
# Basic Auth header (base64 — decode after)
tshark -r cap.pcap -Y "http.authorization" -T fields -e http.authorization

# POSTed form bodies (logins)
tshark -r cap.pcap -Y "http.request.method == POST" \
  -T fields -e http.file_data
```

### Follow a TCP Stream (reconstruct a conversation)
```bash
tshark -r cap.pcap -q -z follow,tcp,ascii,3
```

- `3` is the stream index — find it with `-Y "tcp.stream eq 3"`

### Extract Transferred Files from a Capture
```bash
mkdir loot
tshark -r cap.pcap --export-objects http,loot
ls loot/     # recovered files land here
```

- Works for `http`, `smb`, `tftp`, `imf` (email) — great for "find the hidden file" challenges

### List Every Protocol in the Capture
```bash
tshark -r cap.pcap -q -z io,phs
```

- Fast way to see what's in an unfamiliar `.pcap` (is there FTP? SMB? ICMP tunneling?)

### Grep DNS for Exfil / Beaconing
```bash
tshark -r cap.pcap -Y "dns" -T fields -e dns.qry.name | sort | uniq -c | sort -rn
```

- Long/random subdomains repeated = likely DNS exfiltration

### Live Capture Just HTTP
```bash
sudo tshark -i eth0 -f "tcp port 80" -Y "http.request" \
  -T fields -e http.host -e http.request.uri
```

---
## Output Interpretation

```
  12 3.204 10.10.10.5 → 10.17.72.21 HTTP GET /admin/login.php HTTP/1.1
  14 3.251 10.17.72.21 → 10.10.10.5 HTTP HTTP/1.1 200 OK (text/html)
```

| Field | Meaning |
| ----- | ------- |
| `12` | Packet number in the capture |
| `3.204` | Seconds since start of capture |
| `10.10.10.5 →` | Source → destination |
| `HTTP` | Highest-layer protocol dissected |
| `GET /admin/login.php` | Decoded protocol summary |

---
## Tips & Best Practices

1. **Prefer `-Y` display filters over `-f`** for analysis — they understand protocol fields, BPF doesn't
2. **`--export-objects http,dir`** is the fastest way to recover files hidden in a capture
3. **`-z io,phs` first** on any unknown `.pcap` — tells you what protocols are present before you dig
4. **Use `-T fields -e ...`** to turn packets into greppable/CSV columns for scripting
5. **`-q -z follow,tcp,ascii,N`** reconstructs a full conversation — equivalent to Wireshark's "Follow TCP Stream"
6. **Base64-decode `http.authorization`** values — Basic Auth is just `base64(user:pass)`
7. **tshark reads tcpdump captures** and vice-versa — capture quickly with [tcpdump](./tcpdump.md), analyze deeply with tshark

---
## Quick Reference

```bash
# Read a capture
tshark -r cap.pcap

# All HTTP URLs
tshark -r cap.pcap -Y "http.request" -T fields -e http.host -e http.request.uri

# All DNS queries
tshark -r cap.pcap -Y "dns" -T fields -e dns.qry.name

# Credentials in POST bodies
tshark -r cap.pcap -Y "http.request.method==POST" -T fields -e http.file_data

# Follow TCP stream #3
tshark -r cap.pcap -q -z follow,tcp,ascii,3

# Carve out transferred files
tshark -r cap.pcap --export-objects http,loot

# Protocol breakdown of the whole capture
tshark -r cap.pcap -q -z io,phs
```

---
## Related Tools

- **[tcpdump](./tcpdump.md)** - Lighter CLI sniffer; capture fast with it, analyze deeply with tshark
- **Wireshark** - The GUI; same engine, better for visual stream-following and manual inspection
- **[Nmap](./Nmap.md)** - Active recon to pair with passive capture
- **[Hashcat](./Hashcat.md)** / **[John](./John.md)** - Crack hashes/creds carved out of a capture
- **NetworkMiner** - Alternative pcap carver with a focus on extracted artifacts

---
## Resources

- [tshark Man Page](https://www.wireshark.org/docs/man-pages/tshark.html)
- [Wireshark Display Filter Reference](https://www.wireshark.org/docs/dfref/)
- [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html_chunked/)
