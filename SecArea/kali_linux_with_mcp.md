# Using Kali Linux with MCP (Model Context Protocol)

## Overview

Kali Linux is a Debian-based Linux distribution designed for digital forensics and penetration testing. When combined with MCP (Model Context Protocol), it provides a powerful platform for security professionals and AI assistants to collaborate on security tasks, vulnerability assessments, and penetration testing workflows.

## Prerequisites

### System Requirements
- Kali Linux 2024.x or later
- Python 3.10+
- Node.js 16+ (for some MCP servers)
- Claude Desktop or compatible MCP client
- 8GB+ RAM recommended
- 20GB+ free disk space

### Required Software
```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Python development tools
sudo apt install python3-pip python3-venv python3-dev -y

# Install Node.js (required for some MCP servers)
sudo apt install nodejs npm -y

# Install Git for version control
sudo apt install git -y

# Install additional security tools
sudo apt install nmap metasploit-framework wireshark burpsuite -y
```

## MCP Setup for Security Operations

### 1. Install Claude Desktop
```bash
# Download and install Claude Desktop (if not already installed)
# Visit https://claude.ai/download for the latest version
```

### 2. Create MCP Configuration Directory
```bash
# Create MCP configuration directory
mkdir -p ~/.config/claude-desktop

# Create Claude Desktop configuration file
touch ~/.config/claude-desktop/claude_desktop_config.json
```

### 3. Essential Security MCP Servers

#### A. Nmap Scanner MCP Server
```bash
# Create directory for Nmap MCP server
mkdir -p ~/mcp-servers/nmap-server
cd ~/mcp-servers/nmap-server

# Create Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install required packages
pip install mcp nmap-python

# Create nmap_mcp.py
cat > nmap_mcp.py << 'EOF'
#!/usr/bin/env python3
"""
Nmap MCP Server for security scanning and network reconnaissance
"""

import asyncio
import json
import subprocess
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
import nmap

app = Server("nmap-security-scanner")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="nmap_scan",
            description="Perform network reconnaissance with Nmap",
            inputSchema={
                "type": "object",
                "properties": {
                    "target": {
                        "type": "string",
                        "description": "Target IP address, hostname, or range"
                    },
                    "scan_type": {
                        "type": "string",
                        "enum": ["quick", "full", "stealth", "aggressive"],
                        "description": "Type of scan to perform",
                        "default": "quick"
                    },
                    "ports": {
                        "type": "string",
                        "description": "Port range to scan (e.g., '1-1000', '22,80,443')",
                        "default": "1-1000"
                    }
                },
                "required": ["target"]
            }
        ),
        Tool(
            name="vulnerability_scan",
            description="Scan for common vulnerabilities",
            inputSchema={
                "type": "object",
                "properties": {
                    "target": {
                        "type": "string",
                        "description": "Target IP address or hostname"
                    },
                    "script_categories": {
                        "type": "array",
                        "items": {"type": "string"},
                        "description": "NSE script categories to run",
                        "default": ["vuln", "safe"]
                    }
                },
                "required": ["target"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "nmap_scan":
        try:
            nm = nmap.PortScanner()
            target = arguments["target"]
            scan_type = arguments.get("scan_type", "quick")
            ports = arguments.get("ports", "1-1000")

            # Configure scan arguments based on type
            scan_args = {
                "quick": "-T4 -F",
                "full": "-p " + ports + " -sV -O",
                "stealth": "-sS -T2",
                "aggressive": "-A -T4"
            }

            result = nm.scan(target, arguments=scan_args[scan_type])

            return [TextContent(
                type="text",
                text=json.dumps({
                    "scan_result": result,
                    "target": target,
                    "scan_type": scan_type,
                    "recommendations": "Review open ports and services for security implications"
                }, indent=2)
            )]

        except Exception as e:
            return [TextContent(
                type="text",
                text=f"Error during scan: {str(e)}"
            )]

    elif name == "vulnerability_scan":
        try:
            target = arguments["target"]
            script_categories = arguments.get("script_categories", ["vuln", "safe"])

            # Build NSE script arguments
            script_args = ",".join([f"{cat}" for cat in script_categories])

            # Run nmap with vulnerability scripts
            cmd = [
                "nmap",
                "-sV",
                "--script", script_args,
                target
            ]

            result = subprocess.run(cmd, capture_output=True, text=True)

            return [TextContent(
                type="text",
                text=f"Vulnerability Scan Results for {target}:\n\n{result.stdout}\n\n"
                     f"Errors (if any):\n{result.stderr}\n\n"
                     f"Recommendations:\n"
                     f"- Review any vulnerabilities found\n"
                     f"- Prioritize critical and high-severity issues\n"
                     f"- Verify false positives with manual testing"
            )]

        except Exception as e:
            return [TextContent(
                type="text",
                text=f"Error during vulnerability scan: {str(e)}"
            )]

    else:
        raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
EOF

chmod +x nmap_mcp.py
```

