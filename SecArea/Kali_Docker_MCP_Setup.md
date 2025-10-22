## Setting Up Kali Linux Docker with MCP for Security Pentesting

### Introduction

This guide covers setting up a containerized Kali Linux environment using Docker and integrating it with Model Context Protocol (MCP) for enhanced security testing workflows. This approach provides an isolated, reproducible, and portable pentesting environment that can be easily managed and version-controlled.

**Benefits of Kali in Docker:**
- Isolated environment (safe testing without affecting host)
- Reproducible setups (consistent across team members)
- Easy cleanup and rebuilding
- Resource-efficient compared to full VMs
- Quick deployment and scaling

**What is MCP?**
Model Context Protocol (MCP) is an open protocol that enables seamless integration between AI assistants and external data sources/tools. In pentesting, MCP allows AI-powered automation, intelligent tool selection, and enhanced reporting.

---

## 1. Prerequisites

### Required Software

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
sudo apt install docker.io -y

# Add user to docker group (avoid sudo for docker commands)
sudo usermod -aG docker $USER

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verify Docker installation
docker --version
docker run hello-world

# Install Docker Compose (optional but recommended)
sudo apt install docker-compose -y
```

### System Requirements
- **Minimum:** 2GB RAM, 20GB disk space
- **Recommended:** 4GB+ RAM, 50GB+ disk space
- Linux, macOS, or Windows with WSL2

---

## 2. Setting Up Kali Linux Docker Container

### Method 1: Quick Start with Official Kali Image

```bash
# Pull official Kali Linux image
docker pull kalilinux/kali-rolling

# Run interactive Kali container
docker run -it --name kali-pentest kalilinux/kali-rolling /bin/bash

# Inside container, update and install tools
apt update && apt upgrade -y

# Install Kali metapackages (choose based on needs)
# Minimal headless install
apt install -y kali-tools-top10

# For full toolset (large download ~15GB)
# apt install -y kali-linux-large

# For specific categories
# apt install -y kali-tools-web kali-tools-database kali-tools-passwords
```

### Method 2: Persistent Kali Container with Volume Mounting

```bash
# Create directory for persistent data
mkdir -p ~/kali-workspace

# Run Kali with volume mount
docker run -it \
  --name kali-persistent \
  --hostname kali-docker \
  -v ~/kali-workspace:/root/workspace \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  --network host \
  --cap-add=NET_ADMIN \
  --device /dev/net/tun \
  kalilinux/kali-rolling /bin/bash
```

**Explanation of flags:**
- `-v ~/kali-workspace:/root/workspace` - Mount local directory for persistent storage
- `-v /tmp/.X11-unix` - Enable GUI applications
- `-e DISPLAY=$DISPLAY` - Forward X11 display
- `--network host` - Use host network (needed for some pentest tools)
- `--cap-add=NET_ADMIN` - Allow network manipulation
- `--device /dev/net/tun` - VPN support

### Method 3: Custom Dockerfile for Preconfigured Environment

Create a `Dockerfile`:

```dockerfile
FROM kalilinux/kali-rolling

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC

# Update and install essential tools
RUN apt update && apt upgrade -y && \
    apt install -y \
    kali-tools-top10 \
    metasploit-framework \
    nmap \
    sqlmap \
    nikto \
    dirb \
    gobuster \
    john \
    hydra \
    burpsuite \
    wordlists \
    seclists \
    curl \
    wget \
    git \
    vim \
    nano \
    python3 \
    python3-pip \
    netcat-traditional \
    net-tools \
    iputils-ping \
    dnsutils \
    && apt clean

# Install Python security libraries
RUN pip3 install --no-cache-dir \
    requests \
    beautifulsoup4 \
    scapy \
    paramiko \
    impacket \
    pwntools

# Create workspace directory
RUN mkdir -p /root/workspace /root/tools /root/reports

# Set working directory
WORKDIR /root/workspace

# Copy custom scripts (if any)
# COPY scripts/ /root/tools/

