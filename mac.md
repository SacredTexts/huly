# Connect Mac to Huly on WSL2

## Context
Huly is self-hosted on a Windows machine running WSL2. The Mac connects to it over the local network via SSH for full development access and via browser for the Huly UI.

## Current Configuration
- **Windows LAN IP:** 192.168.1.222
- **WSL2 IP:** 172.23.233.199 (changes on reboot)
- **WSL2 user:** forest
- **Huly port:** 8087
- **SSH port:** 2222 (Windows) -> 22 (WSL2)
- **Selfhost repo:** ~/huly-selfhost
- **Main repo:** ~/huly

---

## Phase 1: WSL2 / Windows side (run from the Windows machine)

Status: **DONE**

### 1.1 Port forwarding (Huly + SSH)
```powershell
# Huly web UI
netsh interface portproxy add v4tov4 listenport=8087 listenaddress=0.0.0.0 connectport=8087 connectaddress=172.23.233.199

# SSH access
netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=22 connectaddress=172.23.233.199
```

### 1.2 Windows Firewall rules
```powershell
netsh advfirewall firewall add rule name="Huly 8087" dir=in action=allow protocol=TCP localport=8087
netsh advfirewall firewall add rule name="SSH 2222" dir=in action=allow protocol=TCP localport=2222
```

### 1.3 SSH server configured in WSL2
- sshd is running on port 22
- PubkeyAuthentication enabled
- AuthorizedKeysFile at ~/.ssh/authorized_keys

### 1.4 Huly configured
- `HOST_ADDRESS=192.168.1.222:8087` set in ~/huly-selfhost/huly_v7.conf
- Containers restarted with new config

---

## Phase 2: Mac side (run from the Mac using Claude Code CLI)

Status: **TODO** - Claude Code on Mac should execute these steps.

### 2.1 Generate SSH key pair (if not already present)
```bash
# Check for existing key
ls ~/.ssh/id_ed25519.pub 2>/dev/null || ssh-keygen -t ed25519 -C "mac-to-wsl2" -f ~/.ssh/id_ed25519 -N ""
```

### 2.2 Copy public key to WSL2
```bash
# This is the one step that requires the WSL2 user's password
ssh-copy-id -p 2222 forest@192.168.1.222
```

### 2.3 Test SSH connection
```bash
ssh -p 2222 forest@192.168.1.222 "echo 'SSH connection successful'; hostname; uname -a"
```

### 2.4 Configure SSH client for convenience
```bash
# Add to ~/.ssh/config
cat >> ~/.ssh/config << 'EOF'

Host huly
    HostName 192.168.1.222
    Port 2222
    User forest
    IdentityFile ~/.ssh/id_ed25519
    StrictHostKeyChecking accept-new
EOF
```

### 2.5 Verify shortcut works
```bash
ssh huly "echo 'SSH alias working'; docker ps --format '{{.Names}}' | head -5"
```

### 2.6 Connect VS Code to WSL2
1. `Cmd+Shift+P` -> `Remote-SSH: Connect to Host...` -> `huly`
2. VS Code opens a remote window connected to WSL2
3. `File > Open Folder` -> `/home/forest/huly` for the platform code
4. Or `/home/forest/huly-selfhost` for selfhost/docker config

### 2.7 Docker management from VS Code
Install the Docker extension on the remote side (`ms-azuretools.vscode-docker`):
- Docker sidebar (whale icon) shows all Huly containers
- Right-click containers to start/stop/restart/view logs
- Edit `~/huly-selfhost/compose.yml` directly and run `docker compose up -d` from the terminal

### 2.9 Connect Claude Code CLI via SSH
```bash
# Launch Claude Code targeting the remote WSL2 environment
claude --ssh huly
```

---

## Phase 3: Verification checklist

Run these from the Mac to confirm everything works:

```bash
# 1. SSH connects without password prompt
ssh huly "whoami"

# 2. Can access the Huly repo
ssh huly "ls ~/huly/package.json"

# 3. Docker is accessible
ssh huly "docker ps --format '{{.Names}} {{.Status}}' | grep huly"

# 4. Huly UI is reachable from Mac browser
# Open: http://192.168.1.222:8087

# 5. VS Code Remote-SSH connects
# Cmd+Shift+P -> "Remote-SSH: Connect to Host..." -> huly

# 6. Claude Code works over SSH
claude --ssh huly
```

---

## After a Windows reboot

The WSL2 IP changes on reboot. Run these steps on the Windows machine to restore access:

### 1. Get the new WSL2 IP (PowerShell)
```powershell
wsl hostname -I
```

### 2. Update port forwarding (PowerShell as Administrator)
```powershell
# Remove old rules
netsh interface portproxy delete v4tov4 listenport=8087 listenaddress=0.0.0.0
netsh interface portproxy delete v4tov4 listenport=2222 listenaddress=0.0.0.0

# Add new rules with updated WSL2 IP
netsh interface portproxy add v4tov4 listenport=8087 listenaddress=0.0.0.0 connectport=8087 connectaddress=<NEW_WSL2_IP>
netsh interface portproxy add v4tov4 listenport=2222 listenaddress=0.0.0.0 connectport=22 connectaddress=<NEW_WSL2_IP>
```

### 3. Restart SSH in WSL2 (if needed)
```bash
sudo service ssh start
```

### 4. Re-run Huly setup (only if Windows LAN IP changed)
```bash
cd ~/huly-selfhost
./setup.sh
# Enter the Windows LAN IP as the host address
# Enter 8087 as the port
# Say No to SSL
```
