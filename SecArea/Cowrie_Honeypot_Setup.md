## Setting Up Cowrie Honeypot with Logging and Monitoring

### Introduction

Cowrie is a medium to high interaction SSH and Telnet honeypot designed to log brute force attacks and the shell interaction performed by attackers. It's an excellent tool for understanding attack patterns, collecting malware samples, and researching attacker behavior in a controlled environment.

**What Cowrie Does:**
- Emulates a vulnerable SSH/Telnet server
- Logs all authentication attempts (username/password combinations)
- Records full shell interaction and commands
- Captures uploaded/downloaded files
- Collects malware samples
- Provides detailed attacker behavior analytics

**Use Cases:**
- Threat intelligence gathering
- Attack pattern analysis
- Security research
- Early warning system for attacks
- Training and education

---

## 1. Prerequisites

### System Requirements

```bash
# Minimum requirements
- Ubuntu 20.04/22.04 or Debian 11/12
- 1GB RAM (2GB+ recommended)
- 10GB disk space (20GB+ for long-term logging)
- Python 3.9+
- Internet connection

# Recommended setup
- Dedicated VM or VPS (isolated from production)
- Public IP address (for attracting attackers)
- Firewall configured properly
```

### Initial System Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install required dependencies
sudo apt install -y \
    git \
    python3 \
    python3-pip \
    python3-venv \
    python3-dev \
    libssl-dev \
    libffi-dev \
    build-essential \
    libpython3-dev \
    python3-minimal \
    authbind \
    virtualenv

# Install additional tools for monitoring
sudo apt install -y \
    rsyslog \
    logrotate \
    jq \
    curl \
    wget
```

---

## 2. Installing Cowrie

### Method 1: Standard Installation

```bash
# Create dedicated user (security best practice)
sudo adduser --disabled-password cowrie
sudo su - cowrie

# Clone Cowrie repository
cd ~
git clone https://github.com/cowrie/cowrie.git
cd cowrie

# Create virtual environment
python3 -m venv cowrie-env
source cowrie-env/bin/activate

# Install Python dependencies
pip install --upgrade pip
pip install -r requirements.txt
```

### Method 2: Docker Installation (Recommended for Quick Setup)

```bash
# Pull official Cowrie image
docker pull cowrie/cowrie

# Create directories for persistent data
mkdir -p ~/cowrie-data/{etc,var}

# Run Cowrie container
docker run -d \
    --name cowrie \
    -p 2222:2222 \
    -p 2223:2223 \
    -v ~/cowrie-data/etc:/cowrie/cowrie-git/etc \
    -v ~/cowrie-data/var:/cowrie/cowrie-git/var \
    cowrie/cowrie
```

---

## 3. Basic Configuration

### Configure Cowrie Settings

```bash
# Copy default configuration
cd ~/cowrie
cp etc/cowrie.cfg.dist etc/cowrie.cfg

# Edit configuration
nano etc/cowrie.cfg
```

**Essential Configuration (`etc/cowrie.cfg`):**

```ini
[honeypot]
# Hostname to display
hostname = server01

# SSH version to advertise
ssh_version_string = SSH-2.0_OpenSSH_8.9p1 Ubuntu-3ubuntu0.1

# Telnet prompt
telnet_prompt = ubuntu login:

# Enable/disable services
ssh_enabled = true
telnet_enabled = true

# Ports (must be different from real SSH)
listen_endpoints = tcp:2222:interface=0.0.0.0
telnet_endpoints = tcp:2223:interface=0.0.0.0

# Logging verbosity
log_path = var/log/cowrie
download_path = var/lib/cowrie/downloads

# Enable/disable features
backend = shell
auth_class = UserDB
interact_enabled = true
sftp_enabled = true
forward_redirect = false

[ssh]
# SSH banner
version = SSH-2.0_OpenSSH_8.9p1 Ubuntu-3ubuntu0.1
enabled = true

# Key file location
rsa_public_key = etc/ssh_host_rsa_key.pub
rsa_private_key = etc/ssh_host_rsa_key
dsa_public_key = etc/ssh_host_dsa_key.pub
dsa_private_key = etc/ssh_host_dsa_key

[telnet]
enabled = true

# Output formats
[output_jsonlog]
logfile = var/log/cowrie/cowrie.json
epoch_timestamp = false

[output_textlog]
logfile = var/log/cowrie/cowrie.log
```

### Configure User Database

```bash
# Edit userdb to add fake credentials
nano etc/userdb.txt
```

**Example `userdb.txt`:**

```
# Format: username:x:password
# x = accept any password
# * = reject login
# ! = fake root access