#### B. Metasploit Framework MCP Server
```bash
# Create directory for Metasploit MCP server
mkdir -p ~/mcp-servers/metasploit-server
cd ~/mcp-servers/metasploit-server

# Create Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install required packages
pip install mcp pymetasploit3

# Create metasploit_mcp.py
cat > metasploit_mcp.py << 'EOF'
#!/usr/bin/env python3
"""
Metasploit Framework MCP Server for penetration testing
"""

import asyncio
import json
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
from metasploit.msfrpc import MsfRpcClient

app = Server("metasploit-framework")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="msfconsole_info",
            description="Get Metasploit framework information and available modules",
            inputSchema={
                "type": "object",
                "properties": {
                    "module_type": {
                        "type": "string",
                        "enum": ["exploit", "payload", "auxiliary", "post", "encoder", "nop"],
                        "description": "Type of modules to list",
                        "default": "exploit"
                    }
                }
            }
        ),
        Tool(
            name="search_exploits",
            description="Search for exploits in Metasploit database",
            inputSchema={
                "type": "object",
                "properties": {
                    "search_term": {
                        "type": "string",
                        "description": "Search term for exploits"
                    },
                    "platform": {
                        "type": "string",
                        "description": "Target platform (e.g., windows, linux, unix)"
                    }
                },
                "required": ["search_term"]
            }
        ),
        Tool(
            name="vulnerability_check",
            description="Check for known vulnerabilities on target",
            inputSchema={
                "type": "object",
                "properties": {
                    "target": {
                        "type": "string",
                        "description": "Target IP address or hostname"
                    },
                    "service": {
                        "type": "string",
                        "description": "Service name or port"
                    }
                },
                "required": ["target"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    try:
        # Connect to Metasploit RPC (assuming msfrpcd is running)
        client = MsfRpcClient('yourpassword', port=55553)  # Change password as needed

        if name == "msfconsole_info":
            module_type = arguments.get("module_type", "exploit")
            modules = client.modules[type]

            return [TextContent(
                type="text",
                text=f"Available {module_type} modules:\n\n" +
                     "\n".join(modules.keys()[:20]) +  # Show first 20 modules
                     f"\n\nTotal {len(modules)} {module_type} modules available.\n\n" +
                     "Note: Full module list truncated for readability."
            )]

        elif name == "search_exploits":
            search_term = arguments["search_term"]
            platform = arguments.get("platform", "")

            # Search exploits
            results = client.modules.exploit
            matching_modules = []

            for module_name in results:
                if search_term.lower() in module_name.lower():
                    if not platform or platform.lower() in module_name.lower():
                        matching_modules.append(module_name)

            return [TextContent(
                type="text",
                text=f"Search Results for '{search_term}':\n\n" +
                     "\n".join(matching_modules[:10]) +
                     f"\n\nFound {len(matching_modules)} matching modules.\n\n" +
                     "Recommendations:\n" +
                     "- Review module descriptions for applicability\n" +
                     "- Check exploit reliability ratings\n" +
                     "- Ensure proper target verification before testing"
            )]

        elif name == "vulnerability_check":
            target = arguments["target"]
            service = arguments.get("service", "")

            # Use auxiliary modules for vulnerability scanning
            scanner_modules = [
                "auxiliary/scanner/portscan/tcp",
                "auxiliary/scanner/smb/smb_ms17_010",
                "auxiliary/scanner/ssh/ssh_login",
                "auxiliary/scanner/http/http_version"
            ]

            recommendations = [
                "Vulnerability Check Recommendations:\n",
                f"Target: {target}",
                f"Service: {service or 'All services'}",
                "\nSuggested Scanners:",
            ]

            recommendations.extend(scanner_modules)
            recommendations.extend([
                "\n\nUsage Guidelines:",
                "- Always get proper authorization before scanning",
                "- Use non-destructive scanning first",
                "- Document all findings",
                "- Verify results with multiple tools"
            ])

            return [TextContent(
                type="text",
                text="\n".join(recommendations)
            )]

        client.logout()

    except Exception as e:
        return [TextContent(
            type="text",
            text=f"Error connecting to Metasploit: {str(e)}\n\n" +
                 "Make sure Metasploit RPC service is running:\n" +
                 "msfrpcd -P yourpassword -S"
        )]

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
EOF

chmod +x metasploit_mcp.py
```

