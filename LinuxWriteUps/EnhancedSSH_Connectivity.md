## Advanced SSH Connectivity Techniques

### Introduction
This guide covers advanced SSH techniques that go beyond basic key authentication. These methods will help you manage multiple servers efficiently, create secure tunnels, use jump hosts, and optimize your SSH workflow. This complements the basic SSH key setup and focuses on power-user techniques.

---

## 1. SSH Config File - Simplify Connection Management

The SSH config file (`~/.ssh/config`) allows you to create connection profiles with custom settings, eliminating the need to remember IP addresses, usernames, ports, and key locations.

### Creating Your SSH Config File

```bash
# Create or edit the config file
nano ~/.ssh/config

# Set appropriate permissions
chmod 600 ~/.ssh/config
```

### Basic Config Example

```ssh-config
# Development Server
Host dev
    HostName 192.168.1.100
    User developer
    Port 2222
    IdentityFile ~/.ssh/dev_key

# Production Server
Host prod
    HostName prod.example.com
    User admin
    Port 22
    IdentityFile ~/.ssh/prod_key
    ServerAliveInterval 60
    ServerAliveCountMax 3

# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
    PreferredAuthentications publickey
```

**Now connect simply with:**
```bash
ssh dev
ssh prod
```

### Useful Config Options

| Option | Purpose | Example |
|--------|---------|---------|
| `ServerAliveInterval` | Keep connection alive | `60` (seconds) |
| `ServerAliveCountMax` | Max keepalive attempts | `3` |
| `Compression` | Enable compression | `yes` |
| `ForwardAgent` | Forward SSH agent | `yes` |
| `LogLevel` | Set logging verbosity | `QUIET`, `ERROR`, `INFO` |
| `StrictHostKeyChecking` | Check host keys | `ask`, `yes`, `no` |

---

## 2. Using Modern SSH Key Types (Ed25519)

Ed25519 keys are more secure, faster, and smaller than traditional RSA keys. They're the recommended choice for new key generation.

### Generate Ed25519 Key

```bash
# Generate Ed25519 key (recommended)
ssh-keygen -t ed25519 -C "your_email@example.com"

# For systems that don't support Ed25519, use RSA 4096
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### Key Type Comparison

| Type | Security | Speed | Key Size | Recommendation |
|------|----------|-------|----------|----------------|
| Ed25519 | Excellent | Fast | Small (68 chars) | **Best choice** |
| RSA 4096 | Good | Slower | Large (3072+ chars) | Legacy systems |
| ECDSA | Good | Fast | Medium | Not recommended |

---

## 3. SSH Jump Hosts (Bastion/Proxy Servers)

Access servers behind firewalls or private networks through an intermediate jump host.

### Method 1: ProxyJump (Modern, Recommended)

```bash
# Direct command
ssh -J jumphost@bastion.com user@internal-server

# Using SSH config
Host internal
    HostName 10.0.1.50
    User admin
    ProxyJump jumphost@bastion.com

# Multiple jump hosts
Host deep-server
    HostName 10.0.2.100
    ProxyJump bastion1,bastion2,bastion3
```

### Method 2: ProxyCommand (Legacy, Compatible)

```ssh-config
Host internal-legacy
    HostName 10.0.1.50
    User admin
    ProxyCommand ssh -W %h:%p jumphost@bastion.com
```

---

## 4. SSH Port Forwarding (Tunneling)

Create encrypted tunnels to access remote services securely.

### Local Port Forwarding
Forward local port to remote service (access remote resource locally).

```bash
# Forward local port 8080 to remote server's port 80
ssh -L 8080:localhost:80 user@server

# Access remote database locally
ssh -L 3306:localhost:3306 user@dbserver
# Now connect to localhost:3306 to access remote MySQL

# Forward to third-party host through SSH server
ssh -L 9000:internal-app:8080 user@jumphost
```

**SSH Config Example:**
```ssh-config
Host db-tunnel
    HostName dbserver.com
    User admin
    LocalForward 3306 localhost:3306
    LocalForward 5432 localhost:5432
