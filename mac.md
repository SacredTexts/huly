# Connect Mac to Huly on WSL2

## Context
Huly is self-hosted on a Windows machine running WSL2. The Mac needs to connect to it over the local network.

## Steps to run on the Windows machine (PowerShell as Administrator)

### 1. Get the Windows LAN IP
```powershell
ipconfig
# Note the IPv4 address of your LAN adapter (e.g., 192.168.1.x)
```

### 2. Get the WSL2 IP
```powershell
wsl hostname -I
# e.g., 172.23.233.199
```

### 3. Set up port forwarding from Windows to WSL2
```powershell
netsh interface portproxy add v4tov4 listenport=8087 listenaddress=0.0.0.0 connectport=8087 connectaddress=<WSL2_IP>
```

### 4. Allow port 8087 through Windows Firewall
```powershell
netsh advfirewall firewall add rule name="Huly 8087" dir=in action=allow protocol=TCP localport=8087
```

### 5. Re-run Huly setup in WSL2 with the Windows LAN IP
```bash
cd ~/huly-selfhost
./setup.huly
# Enter the Windows LAN IP (e.g., 192.168.1.100) as the host address
# Enter 8087 as the port
# Say No to SSL
```

## From the Mac

Open in browser: `http://<WINDOWS_LAN_IP>:8087`

## Notes
- The WSL2 IP changes on reboot. You'll need to update the port forwarding rule each time.
- To automate this, you can add a startup script on Windows that re-creates the portproxy rule.
- Make sure both machines are on the same network.