root:x:*
admin:x:admin
admin:x:123456
admin:x:password
admin:x:admin123
ubuntu:x:ubuntu
user:x:user
test:x:test
pi:x:raspberry
postgres:x:postgres

# You can add thousands of common credentials
```

---

## 4. Port Forwarding Configuration

To listen on standard ports (22, 23) without running as root, use authbind:

### Setup Authbind

```bash
# Create authbind configuration
sudo touch /etc/authbind/byport/22
sudo touch /etc/authbind/byport/23
sudo chmod 777 /etc/authbind/byport/22
sudo chmod 777 /etc/authbind/byport/23

# Update Cowrie config to listen on standard ports
nano etc/cowrie.cfg
```

```ini
listen_endpoints = tcp:22:interface=0.0.0.0
telnet_endpoints = tcp:23:interface=0.0.0.0
```

### Alternative: IPTables Port Forwarding

```bash
# Forward port 22 to 2222 (Cowrie SSH)
sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 2222

# Forward port 23 to 2223 (Cowrie Telnet)
sudo iptables -t nat -A PREROUTING -p tcp --dport 23 -j REDIRECT --to-port 2223

# Save iptables rules
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

**Important:** Make sure you have alternative SSH access (different port) before applying these rules!

```bash
# FIRST: Configure real SSH on different port
sudo nano /etc/ssh/sshd_config
# Change: Port 22 → Port 2022

sudo systemctl restart sshd

# Verify you can connect on new port before continuing
ssh -p 2022 user@your-server
```

---

## 5. Running Cowrie

### Start Cowrie

```bash
# As cowrie user
cd ~/cowrie
source cowrie-env/bin/activate

# Start with authbind (if using ports 22/23)
authbind --deep bin/cowrie start

# Or start normally (if using high ports)
bin/cowrie start

# Check status
bin/cowrie status

# View logs in real-time
tail -f var/log/cowrie/cowrie.log
```

### Enable Systemd Service

Create `/etc/systemd/system/cowrie.service`:

```ini
[Unit]
Description=Cowrie SSH/Telnet Honeypot
After=network.target

[Service]
Type=forking
User=cowrie
Group=cowrie
WorkingDirectory=/home/cowrie/cowrie
ExecStart=/usr/bin/authbind --deep /home/cowrie/cowrie/bin/cowrie start
ExecStop=/home/cowrie/cowrie/bin/cowrie stop
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable cowrie
sudo systemctl start cowrie
sudo systemctl status cowrie
```

---

## 6. Logging Configuration

### JSON Logging (Recommended)

Enable comprehensive JSON logging in `etc/cowrie.cfg`:

```ini
[output_jsonlog]
enabled = true
logfile = var/log/cowrie/cowrie.json
epoch_timestamp = false
```

### Syslog Integration

```ini
[output_syslog]
enabled = true
facility = LOCAL0
format = text
```

Configure rsyslog to capture Cowrie logs:

```bash
sudo nano /etc/rsyslog.d/cowrie.conf
```

```
# Cowrie honeypot logs
local0.*    /var/log/cowrie-syslog.log
```

Restart rsyslog:

```bash
sudo systemctl restart rsyslog
```

### Database Logging (MySQL/PostgreSQL)

**For MySQL:**

```bash
# Install MySQL
sudo apt install mysql-server python3-mysqldb

# Create database
sudo mysql
```