#### C. Security Analysis MCP Server
```bash
# Create directory for Security Analysis MCP server
mkdir -p ~/mcp-servers/security-analysis
cd ~/mcp-servers/security-analysis

# Create Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install required packages
pip install mcp python-magic-bin

# Create security_analysis_mcp.py
cat > security_analysis_mcp.py << 'EOF'
#!/usr/bin/env python3
"""
Security Analysis MCP Server for malware analysis and security investigations
"""

import asyncio
import os
import hashlib
import json
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

app = Server("security-analysis")

@app.list_tools()
async def list_tools():
    return [
        Tool(
            name="file_hash_analysis",
            description="Calculate and analyze file hashes",
            inputSchema={
                "type": "object",
                "properties": {
                    "file_path": {
                        "type": "string",
                        "description": "Path to the file to analyze"
                    },
                    "hash_algorithms": {
                        "type": "array",
                        "items": {"type": "string"},
                        "description": "Hash algorithms to use",
                        "default": ["md5", "sha1", "sha256"]
                    }
                },
                "required": ["file_path"]
            }
        ),
        Tool(
            name="security_tool_runner",
            description="Run common security analysis tools",
            inputSchema={
                "type": "object",
                "properties": {
                    "tool": {
                        "type": "string",
                        "enum": ["strings", "file", "hexdump", "objdump"],
                        "description": "Security tool to run"
                    },
                    "target_file": {
                        "type": "string",
                        "description": "Target file path"
                    },
                    "options": {
                        "type": "string",
                        "description": "Additional tool options"
                    }
                },
                "required": ["tool", "target_file"]
            }
        ),
        Tool(
            name="log_analysis",
            description="Analyze system logs for security events",
            inputSchema={
                "type": "object",
                "properties": {
                    "log_file": {
                        "type": "string",
                        "description": "Path to log file"
                    },
                    "pattern": {
                        "type": "string",
                        "description": "Pattern to search for (regex supported)"
                    },
                    "context_lines": {
                        "type": "integer",
                        "description": "Number of context lines to show",
                        "default": 3
                    }
                },
                "required": ["log_file"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "file_hash_analysis":
        try:
            file_path = arguments["file_path"]
            hash_algorithms = arguments.get("hash_algorithms", ["md5", "sha1", "sha256"])

            if not os.path.exists(file_path):
                return [TextContent(
                    type="text",
                    text=f"Error: File '{file_path}' not found"
                )]

            hashes = {}
            file_size = os.path.getsize(file_path)

            with open(file_path, 'rb') as f:
                file_data = f.read()

                for algorithm in hash_algorithms:
                    if algorithm == "md5":
                        hashes["md5"] = hashlib.md5(file_data).hexdigest()
                    elif algorithm == "sha1":
                        hashes["sha1"] = hashlib.sha1(file_data).hexdigest()
                    elif algorithm == "sha256":
                        hashes["sha256"] = hashlib.sha256(file_data).hexdigest()

            return [TextContent(
                type="text",
                text=f"File Analysis Results:\n\n" +
                     f"File: {file_path}\n" +
                     f"Size: {file_size} bytes\n\n" +
                     "Hashes:\n" +
                     "\n".join([f"{algo.upper()}: {hash_value}" for algo, hash_value in hashes.items()]) +
                     "\n\nRecommendations:\n" +
                     "- Use VirusTotal to check hashes against malware databases\n" +
                     "- Document hash values for forensic analysis\n" +
                     "- Compare with known good/bad file signatures"
            )]

        except Exception as e:
            return [TextContent(
                type="text",
                text=f"Error analyzing file: {str(e)}"
            )]

    elif name == "security_tool_runner":
        try:
            tool = arguments["tool"]
            target_file = arguments["target_file"]
            options = arguments.get("options", "")

            if not os.path.exists(target_file):
                return [TextContent(
                    type="text",
                    text=f"Error: File '{target_file}' not found"
                )]

            # Build command based on tool
            commands = {
                "strings": f"strings {options} '{target_file}'",
                "file": f"file {options} '{target_file}'",
                "hexdump": f"hexdump -C {options} '{target_file}'",
                "objdump": f"objdump {options} '{target_file}'"
            }

            if tool not in commands:
                return [TextContent(
                    type="text",
                    text=f"Error: Unsupported tool '{tool}'"
                )]

            # Run the command
            import subprocess
            result = subprocess.run(commands[tool], shell=True, capture_output=True, text=True)

            return [TextContent(
                type="text",
                text=f"{tool.upper()} Analysis Results:\n\n" +
                     f"Command: {commands[tool]}\n\n" +
                     f"Output:\n{result.stdout}\n\n" +
                     f"Errors:\n{result.stderr}\n\n" +
                     "Security Recommendations:\n" +
                     "- Review output for suspicious patterns\n" +
                     "- Look for encoded/obfuscated content\n" +
                     "- Identify potential malware indicators"
            )]

        except Exception as e:
            return [TextContent(
                type="text",
                text=f"Error running security tool: {str(e)}"
            )]

    elif name == "log_analysis":
        try:
            log_file = arguments["log_file"]
            pattern = arguments.get("pattern", "")
            context_lines = arguments.get("context_lines", 3)

            if not os.path.exists(log_file):
                return [TextContent(
                    type="text",
                    text=f"Error: Log file '{log_file}' not found"
                )]

            # Read log file
            with open(log_file, 'r') as f:
                log_lines = f.readlines()

            # Search for pattern if provided
            if pattern:
                import re
                matching_lines = []
                for i, line in enumerate(log_lines):
                    if re.search(pattern, line, re.IGNORECASE):
                        start = max(0, i - context_lines)
                        end = min(len(log_lines), i + context_lines + 1)
                        matching_lines.append((i + 1, line.strip(), log_lines[start:end]))

                if matching_lines:
                    result = f"Log Analysis Results for '{log_file}':\n\n"
                    result += f"Pattern: {pattern}\n"
                    result += f"Found {len(matching_lines)} matches\n\n"

                    for line_num, match_line, context in matching_lines[:10]:  # Limit to 10 matches
                        result += f"Line {line_num}: {match_line}\n"
                        result += "Context:\n"
                        for ctx_line in context:
                            result += f"  {ctx_line.rstrip()}\n"
                        result += "\n"
                else:
                    result = f"No matches found for pattern '{pattern}' in '{log_file}'"
            else:
                # General log statistics
                result = f"Log Analysis for '{log_file}':\n\n"
                result += f"Total lines: {len(log_lines)}\n"

                # Count common log patterns
                error_count = sum(1 for line in log_lines if "error" in line.lower())
                warning_count = sum(1 for line in log_lines if "warning" in line.lower())
                failed_count = sum(1 for line in log_lines if "failed" in line.lower())

                result += f"Error entries: {error_count}\n"
                result += f"Warning entries: {warning_count}\n"
                result += f"Failed entries: {failed_count}\n\n"

                result += "Security Recommendations:\n" +
                         "- Search for specific attack patterns\n" +
                         "- Monitor for authentication failures\n" +
                         "- Track unusual access patterns\n" +
                         "- Correlate with network security logs"

            return [TextContent(
                type="text",
                text=result
            )]

        except Exception as e:
            return [TextContent(
                type="text",
                text=f"Error analyzing log file: {str(e)}"
            )]

    else:
        raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
EOF

chmod +x security_analysis_mcp.py
```