```

### Remote Port Forwarding
Forward remote port to local service (expose local resource remotely).

```bash
# Make local port 3000 available on remote server port 8080
ssh -R 8080:localhost:3000 user@server

# Share local web server with remote team
ssh -R 9090:localhost:80 user@publicserver
```

### Dynamic Port Forwarding (SOCKS Proxy)
Create a SOCKS proxy for routing all traffic through SSH.

```bash
# Create SOCKS proxy on port 9999
ssh -D 9999 user@server

# Use with curl
curl --socks5 localhost:9999 https://example.com

# Configure browser to use SOCKS5 proxy: localhost:9999
```

**SSH Config Example:**
```ssh-config
Host socks-proxy
    HostName proxy.example.com
    User proxyuser
    DynamicForward 9999
```

---

## 5. SSH Connection Multiplexing (ControlMaster)

Reuse existing SSH connections for faster subsequent connections and reduced overhead.

### Enable Multiplexing in SSH Config

```ssh-config
Host *
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h:%p
    ControlPersist 10m
```

### Create Socket Directory

```bash
mkdir -p ~/.ssh/sockets
chmod 700 ~/.ssh/sockets
```

### Benefits
- First connection: Normal speed
- Subsequent connections: **Instant** (no re-authentication)
- Shares single TCP connection
- Reduces server load

### Check Active Connections

```bash
# List active multiplexed connections
ls -l ~/.ssh/sockets/

# Check specific connection status
ssh -O check user@server

# Close multiplexed connection
ssh -O exit user@server
```

---

## 6. SSH Agent and Agent Forwarding

### Start SSH Agent

```bash
# Start agent
eval "$(ssh-agent -s)"

# Add key to agent
ssh-add ~/.ssh/id_ed25519

# Add key with timeout (8 hours)
ssh-add -t 28800 ~/.ssh/id_ed25519

# List loaded keys
ssh-add -l

# Remove all keys from agent
ssh-add -D
```

### Agent Forwarding
Use local SSH keys on remote servers without copying them.

```bash
# Enable for single connection
ssh -A user@server

# Enable in SSH config
Host trusted-server
    HostName server.com
    ForwardAgent yes
```

**Warning:** Only use agent forwarding on trusted servers, as root users can hijack your agent.

### Auto-start SSH Agent (Add to ~/.bashrc or ~/.zshrc)

```bash
# Auto-start SSH agent
if ! pgrep -u "$USER" ssh-agent > /dev/null; then
    ssh-agent > ~/.ssh-agent-env
fi
if [[ ! "$SSH_AUTH_SOCK" ]]; then
    source ~/.ssh-agent-env > /dev/null
fi
```

---

## 7. SSH Escape Sequences

Press `Enter`, then `~` followed by a command key to control active SSH session.

| Sequence | Action |
|----------|--------|
| `~.` | Disconnect (useful for frozen sessions) |
| `~^Z` | Suspend SSH session (return with `fg`) |
| `~#` | List forwarded connections |
| `~&` | Background SSH (logout when done) |
| `~?` | Show all escape sequences |
| `~~` | Send literal `~` character |

**Example:**
```bash
# Session frozen? Press: Enter ~ .
# This forces disconnect without closing terminal
```

---

## 8. Advanced SSH Techniques

### Execute Remote Commands

```bash
# Run single command
ssh user@server "ls -la /var/log"

# Run multiple commands
ssh user@server "uptime; df -h; free -m"

# Run local script on remote server
ssh user@server 'bash -s' < local_script.sh

# Run command with sudo (using -t for pseudo-terminal)
ssh -t user@server "sudo systemctl restart nginx"
```

### Copy Files While Connected

While in an SSH session, press `Enter` then `~C` to get ssh command prompt:

```
ssh> -L 8080:localhost:80
Forwarding port.
```

### SSH with Different Identity Files

```bash
# Use specific key
ssh -i ~/.ssh/special_key user@server

# Try multiple keys
ssh -i ~/.ssh/key1 -i ~/.ssh/key2 user@server
```

