# Connect Mac to Huly on WSL2

## Context
Huly is self-hosted on a Windows machine running WSL2. The Mac connects to it over the local network.

## Current Configuration
- **Windows LAN IP:** 192.168.1.222
- **WSL2 IP:** 172.23.233.199
- **Huly port:** 8087
- **Selfhost repo:** ~/huly-selfhost

## WSL2 / Windows side (already done)

The following steps have been completed:

1. Port forwarding from Windows to WSL2:
   ```powershell
   netsh interface portproxy add v4tov4 listenport=8087 listenaddress=0.0.0.0 connectport=8087 connectaddress=172.23.233.199
   ```

2. Windows Firewall rule for port 8087:
   ```powershell
   netsh advfirewall firewall add rule name="Huly 8087" dir=in action=allow protocol=TCP localport=8087
   ```

3. Huly configured with `HOST_ADDRESS=192.168.1.222:8087` and restarted.

## From the Mac

Open in browser: `http://192.168.1.222:8087`

## After a reboot

The WSL2 IP changes on reboot. Run these steps to restore access:

### 1. Get the new WSL2 IP (PowerShell)
```powershell
wsl hostname -I
```

### 2. Update port forwarding (PowerShell as Administrator)
```powershell
netsh interface portproxy delete v4tov4 listenport=8087 listenaddress=0.0.0.0
netsh interface portproxy add v4tov4 listenport=8087 listenaddress=0.0.0.0 connectport=8087 connectaddress=<NEW_WSL2_IP>
```

### 3. Re-run Huly setup in WSL2 (only if Windows LAN IP changed)
```bash
cd ~/huly-selfhost
./setup.sh
# Enter the Windows LAN IP as the host address
# Enter 8087 as the port
# Say No to SSL
```