## Configure Claude Desktop

Edit the Claude Desktop configuration file:

```bash
nano ~/.config/claude-desktop/claude_desktop_config.json
```

Add the following configuration:

```json
{
  "mcpServers": {
    "nmap-security": {
      "command": "/home/yourusername/mcp-servers/nmap-server/venv/bin/python",
      "args": ["/home/yourusername/mcp-servers/nmap-server/nmap_mcp.py"]
    },
    "metasploit-framework": {
      "command": "/home/yourusername/mcp-servers/metasploit-server/venv/bin/python",
      "args": ["/home/yourusername/mcp-servers/metasploit-server/metasploit_mcp.py"]
    },
    "security-analysis": {
      "command": "/home/yourusername/mcp-servers/security-analysis/venv/bin/python",
      "args": ["/home/yourusername/mcp-servers/security-analysis/security_analysis_mcp.py"]
    }
  }
}
```

**Important**: Replace `/home/yourusername/` with your actual home directory path.

## Security Workflow Examples

### 1. Network Reconnaissance
```
User: "Perform a network scan on 192.168.1.0/24 to identify live hosts and open ports"

Claude will use the nmap_scan tool to:
- Discover live hosts in the network range
- Identify open ports and services
- Provide recommendations for further investigation
```

### 2. Vulnerability Assessment
```
User: "Scan the web server at 192.168.1.100 for common vulnerabilities"

Claude will use the vulnerability_scan tool to:
- Run NSE scripts for known vulnerabilities
- Check for weak configurations
- Suggest remediation steps
```