### Disable Strict Host Key Checking (Testing Only)

```bash
# Disable for single connection (not recommended for production)
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null user@server
```

### SSH with Custom Port

```bash
# Command line
ssh -p 2222 user@server

# SSH config
Host custom-port
    HostName server.com
    Port 2222
```

---

## 9. SSH Tunneling for Database Access

### MySQL/MariaDB Tunnel

```bash
# Create tunnel
ssh -L 3306:localhost:3306 user@dbserver

# In another terminal, connect to database
mysql -h 127.0.0.1 -P 3306 -u dbuser -p
```

### PostgreSQL Tunnel

```bash
# Create tunnel
ssh -L 5432:localhost:5432 user@pgserver

# Connect to PostgreSQL
psql -h localhost -p 5432 -U dbuser -d database
```

### MongoDB Tunnel

```bash
# Create tunnel
ssh -L 27017:localhost:27017 user@mongoserver

# Connect with MongoDB client
mongosh --host localhost --port 27017
```

---

## 10. SSH Security Hardening

### Client-Side Security (~/.ssh/config)

```ssh-config
Host *
    # Use only key authentication
    PreferredAuthentications publickey

    # Disable weak algorithms
    KexAlgorithms curve25519-sha256,diffie-hellman-group-exchange-sha256
    Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
    MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com

    # Verify host keys
    StrictHostKeyChecking ask
    HashKnownHosts yes

    # Keep connections alive
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

### Server-Side Hardening (/etc/ssh/sshd_config)

```bash
# Edit SSH server config (requires root)
sudo nano /etc/ssh/sshd_config
```

**Recommended settings:**
```
# Disable root login
PermitRootLogin no

# Disable password authentication
PasswordAuthentication no
PubkeyAuthentication yes

# Allow only specific users
AllowUsers user1 user2

# Change default port (security through obscurity)
Port 2222

# Disable empty passwords
PermitEmptyPasswords no

# Limit authentication attempts
MaxAuthTries 3

# Set login grace time
LoginGraceTime 30s
```

**Restart SSH service:**
```bash
sudo systemctl restart sshd
```

---

## 11. SSH for Cloud Services

### AWS EC2

```ssh-config
Host aws-prod
    HostName ec2-54-123-45-67.compute-1.amazonaws.com
    User ec2-user
    IdentityFile ~/.ssh/aws-prod-key.pem
    ServerAliveInterval 60
```

### Google Cloud Platform

```ssh-config
Host gcp-instance
    HostName 35.123.45.67
    User your-username
    IdentityFile ~/.ssh/google_compute_engine
```

### DigitalOcean Droplet

```ssh-config
Host do-droplet
    HostName 159.123.45.67
    User root
    IdentityFile ~/.ssh/digitalocean_key
```

---

## 12. Useful SSH Commands Reference

```bash
# Test SSH connection without executing commands
ssh -T user@server

# Verbose output (debugging)
ssh -v user@server    # Level 1
ssh -vv user@server   # Level 2
ssh -vvv user@server  # Level 3 (maximum verbosity)

# Dry-run/test config syntax
ssh -G user@server

# Execute command and disconnect
ssh user@server 'command' && exit

# Keep SSH session alive from client
ssh -o ServerAliveInterval=60 user@server

# Connect with X11 forwarding (run GUI apps)
ssh -X user@server
ssh -Y user@server  # Trusted X11 forwarding

# Compress data for slow connections
ssh -C user@server

# Allocate pseudo-terminal (for interactive commands)
ssh -t user@server "sudo command"
```

---

## 13. Troubleshooting Common SSH Issues

### Permission Denied (publickey)

```bash
# Check SSH with verbose output
ssh -vvv user@server

# Common fixes:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/config

# On remote server:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

### Connection Timeout

```bash
# Test if port is open
nc -zv server.com 22

# Try different port
ssh -p 2222 user@server

# Check firewall rules
sudo ufw status
```

### Too Many Authentication Failures

