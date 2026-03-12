## Virtual Private Network

OpenVPN is an open-source VPN protocol that creates encrypted tunnels for secure communication over untrusted networks. It uses SSL/TLS for key exchange and supports symmetric encryption for data.

---
## Basic Syntax
```bash
sudo openvpn .ovpn
```

---
### Connect to a VPN
```bash
sudo openvpn /path/to/config.ovpn
```

- Requires `.ovpn` configuration file
- Runs in foreground (shows connection logs)
- Press `Ctrl+C` to disconnect
### Connect in Background
```bash
sudo openvpn /path/to/config.ovpn --daemon --log-append /var/log/openvpn.log
```

- `--daemon`: Run as background process
- `--log-append`: Save logs to file
### List Active Connections
```bash
sudo systemctl status openvpn@config-name
```
### Stop OpenVPN
```bash
sudo systemctl stop openvpn@config-name
```

---
## Configuration Files
### File Types

| Format  | Description                |
| ------- | -------------------------- |
| `.ovpn` | OpenVPN configuration file |
| `.conf` | Alternative config format  |
| `.key`  | Private key file           |
| `.crt`  | Certificate file           |
### Configuration Location
```bash
# Linux
~/.config/openvpn/
/etc/openvpn/

# macOS
~/Library/openvpn/
```

---
### Logging & Debugging

|Option|Description|
|---|---|
|`verb 3`|Verbosity level (0-11)|
|`log file.log`|Log to file|
|`log-append file.log`|Append to log|
|`mute 10`|Suppress repeated messages|

---
## Practical Examples
### Connect with Custom Log
```bash
sudo openvpn ~/openvpn-configs/client.ovpn --log-append ~/openvpn.log
```

### Connect to Specific Server
```bash
sudo openvpn ~/openvpn-configs/client.ovpn --remote vpn.example.com 1194
```

### Override Cipher
```bash
sudo openvpn ~/openvpn-configs/client.ovpn --cipher AES-128-CBC
```

### Run as Systemd Service (Linux)
```bash

sudo cp client.ovpn /etc/openvpn/client.conf

sudo systemctl start openvpn@client
sudo systemctl enable openvpn@client
sudo systemctl status openvpn@client
```

### Disconnect from Daemon Process
```bash
sudo systemctl stop openvpn@client
```

---

## Troubleshooting
### Check IP Address
```bash
curl ifconfig.me

```
### Common Errors

| Error                   | Solution                                             |
| ----------------------- | ---------------------------------------------------- |
| `TLS Error`             | Check certificates are valid and paths correct       |
| `Connection timed out`  | Verify VPN server is online and firewall allows port |
| `Cannot open config`    | Check file path and permissions                      |
| `Authentication failed` | Verify credentials in config or certificate files    |

---

## Security Best Practices

1. **Protect private keys**
    ```bash
    chmod 600 client.key
    ```
    
2. **Use strong ciphers**
    ```
    cipher AES-256-CBC
    auth SHA256
    tls-version-min 1.2
    ```
    
3. **Keep config files secure**
    ```bash
    chmod 600 client.ovpn
    ```
    
4. **Enable kill switch** (disconnect if VPN drops)
    ```
    down /etc/openvpn/update-resolv-conf
    script-security 2
    ```
    
5. **Verify certificates** before connecting    
    ```bash
    openssl x509 -in ca.crt -text -noout
    ```

---
## VPN Types by Use Case

|Use Case|Protocol|Cipher|
|---|---|---|
|General Privacy|UDP|AES-256-CBC|
|Stability (Mobile)|TCP|AES-256-CBC|
|Performance|UDP|AES-128-CBC|
|Maximum Security|TCP|AES-256-GCM|

---
## Port Configuration
### Common VPN Ports

|Port|Protocol|Use Case|
|---|---|---|
|1194|UDP/TCP|Default OpenVPN|
|443|TCP|Blend in with HTTPS|
|80|TCP|Blend in with HTTP|
|8080|TCP|Alternative|

```bash
sudo openvpn /path/to/config.ovpn --remote vpn.example.com 443
```

---

## Tips & Best Practices

1. **Use UDP for speed**, TCP for reliability
2. **Keep logs** for debugging connection issues
3. **Rotate certificates** regularly in production
4. **Monitor bandwidth** to detect unusual activity
5. **Test for DNS leaks** after connecting:
    ```bash
    nslookup example.com
    ```

---
## Quick Reference Commands
```bash
# Start VPN
sudo openvpn /path/to/config.ovpn

# Start in background
sudo openvpn /path/to/config.ovpn --daemon

# Check if running
ps aux | grep openvpn

# View logs
tail -f /var/log/openvpn.log

# Stop VPN (in another terminal)
sudo killall openvpn

# Verify connection
curl ifconfig.me
```

---
## Resources
- [Official OpenVPN Documentation](https://openvpn.net/community-resources/)
- [VPN Security Best Practices](https://openvpn.net/security-overview/)
- [Configuration Reference](https://openvpn.net/man/openvpn.html)