### 3. File Analysis
```
User: "Analyze this suspicious file: /tmp/suspicious.exe"

Claude will use the security_analysis tools to:
- Calculate file hashes (MD5, SHA1, SHA256)
- Run file type identification
- Extract strings for analysis
- Provide security recommendations
```

### 4. Log Analysis
```
User: "Check the Apache access log for suspicious activity"

Claude will use the log_analysis tool to:
- Identify patterns of attacks
- Highlight failed authentication attempts
- Provide security recommendations
```

## Best Practices and Security Considerations

### 1. Legal and Ethical Guidelines
- **Always obtain proper authorization** before scanning or testing
- **Follow local laws and regulations** regarding security testing
- **Use only on networks and systems you own** or have explicit permission to test
- **Document all activities** for accountability and audit purposes

### 2. Safe Scanning Practices
```bash
# Use non-intrusive scanning first
nmap -sS -T2 target

# Limit scan scope to avoid network disruption
nmap -p 1-1000 target

# Use timing templates appropriately
# T0 = Paranoid (very slow, stealthy)
# T1 = Sneaky (slow, avoids IDS)
# T2 = Polite (reduced bandwidth)
# T3 = Normal (default)
# T4 = Aggressive (faster, more detectable)
# T5 = Insane (very fast, may miss ports)
```

### 3. Environment Setup
```bash
# Create dedicated user for security testing
sudo useradd -m securityuser
sudo usermod -aG sudo securityuser

# Set up secure workspace
mkdir -p ~/security-workspace/{scans,reports,evidence,logs}

# Configure proper file permissions
chmod 700 ~/security-workspace
```

### 4. Documentation and Reporting
- Maintain detailed logs of all security activities
- Create comprehensive reports with findings and recommendations
- Store evidence in secure, tamper-evident locations
- Follow incident response procedures for any discovered issues

### 5. Metasploit RPC Security
```bash
# Secure Metasploit RPC configuration
# Edit /usr/share/metasploit-framework/config/msfrpcd.yml

# Set strong password
# Restrict access to localhost
# Use SSL/TLS connections
# Implement IP whitelist if needed
```

## Integration with Other Kali Tools

### Wireshark Integration
```bash
# Capture network traffic for analysis
sudo wireshark -i eth0 -w /tmp/network_capture.pcap

# Use MCP servers to analyze captured files
```

### Burp Suite Integration
```bash
# Launch Burp Suite for web application testing
burpsuite

# Export findings for MCP analysis
```

### John the Ripper Integration
```bash
# Use with security analysis MCP for password cracking
john --wordlist=/usr/share/wordlists/rockyou.txt hash_file.txt
```

## Troubleshooting

### Common Issues

1. **MCP Server Connection Failed**
   ```bash
   # Check if Claude Desktop is running
   ps aux | grep claude

   # Verify MCP server scripts are executable
   ls -la ~/mcp-servers/*/venv/bin/python

   # Check Claude Desktop logs
   journalctl -u claude-desktop
   ```

2. **Nmap Permission Denied**
   ```bash
   # Install Nmap if missing
   sudo apt install nmap -y

   # Check user permissions
   groups $USER

   # Add user to appropriate groups if needed
   sudo usermod -aG netdev $USER
   ```

3. **Metasploit RPC Connection Failed**
   ```bash
   # Start Metasploit RPC service
   msfrpcd -P yourpassword -S

   # Check if service is running
   netstat -tlnp | grep 55553

   # Update Metasploit framework
   sudo msfupdate
   ```

4. **File Access Permissions**
   ```bash
   # Fix file permissions for MCP servers
   chmod +x ~/mcp-servers/*/*.py

   # Check directory permissions
   ls -la ~/mcp-servers/
   ```

### Getting Help

- **Kali Linux Documentation**: https://www.kali.org/docs/
- **MCP Protocol Documentation**: Available in Claude Desktop help
- **Security Tool Documentation**: Man pages and online resources
- **Community Support**: Kali Linux forums and security communities

## Conclusion

Integrating Kali Linux with MCP provides a powerful platform for security professionals to leverage AI assistance in their penetration testing and security analysis workflows. By following proper security practices, legal guidelines, and ethical considerations, you can enhance your security testing capabilities while maintaining professional standards.

Remember that these tools are powerful and should only be used responsibly and with proper authorization. Always prioritize security, legality, and ethical conduct in your security testing activities.