```sql
CREATE DATABASE cowrie;
CREATE USER 'cowrie'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON cowrie.* TO 'cowrie'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

**Configure in `cowrie.cfg`:**

```ini
[output_mysql]
enabled = true
host = localhost
database = cowrie
username = cowrie
password = secure_password
port = 3306
```

**Initialize database:**

```bash
cd ~/cowrie
mysql -u cowrie -p cowrie < docs/sql/mysql.sql
```

### Log Rotation

Create `/etc/logrotate.d/cowrie`:

```
/home/cowrie/cowrie/var/log/cowrie/*.log /home/cowrie/cowrie/var/log/cowrie/*.json {
    daily
    rotate 30
    compress
    delaycompress
    notifempty
    missingok
    create 0644 cowrie cowrie
    postrotate
        /home/cowrie/cowrie/bin/cowrie restart > /dev/null
    endscript
}
```

---

## 7. Monitoring Solutions

### Option 1: Real-time Monitoring with Cowrie's Built-in Tools

**Monitor live attacks:**

```bash
# Watch JSON logs
tail -f var/log/cowrie/cowrie.json | jq '.'

# Watch text logs
tail -f var/log/cowrie/cowrie.log

# Filter for login attempts
tail -f var/log/cowrie/cowrie.json | jq 'select(.eventid=="cowrie.login.failed")'

# Filter for successful logins
tail -f var/log/cowrie/cowrie.json | jq 'select(.eventid=="cowrie.login.success")'

# Monitor executed commands
tail -f var/log/cowrie/cowrie.json | jq 'select(.eventid=="cowrie.command.input")'
```

### Option 2: ELK Stack Integration (Elasticsearch, Logstash, Kibana)

**Install Filebeat:**

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update && sudo apt install filebeat
```

**Configure Filebeat (`/etc/filebeat/filebeat.yml`):**

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /home/cowrie/cowrie/var/log/cowrie/cowrie.json
  json.keys_under_root: true
  json.add_error_key: true

output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "cowrie-%{+yyyy.MM.dd}"

setup.template.name: "cowrie"
setup.template.pattern: "cowrie-*"
```

**Start Filebeat:**

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

### Option 3: Splunk Integration

Configure Splunk forwarder in `cowrie.cfg`:

```ini
[output_splunk]
enabled = true
host = splunk-server.example.com
port = 8088
token = YOUR-HEC-TOKEN
index = cowrie
sourcetype = cowrie_json
source = cowrie-honeypot
```

### Option 4: Custom Monitoring Dashboard

**Create monitoring script (`monitor.py`):**

```python
#!/usr/bin/env python3

import json
import sys
from collections import defaultdict
from datetime import datetime

def analyze_logs(logfile):
    stats = {
        'total_attempts': 0,
        'failed_logins': 0,
        'successful_logins': 0,
        'commands_executed': 0,
        'unique_ips': set(),
        'usernames': defaultdict(int),
        'passwords': defaultdict(int),
        'commands': defaultdict(int),
        'malware_downloads': 0
    }

    try:
        with open(logfile, 'r') as f:
            for line in f:
                try:
                    entry = json.loads(line)
                    eventid = entry.get('eventid', '')

                    if 'src_ip' in entry:
                        stats['unique_ips'].add(entry['src_ip'])

                    if eventid == 'cowrie.login.failed':
                        stats['failed_logins'] += 1
                        stats['usernames'][entry.get('username', 'unknown')] += 1
                        stats['passwords'][entry.get('password', 'unknown')] += 1

                    elif eventid == 'cowrie.login.success':
                        stats['successful_logins'] += 1

                    elif eventid == 'cowrie.command.input':
                        stats['commands_executed'] += 1
                        stats['commands'][entry.get('input', 'unknown')] += 1

                    elif eventid == 'cowrie.session.file_download':
                        stats['malware_downloads'] += 1

                except json.JSONDecodeError:
                    continue

    except FileNotFoundError:
        print(f"Error: Log file {logfile} not found")
        sys.exit(1)

    return stats

def print_report(stats):
    print("\n" + "="*60)
    print("COWRIE HONEYPOT ACTIVITY REPORT")
    print("="*60)
    print(f"\nTimestamp: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print(f"\nTotal Failed Logins: {stats['failed_logins']}")
    print(f"Total Successful Logins: {stats['successful_logins']}")
    print(f"Commands Executed: {stats['commands_executed']}")
    print(f"Unique IP Addresses: {len(stats['unique_ips'])}")
    print(f"Malware Downloads: {stats['malware_downloads']}")

    print("\n--- Top 10 Usernames ---")
    for username, count in sorted(stats['usernames'].items(),
                                  key=lambda x: x[1],
                                  reverse=True)[:10]:
        print(f"  {username}: {count}")

    print("\n--- Top 10 Passwords ---")
    for password, count in sorted(stats['passwords'].items(),
                                  key=lambda x: x[1],
                                  reverse=True)[:10]:
        print(f"  {password}: {count}")

    print("\n--- Top 10 Commands ---")
    for command, count in sorted(stats['commands'].items(),
                                 key=lambda x: x[1],
                                 reverse=True)[:10]:
        print(f"  {command}: {count}")

    print("\n--- Recent Attacker IPs ---")
    for ip in list(stats['unique_ips'])[-10:]:
        print(f"  {ip}")

    print("\n" + "="*60 + "\n")

if __name__ == '__main__':
    logfile = '/home/cowrie/cowrie/var/log/cowrie/cowrie.json'
    if len(sys.argv) > 1:
        logfile = sys.argv[1]

    stats = analyze_logs(logfile)
    print_report(stats)
```

Make executable and run:

```bash
chmod +x monitor.py
./monitor.py
```

### Option 5: Email Alerts

Create alert script (`alert.sh`):

```bash
#!/bin/bash

LOGFILE="/home/cowrie/cowrie/var/log/cowrie/cowrie.json"
EMAIL="admin@example.com"
THRESHOLD=50

# Count failed logins in last hour
COUNT=$(tail -n 1000 "$LOGFILE" | jq -r 'select(.eventid=="cowrie.login.failed")' | wc -l)

if [ "$COUNT" -gt "$THRESHOLD" ]; then
    SUBJECT="Cowrie Alert: High Attack Activity"
    MESSAGE="Warning: $COUNT failed login attempts detected in the last hour"

    echo "$MESSAGE" | mail -s "$SUBJECT" "$EMAIL"
fi
```

Add to crontab:

```bash
crontab -e
```

```
# Check every hour
0 * * * * /home/cowrie/alert.sh
```

---

## 8. Analyzing Captured Data

### View Downloaded Files

```bash
# List all downloads
ls -lh ~/cowrie/var/lib/cowrie/downloads/

# Analyze with VirusTotal
curl --request POST \
  --url https://www.virustotal.com/api/v3/files \
  --header 'x-apikey: YOUR_API_KEY' \
  --form file=@/path/to/downloaded/file
```

### Extract Statistics

```bash
# Most common usernames
jq -r 'select(.eventid=="cowrie.login.failed") | .username' \
    var/log/cowrie/cowrie.json | sort | uniq -c | sort -nr | head -20

# Most common passwords
jq -r 'select(.eventid=="cowrie.login.failed") | .password' \
    var/log/cowrie/cowrie.json | sort | uniq -c | sort -nr | head -20

# Attacker IP addresses
jq -r '.src_ip' var/log/cowrie/cowrie.json | sort -u

# Commands executed
jq -r 'select(.eventid=="cowrie.command.input") | .input' \
    var/log/cowrie/cowrie.json | sort | uniq -c | sort -nr
```

### Generate Geographic Report

```bash
# Install geoip tools
sudo apt install geoip-bin geoip-database

# Get countries of attackers
jq -r '.src_ip' var/log/cowrie/cowrie.json | sort -u | \
    while read ip; do
        country=$(geoiplookup "$ip" | cut -d: -f2 | cut -d, -f1)
        echo "$country"
    done | sort | uniq -c | sort -nr
```

---

## 9. Integration with Threat Intelligence

### Send Data to AbuseIPDB

```bash
# Report malicious IPs to AbuseIPDB
report_ip() {
    IP=$1
    curl -X POST https://api.abuseipdb.com/api/v2/report \
        -H "Key: YOUR_API_KEY" \
        -H "Accept: application/json" \
        -d "ip=$IP" \
        -d "categories=18,22" \
        -d "comment=SSH honeypot attack"
}

# Extract and report IPs
jq -r '.src_ip' var/log/cowrie/cowrie.json | sort -u | \
    while read ip; do
        report_ip "$ip"
    done
```

### Integration with MISP (Malware Information Sharing Platform)

Configure in `cowrie.cfg`:

```ini
[output_misp]
enabled = true
url = https://misp.example.com
apikey = YOUR_MISP_API_KEY
verify_cert = true
```

---

## 10. Security Best Practices

### Isolate the Honeypot

```bash
# Use separate network segment
# Configure firewall to restrict outbound connections
sudo ufw default deny outgoing
sudo ufw allow out 53/udp  # DNS
sudo ufw allow out 80/tcp  # HTTP (for updates only)
sudo ufw allow out 443/tcp # HTTPS (for updates only)
sudo ufw enable

# Allow inbound only on honeypot ports
sudo ufw allow 22/tcp
sudo ufw allow 23/tcp
```

### Monitor System Resources

```bash
# Create monitoring script
nano ~/monitor_system.sh
```

```bash
#!/bin/bash

# Check CPU usage
CPU=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')

# Check memory usage
MEM=$(free | grep Mem | awk '{print ($3/$2) * 100.0}')

# Check disk usage
DISK=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')

echo "$(date): CPU: ${CPU}% | Memory: ${MEM}% | Disk: ${DISK}%"

# Alert if thresholds exceeded
if (( $(echo "$CPU > 80" | bc -l) )); then
    echo "HIGH CPU USAGE ALERT"
fi
```

### Regular Backups

```bash
# Backup script
#!/bin/bash

BACKUP_DIR="/backup/cowrie"
DATE=$(date +%Y%m%d)

mkdir -p $BACKUP_DIR

# Backup logs
tar -czf $BACKUP_DIR/logs-$DATE.tar.gz \
    /home/cowrie/cowrie/var/log/

# Backup downloads
tar -czf $BACKUP_DIR/downloads-$DATE.tar.gz \
    /home/cowrie/cowrie/var/lib/cowrie/downloads/

# Keep only last 30 days
find $BACKUP_DIR -mtime +30 -delete
```

### Update Cowrie Regularly

```bash
# Create update script
cd ~/cowrie
git pull
source cowrie-env/bin/activate
pip install --upgrade -r requirements.txt
bin/cowrie restart
```

---

## 11. Advanced Configuration

### Custom File System

```bash
# Customize fake filesystem
cp share/cowrie/fs.pickle var/lib/cowrie/fs.pickle
nano share/cowrie/txtcmds/
```

### Proxy Support

```ini
[proxy]
backend = simple
backend_ssh_host = localhost
backend_ssh_port = 2022
backend_user = richard
backend_pass = foobar
```

### Docker Compose Setup

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  cowrie:
    image: cowrie/cowrie
    container_name: cowrie-honeypot
    restart: unless-stopped
    ports:
      - "2222:2222"
      - "2223:2223"
    volumes:
      - ./cowrie-data/etc:/cowrie/cowrie-git/etc
      - ./cowrie-data/var:/cowrie/cowrie-git/var
    environment:
      - TZ=UTC
    networks:
      - honeypot-net

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.10.0
    container_name: cowrie-elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    networks:
      - honeypot-net

  kibana:
    image: docker.elastic.co/kibana/kibana:8.10.0
    container_name: cowrie-kibana
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    depends_on:
      - elasticsearch
    networks:
      - honeypot-net

volumes:
  elasticsearch-data:

networks:
  honeypot-net:
```

Start the stack:

```bash
docker-compose up -d
```

---

## 12. Troubleshooting

### Cowrie Won't Start

```bash
# Check logs
tail -f var/log/cowrie/cowrie.log

# Check if port is already in use
sudo netstat -tulpn | grep 2222

# Verify Python dependencies
pip list
pip install -r requirements.txt

# Check file permissions
ls -la var/log/cowrie/
```

### No Attacks Detected

```bash
# Verify ports are accessible from internet
# From external machine:
nmap -p 22,23 your-server-ip

# Check firewall
sudo ufw status

# Verify Cowrie is listening
sudo netstat -tulpn | grep python
```

### High Resource Usage

```bash
# Limit log file size
# Edit cowrie.cfg
[output_jsonlog]
rotate_size = 100M
rotate_count = 5

# Check for log rotation
ls -lh var/log/cowrie/
```

---

## 13. Useful Resources

### Official Documentation
- **Cowrie GitHub**: https://github.com/cowrie/cowrie
- **Cowrie Documentation**: https://cowrie.readthedocs.io/
- **Cowrie Wiki**: https://github.com/cowrie/cowrie/wiki

### Community Resources
- **Cowrie Slack**: https://cowrie.slack.com/
- **Threat Intelligence Feeds**: https://otx.alienvault.com/

### Related Tools
- **Honeytrap**: Multi-service honeypot
- **T-Pot**: Multi-honeypot platform
- **Modern Honey Network (MHN)**: Centralized honeypot management

---

## Summary

This guide covered:
- Installing and configuring Cowrie honeypot
- Setting up comprehensive logging (JSON, syslog, database)
- Implementing real-time monitoring solutions
- Integrating with ELK Stack, Splunk, and MISP
- Analyzing captured attack data
- Security best practices and isolation
- Advanced configurations and Docker setup
- Troubleshooting common issues

**Key Takeaways:**
- Always isolate honeypots from production systems
- Regularly monitor and analyze logs
- Keep Cowrie updated
- Share intelligence with community
- Follow responsible disclosure practices

**Next Steps:**
1. Deploy Cowrie in isolated environment
2. Configure logging and monitoring
3. Set up automated analysis
4. Integrate with threat intelligence platforms
5. Contribute findings to security community

Remember: Use honeypots ethically and legally. Ensure you have proper authorization and follow local regulations regarding cybersecurity research.