# Set default shell
CMD ["/bin/bash"]
```

Build and run:

```bash
# Build custom image
docker build -t kali-custom:latest .

# Run custom container
docker run -it \
  --name kali-custom \
  -v ~/kali-workspace:/root/workspace \
  --network host \
  --cap-add=NET_ADMIN \
  kali-custom:latest
```

---

## 3. Docker Compose Setup for Complete Environment

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  kali-pentest:
    image: kalilinux/kali-rolling
    container_name: kali-pentest
    hostname: kali-docker
    stdin_open: true
    tty: true
    network_mode: host
    cap_add:
      - NET_ADMIN
      - NET_RAW
    devices:
      - /dev/net/tun
    volumes:
      - ./workspace:/root/workspace
      - ./tools:/root/tools
      - ./reports:/root/reports
      - /tmp/.X11-unix:/tmp/.X11-unix
    environment:
      - DISPLAY=${DISPLAY}
      - TZ=UTC
    command: /bin/bash

  # Optional: Add vulnerable target containers for practice
  dvwa:
    image: vulnerables/web-dvwa
    container_name: dvwa-target
    ports:
      - "8080:80"
    environment:
      - MYSQL_PASS=password

  metasploitable:
    image: tleemcjr/metasploitable2
    container_name: metasploitable2
    ports:
      - "8081:80"
      - "2222:22"

  # Optional: Database for storing findings
  postgres:
    image: postgres:15
    container_name: pentest-db
    environment:
      - POSTGRES_DB=pentestdb
      - POSTGRES_USER=pentester
      - POSTGRES_PASSWORD=securepass123
    volumes:
      - pgdata:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  pgdata:
```

Start the environment:

```bash
# Start all services
docker-compose up -d

# Access Kali container
docker-compose exec kali-pentest /bin/bash

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

---

## 4. Integrating MCP (Model Context Protocol) with Kali

### Understanding MCP for Pentesting

MCP enables AI assistants to:
- Suggest appropriate tools for specific vulnerabilities
- Parse and analyze scan results
- Generate exploit scripts
- Create detailed reports
- Automate repetitive tasks

### Setting Up MCP Server in Kali

```bash
# Install Node.js (required for MCP)
apt install -y nodejs npm

# Create MCP server directory
mkdir -p /root/mcp-server
cd /root/mcp-server

# Initialize Node project
npm init -y

# Install MCP SDK
npm install @modelcontextprotocol/sdk
```

### Create MCP Server for Pentesting Tools

Create `pentest-mcp-server.js`:

```javascript
#!/usr/bin/env node

import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { exec } from "child_process";
import { promisify } from "util";

const execAsync = promisify(exec);