```bash
# Specify exact key to use
ssh -o IdentitiesOnly=yes -i ~/.ssh/specific_key user@server

# Or limit keys in config
Host problem-server
    IdentitiesOnly yes
    IdentityFile ~/.ssh/correct_key
```

### Known Hosts Conflict

```bash
# Remove specific host from known_hosts
ssh-keygen -R server.com

# Or remove specific IP
ssh-keygen -R 192.168.1.100

# Remove line number (e.g., line 15)
sed -i '15d' ~/.ssh/known_hosts
```

---

## 14. SSH Aliases and Shell Functions

Add to `~/.bashrc` or `~/.zshrc`:

```bash
# Quick SSH aliases
alias sshprod='ssh production-server'
alias sshdev='ssh development-server'

# SSH with automatic tunnel
ssh-tunnel() {
    ssh -L ${2:-8080}:localhost:${2:-8080} $1
}
# Usage: ssh-tunnel user@server 3000

# Copy SSH key to server
ssh-copy-key() {
    ssh-copy-id -i ~/.ssh/id_ed25519.pub $1
}
# Usage: ssh-copy-key user@server

# SSH and change to directory
sshcd() {
    ssh -t $1 "cd $2; bash --login"
}
# Usage: sshcd user@server /var/www/html
```

---

## 15. Complete SSH Config Example

```ssh-config
# ~/.ssh/config - Complete example

# Global defaults for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    Compression yes
    ControlMaster auto
    ControlPath ~/.ssh/sockets/%r@%h:%p
    ControlPersist 10m

# Development server
Host dev
    HostName 192.168.1.100
    User developer
    IdentityFile ~/.ssh/dev_key
    LocalForward 3000 localhost:3000
    LocalForward 3306 localhost:3306

# Production server (jump host required)
Host prod
    HostName 10.0.1.50
    User admin
    IdentityFile ~/.ssh/prod_key
    ProxyJump bastion
    StrictHostKeyChecking yes

# Bastion/Jump host
Host bastion
    HostName bastion.company.com
    User jumpuser
    Port 2222
    IdentityFile ~/.ssh/bastion_key

# Database server with SOCKS proxy
Host db-proxy
    HostName db.internal.com
    User dbadmin
    DynamicForward 9999
    LocalForward 5432 localhost:5432

# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
    PreferredAuthentications publickey

# Personal server with multiple tunnels
Host homelab
    HostName homelab.example.com
    User admin
    Port 2222
    IdentityFile ~/.ssh/homelab_key
    LocalForward 8080 localhost:80
    LocalForward 8443 localhost:443
    LocalForward 9091 localhost:9091
    RemoteForward 3000 localhost:3000

# Wildcard for AWS servers
Host aws-*
    User ec2-user
    IdentityFile ~/.ssh/aws-key.pem
    StrictHostKeyChecking no
    UserKnownHostsFile /dev/null

# Specific AWS instance
Host aws-web
    HostName ec2-54-123-45-67.compute-1.amazonaws.com
```

---

## 16. Additional Resources

### Official Documentation
- **OpenSSH Manual**: https://www.openssh.com/manual.html
- **SSH Config Man Page**: `man ssh_config`
- **SSH Man Page**: `man ssh`

### Security Best Practices
- **Mozilla SSH Guidelines**: https://infosec.mozilla.org/guidelines/openssh
- **SSH Audit Tool**: https://github.com/jtesta/ssh-audit

### Tools and Utilities
- **Mosh** (mobile shell): https://mosh.org/ - Better SSH for unstable connections
- **Eternal Terminal**: https://eternalterminal.dev/ - Re-connectable SSH
- **tmux/screen**: Terminal multiplexers for persistent sessions

---

## Summary

This guide covered advanced SSH techniques including:
- SSH config files for simplified management
- Modern Ed25519 keys
- Jump hosts and proxy configurations
- Port forwarding (local, remote, dynamic)
- Connection multiplexing for speed
- SSH agent and forwarding
- Security hardening
- Troubleshooting common issues

Master these techniques to significantly improve your SSH workflow, security, and efficiency when managing remote servers.