const server = new Server(
  {
    name: "kali-pentest-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// Define available pentesting tools
server.setRequestHandler("tools/list", async () => {
  return {
    tools: [
      {
        name: "nmap_scan",
        description: "Perform nmap network scan",
        inputSchema: {
          type: "object",
          properties: {
            target: {
              type: "string",
              description: "Target IP or hostname",
            },
            scan_type: {
              type: "string",
              enum: ["quick", "full", "stealth", "udp", "version"],
              description: "Type of scan to perform",
            },
          },
          required: ["target"],
        },
      },
      {
        name: "gobuster_dir",
        description: "Directory brute-forcing with Gobuster",
        inputSchema: {
          type: "object",
          properties: {
            url: {
              type: "string",
              description: "Target URL",
            },
            wordlist: {
              type: "string",
              description: "Path to wordlist (default: common.txt)",
            },
          },
          required: ["url"],
        },
      },
      {
        name: "sqlmap_scan",
        description: "SQL injection detection with SQLMap",
        inputSchema: {
          type: "object",
          properties: {
            url: {
              type: "string",
              description: "Target URL with parameter",
            },
            data: {
              type: "string",
              description: "POST data (optional)",
            },
          },
          required: ["url"],
        },
      },
      {
        name: "nikto_scan",
        description: "Web server vulnerability scan with Nikto",
        inputSchema: {
          type: "object",
          properties: {
            host: {
              type: "string",
              description: "Target host",
            },
          },
          required: ["host"],
        },
      },
    ],
  };
});

// Handle tool execution
server.setRequestHandler("tools/call", async (request) => {
  const { name, arguments: args } = request.params;

  try {
    let command;
    let output;

    switch (name) {
      case "nmap_scan":
        const scanTypes = {
          quick: "-F",
          full: "-p-",
          stealth: "-sS",
          udp: "-sU",
          version: "-sV",
        };
        const scanFlag = scanTypes[args.scan_type] || "-F";
        command = `nmap ${scanFlag} ${args.target}`;
        output = await execAsync(command, { timeout: 300000 });
        break;

      case "gobuster_dir":
        const wordlist = args.wordlist || "/usr/share/wordlists/dirb/common.txt";
        command = `gobuster dir -u ${args.url} -w ${wordlist} -q`;
        output = await execAsync(command, { timeout: 300000 });
        break;

      case "sqlmap_scan":
        const dataParam = args.data ? `--data="${args.data}"` : "";
        command = `sqlmap -u "${args.url}" ${dataParam} --batch --random-agent`;
        output = await execAsync(command, { timeout: 300000 });
        break;

      case "nikto_scan":
        command = `nikto -h ${args.host}`;
        output = await execAsync(command, { timeout: 300000 });
        break;

      default:
        throw new Error(`Unknown tool: ${name}`);
    }

    return {
      content: [
        {
          type: "text",
          text: `Command: ${command}\n\nOutput:\n${output.stdout}${
            output.stderr ? `\n\nErrors:\n${output.stderr}` : ""
          }`,
        },
      ],
    };
  } catch (error) {
    return {
      content: [
        {
          type: "text",
          text: `Error executing ${name}: ${error.message}`,
        },
      ],
      isError: true,
    };
  }
});

// Start server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Kali Pentest MCP Server running on stdio");
}

main().catch(console.error);
```

Make it executable:

```bash
chmod +x pentest-mcp-server.js
```

### Create Python-based MCP Server Alternative

Create `pentest_mcp_server.py`:

```python
#!/usr/bin/env python3

import asyncio
import json
import subprocess
from typing import Any
from mcp.server import Server, NotificationOptions
from mcp.server.models import InitializationOptions
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

# Define pentesting tools
TOOLS = [
    Tool(
        name="nmap_quick_scan",
        description="Perform quick nmap scan on target",
        inputSchema={
            "type": "object",
            "properties": {
                "target": {"type": "string", "description": "IP or hostname"}
            },
            "required": ["target"]
        }
    ),
    Tool(
        name="port_scan",
        description="Scan specific ports on target",
        inputSchema={
            "type": "object",
            "properties": {
                "target": {"type": "string"},
                "ports": {"type": "string", "description": "Port range (e.g., 1-1000)"}
            },
            "required": ["target", "ports"]
        }
    ),
    Tool(
        name="web_enum",
        description="Enumerate web directories",
        inputSchema={
            "type": "object",
            "properties": {
                "url": {"type": "string"},
                "wordlist": {"type": "string", "description": "Optional wordlist path"}
            },
            "required": ["url"]
        }
    )
]

async def run_command(cmd: list[str], timeout: int = 300) -> dict:
    """Execute shell command and return output"""
    try:
        process = await asyncio.create_subprocess_exec(
            *cmd,
            stdout=asyncio.subprocess.PIPE,
            stderr=asyncio.subprocess.PIPE
        )
        stdout, stderr = await asyncio.wait_for(
            process.communicate(),
            timeout=timeout
        )
        return {
            "stdout": stdout.decode(),
            "stderr": stderr.decode(),
            "returncode": process.returncode
        }
    except asyncio.TimeoutError:
        return {"error": "Command timeout"}
    except Exception as e:
        return {"error": str(e)}

async def handle_tool_call(name: str, arguments: dict) -> list[TextContent]:
    """Handle pentesting tool execution"""

    if name == "nmap_quick_scan":
        result = await run_command([
            "nmap", "-F", "-T4", arguments["target"]
        ])

    elif name == "port_scan":
        result = await run_command([
            "nmap", "-p", arguments["ports"], arguments["target"]
        ])

    elif name == "web_enum":
        wordlist = arguments.get("wordlist", "/usr/share/wordlists/dirb/common.txt")
        result = await run_command([
            "gobuster", "dir", "-u", arguments["url"],
            "-w", wordlist, "-q"
        ])
    else:
        return [TextContent(type="text", text=f"Unknown tool: {name}")]

    output = result.get("stdout", "") + result.get("stderr", "")
    if "error" in result:
        output = f"Error: {result['error']}"

    return [TextContent(type="text", text=output)]

async def main():
    """Run MCP server"""
    server = Server("kali-pentest-mcp")

    @server.list_tools()
    async def list_tools() -> list[Tool]:
        return TOOLS

    @server.call_tool()
    async def call_tool(name: str, arguments: Any) -> list[TextContent]:
        return await handle_tool_call(name, arguments)

    async with stdio_server() as (read_stream, write_stream):
        await server.run(
            read_stream,
            write_stream,
            InitializationOptions(
                server_name="kali-pentest-mcp",
                server_version="1.0.0"
            )
        )

if __name__ == "__main__":
    asyncio.run(main())
```

Install Python dependencies:

```bash
pip3 install mcp asyncio
chmod +x pentest_mcp_server.py
```

---

## 5. Configuring MCP Client Integration

### Configure Claude Desktop to use Kali MCP Server

Create or edit `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%/Claude/claude_desktop_config.json` (Windows):

```json
{
  "mcpServers": {
    "kali-pentest": {
      "command": "docker",
      "args": [
        "exec",
        "-i",
        "kali-pentest",
        "node",
        "/root/mcp-server/pentest-mcp-server.js"
      ]
    }
  }
}
```

### Using MCP from Command Line

```bash
# Install MCP CLI (if available)
npm install -g @modelcontextprotocol/cli

# Connect to MCP server
mcp connect stdio node /root/mcp-server/pentest-mcp-server.js

# List available tools
mcp tools list

# Execute tool
mcp tools call nmap_scan '{"target": "192.168.1.1", "scan_type": "quick"}'
```

---

## 6. Essential Kali Tools Setup Inside Container

### Install Tool Categories

```bash
# Web application testing
apt install -y kali-tools-web

# Database assessment
apt install -y kali-tools-database

# Password attacks
apt install -y kali-tools-passwords

# Wireless attacks
apt install -y kali-tools-wireless

# Exploitation tools
apt install -y kali-tools-exploitation

# Forensics tools
apt install -y kali-tools-forensics

# Sniffing & spoofing
apt install -y kali-tools-sniffing

# Information gathering
apt install -y kali-tools-information-gathering
```

### Install Wordlists

```bash
# Install SecLists (comprehensive wordlist collection)
apt install -y seclists

# Extract rockyou wordlist
gunzip /usr/share/wordlists/rockyou.txt.gz

# List available wordlists
ls -lh /usr/share/wordlists/
ls -lh /usr/share/seclists/
```

### Configure Metasploit

```bash
# Initialize Metasploit database
service postgresql start
msfdb init

# Start Metasploit
msfconsole

# Inside msfconsole, verify database
msf6 > db_status
msf6 > workspace -a pentest-project
msf6 > workspace
```

---

## 7. Common Pentesting Workflows with Docker

### Network Scanning Workflow

```bash
# Start Kali container
docker start kali-pentest
docker exec -it kali-pentest /bin/bash

# Inside container:
# 1. Network discovery
nmap -sn 192.168.1.0/24 -oA /root/workspace/network-discovery

# 2. Port scanning
nmap -p- -T4 192.168.1.100 -oA /root/workspace/port-scan

# 3. Service enumeration
nmap -sV -sC -p 22,80,443 192.168.1.100 -oA /root/workspace/service-enum

# 4. Vulnerability scanning
nmap --script vuln 192.168.1.100 -oA /root/workspace/vuln-scan

# Results are saved in ~/kali-workspace/ on host
```

### Web Application Testing Workflow

```bash
# 1. Directory enumeration
gobuster dir -u http://target.com -w /usr/share/wordlists/dirb/common.txt -o /root/workspace/dirs.txt

# 2. Subdomain enumeration
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -o /root/workspace/subdomains.txt

# 3. Technology detection
whatweb http://target.com

# 4. Nikto scan
nikto -h http://target.com -o /root/workspace/nikto-results.txt

# 5. SQL injection testing
sqlmap -u "http://target.com/page?id=1" --batch --dbs

# 6. Burp Suite (with GUI forwarding)
burpsuite &
```

### Password Cracking Workflow

```bash
# Using John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt /root/workspace/hashes.txt

# Using Hashcat (requires GPU, limited in Docker)
hashcat -m 0 -a 0 /root/workspace/hashes.txt /usr/share/wordlists/rockyou.txt

# Using Hydra for SSH brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.100
```

---

## 8. Automating Pentests with MCP

### Example MCP-Powered Workflow

```bash
# Using AI assistant with MCP to automate reconnaissance

# 1. Ask AI to perform initial scan
"Use nmap_scan tool to quickly scan 192.168.1.100"

# 2. AI analyzes results and suggests next steps
"Based on open ports, enumerate web services on port 80"

# 3. AI uses gobuster_dir tool automatically
"Found /admin directory, scan for SQL injection"

# 4. AI uses sqlmap_scan tool
"Detected SQLi vulnerability, generate exploit report"
```

### Creating Automated Scan Scripts

Create `/root/tools/auto-scan.sh`:

```bash
#!/bin/bash

TARGET=$1
WORKSPACE="/root/workspace/${TARGET}"

mkdir -p $WORKSPACE

echo "[+] Starting automated scan for $TARGET"

# Network scan
echo "[*] Running nmap scan..."
nmap -sV -sC -p- $TARGET -oA $WORKSPACE/nmap-full

# Web enumeration
echo "[*] Checking for web services..."
PORTS=$(grep -oP '\d+/tcp.*http' $WORKSPACE/nmap-full.gnmap | cut -d'/' -f1)

for PORT in $PORTS; do
    echo "[*] Enumerating port $PORT..."
    gobuster dir -u http://$TARGET:$PORT -w /usr/share/wordlists/dirb/common.txt -o $WORKSPACE/gobuster-$PORT.txt
    nikto -h http://$TARGET:$PORT -o $WORKSPACE/nikto-$PORT.txt
done

# Vulnerability scanning
echo "[*] Running vulnerability checks..."
nmap --script vuln -p $PORTS $TARGET -oA $WORKSPACE/vuln-scan

echo "[+] Scan complete! Results saved in $WORKSPACE"
```

Make executable:

```bash
chmod +x /root/tools/auto-scan.sh
```

Usage:

```bash
/root/tools/auto-scan.sh 192.168.1.100
```

---

## 9. Security Best Practices

### Container Security

```bash
# Run container with limited privileges
docker run -it \
  --name kali-secure \
  --cap-drop=ALL \
  --cap-add=NET_ADMIN \
  --security-opt=no-new-privileges \
  --read-only \
  --tmpfs /tmp \
  -v ~/kali-workspace:/root/workspace \
  kalilinux/kali-rolling

# Use rootless Docker (recommended)
dockerd-rootless-setuptool.sh install

# Scan container for vulnerabilities
docker scan kalilinux/kali-rolling
```

### Legal and Ethical Considerations

**IMPORTANT:** Only perform pentesting on:
- Systems you own
- Systems you have written permission to test
- Authorized lab environments (DVWA, HackTheBox, TryHackMe, etc.)

**Create Rules of Engagement document:**

```markdown
# Penetration Testing Rules of Engagement

## Scope
- Target systems: [List authorized targets]
- Out of scope: [List excluded systems]
- Testing period: [Start date] to [End date]

## Authorized Activities
- Network scanning
- Vulnerability assessment
- Exploitation (with approval)
- Social engineering (if authorized)

## Restrictions
- No DoS attacks
- No data destruction
- No unauthorized data exfiltration
- Testing only during approved hours

## Contact Information
- Primary contact: [Name, email, phone]
- Emergency contact: [Name, email, phone]

## Authorization
Signed: _________________ Date: _________
```

### Data Protection

```bash
# Encrypt workspace directory
# Install cryptsetup
apt install -y cryptsetup

# Create encrypted volume
cryptsetup luksFormat /path/to/encrypted-volume
cryptsetup luksOpen /path/to/encrypted-volume kali-encrypted

# Mount in Docker
docker run -it \
  -v /dev/mapper/kali-encrypted:/root/workspace \
  kalilinux/kali-rolling
```

---

## 10. Useful Docker Commands for Kali Management

```bash
# Container lifecycle
docker start kali-pentest
docker stop kali-pentest
docker restart kali-pentest
docker exec -it kali-pentest /bin/bash

# View logs
docker logs kali-pentest

# Monitor resource usage
docker stats kali-pentest

# Export container
docker export kali-pentest > kali-backup.tar

# Import container
docker import kali-backup.tar kali-restored:latest

# Save image
docker save kalilinux/kali-rolling > kali-image.tar

# Load image
docker load < kali-image.tar

# Remove container (keep data in volumes)
docker rm kali-pentest

# Remove container and volumes
docker rm -v kali-pentest

# Clean up unused resources
docker system prune -a
```

---

## 11. Troubleshooting

### Common Issues

**GUI applications not working:**
```bash
# On host, allow Docker to access X server
xhost +local:docker

# Run container with proper display forwarding
docker run -it \
  -e DISPLAY=$DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  kalilinux/kali-rolling
```

**Network tools require root:**
```bash
# Run container with --privileged flag (use cautiously)
docker run -it --privileged kalilinux/kali-rolling

# Or add specific capabilities
docker run -it --cap-add=NET_ADMIN --cap-add=NET_RAW kalilinux/kali-rolling
```

**Metasploit database connection issues:**
```bash
# Inside container
service postgresql start
msfdb reinit
```

**Performance issues:**
```bash
# Limit resources
docker run -it \
  --cpus="2.0" \
  --memory="4g" \
  kalilinux/kali-rolling
```

---

## 12. Additional Resources

### Official Documentation
- **Kali Docker Images**: https://www.kali.org/docs/containers/official-kalilinux-docker-images/
- **Docker Documentation**: https://docs.docker.com/
- **MCP Protocol**: https://modelcontextprotocol.io/

### Practice Environments
- **DVWA** (Damn Vulnerable Web App): http://www.dvwa.co.uk/
- **HackTheBox**: https://www.hackthebox.com/
- **TryHackMe**: https://tryhackme.com/
- **VulnHub**: https://www.vulnhub.com/

### Kali Resources
- **Kali Tools**: https://www.kali.org/tools/
- **Kali Training**: https://kali.training/
- **Offensive Security**: https://www.offensive-security.com/

### MCP Integration Examples
- **MCP Servers**: https://github.com/modelcontextprotocol/servers
- **MCP Documentation**: https://modelcontextprotocol.io/docs

---

## Summary

This guide covered:
- Setting up Kali Linux in Docker with persistent storage
- Creating custom Kali images with Dockerfile
- Using Docker Compose for complete pentest environments
- Integrating MCP for AI-assisted pentesting
- Common pentesting workflows
- Security best practices
- Troubleshooting tips

**Next Steps:**
1. Set up your Kali Docker environment
2. Practice with vulnerable applications (DVWA, Metasploitable)
3. Integrate MCP for workflow automation
4. Always test ethically and legally
5. Document findings professionally

Remember: **With great power comes great responsibility.** Only test systems you have permission to assess.
