## Tailscale: Secure Mesh Networking and Windows Deployment

### Introduction

Tailscale is a mesh VPN built on top of WireGuard. Instead of routing all of
your traffic through a central VPN concentrator, it builds direct, encrypted,
peer-to-peer connections between your devices and gives each one a stable
private IP address that works from anywhere.

The practical effect: a laptop in a coffee shop, a server in a datacentre, and
a Windows box behind a home router with no port forwarding can all reach each
other as if they were on the same LAN - without opening a single inbound port
on any firewall.

**What Tailscale gives you:**

- A private network ("tailnet") spanning every device you enrol
- Stable IPs that follow devices across networks, NAT, and reboots
- Encrypted device-to-device traffic using WireGuard
- Identity-based access control tied to your SSO provider
- No inbound firewall rules and no port forwarding
- DNS names for your devices via MagicDNS

**Common use cases:**

- Remote administration of servers without exposing RDP or SSH to the internet
- Connecting to home or lab equipment while travelling
- Linking cloud VPCs, on-premise networks, and laptops into one flat network
- Giving contractors scoped access to specific hosts and ports
- Routing traffic through a specific country or network via an exit node

> **Scope of this guide.** Sections 1-3 cover how Tailscale works and the
> concepts you need. Sections 4-5 cover deploying and driving it on Windows,
> including the unattended-mode trap that catches most people. Sections 6+
> cover hardening a tailnet beyond the permissive defaults.

---

## 1. How Tailscale Actually Works

Understanding the split between the control plane and the data plane explains
most of Tailscale's behaviour, including its privacy properties.

### Control Plane vs Data Plane

| Plane | What it does | Who runs it |
|---|---|---|
| **Control plane** (coordination server) | Distributes public keys, device metadata, ACL policy, and DNS config | Tailscale (or your own Headscale) |
| **Data plane** | Carries your actual packets, encrypted with WireGuard | Directly between your devices |

Each device generates a WireGuard keypair locally. The **private key never
leaves the device.** The coordination server only ever sees public keys and
metadata, so it can tell your devices how to find each other but cannot
decrypt the traffic between them.

### Connection Establishment

1. Device authenticates to the coordination server via your identity provider.
2. It uploads its **public** key and learns the public keys of peers it is
   allowed to talk to (as filtered by your ACL policy).
3. Both devices attempt NAT traversal to open a direct UDP path.
4. If direct connection succeeds, traffic flows peer-to-peer.
5. If it fails, traffic falls back to a **DERP** relay.

### DERP Relays

DERP (Designated Encrypted Relay for Packets) is the fallback path used when
two devices cannot establish a direct connection - typically because both sit
behind restrictive or symmetric NAT.

Important: DERP relays forward **WireGuard-encrypted** packets. The relay
cannot read your traffic; it only sees encrypted blobs and the public keys
involved. Relayed connections are slower and have higher latency than direct
ones, but they are not less private.

Check which path a connection is using:

```powershell
tailscale status
```

A line ending in `direct 203.0.113.5:41641` is a direct connection. A line
reading `relay "syd"` is going through a DERP relay in Sydney.

### Addressing

Every node gets:

- An IPv4 address from the **100.64.0.0/10** CGNAT range (e.g. `100.101.102.103`)
- An IPv6 address from a **fd7a:115c:a1e0::/48** unique-local prefix

The CGNAT range is used deliberately: it is reserved for carrier-grade NAT, so
it very rarely collides with home or corporate LAN addressing the way
`192.168.x.x` or `10.x.x.x` would.

These addresses are **stable**. A device keeps its tailnet IP as it moves
between Wi-Fi, cellular, and ethernet.

### MagicDNS

MagicDNS gives each device a DNS name, so you can use `tailscale ping myserver`
instead of memorising `100.101.102.103`. Names resolve as:

```
<device-name>.<tailnet-name>.ts.net
```

Short names (`myserver`) also resolve when MagicDNS is enabled, because
Tailscale adds the tailnet domain to the DNS search path.

---

## 2. Core Concepts and Terminology

| Term | Meaning |
|---|---|
| **Tailnet** | Your private network - the collection of all your devices and users |
| **Node** | A single device enrolled in the tailnet |
| **Node key** | Per-device WireGuard key, subject to key expiry |
| **Auth key** | Pre-generated credential used to enrol a device without interactive login |
| **Tag** | Machine identity label (`tag:server`) that replaces user ownership |
| **ACL policy** | HuJSON document defining who may reach what, on which ports |
| **Exit node** | A node that routes all of a client's internet traffic |
| **Subnet router** | A node advertising routes to a non-Tailscale subnet |
| **DERP** | Encrypted relay used when direct connection fails |
| **Tailnet lock** | Cryptographic signing of node keys, so the control plane cannot inject nodes |
| **Taildrop** | Direct device-to-device file transfer |
| **Serve / Funnel** | Expose a local service to your tailnet (Serve) or the public internet (Funnel) |

### Ports and Firewall Requirements

Tailscale needs **outbound** access only. Nothing inbound needs to be opened.

| Direction | Protocol / Port | Purpose |
|---|---|---|
| Outbound | UDP 41641 | Direct WireGuard traffic (default local port) |
| Outbound | UDP 3478 | STUN, for NAT traversal |
| Outbound | TCP 443 | Coordination server and DERP relay fallback |

If a network only permits TCP 443, Tailscale still works - it falls back to
DERP over HTTPS - but every connection will be relayed and slower.

---

## 3. Account and Tailnet Setup

Before touching Windows, set up the tailnet itself.

1. Sign up at [https://login.tailscale.com](https://login.tailscale.com) using
   an identity provider (Google, Microsoft, GitHub, Okta, or custom OIDC/SAML).
   The provider you choose becomes the identity source for your ACLs, so pick
   the one that already governs your organisation.
2. Note your **tailnet name** - shown in the admin console. It forms your
   MagicDNS domain (`example-name.ts.net`).
3. Open the **Access Controls** page. Read the default policy before you enrol
   anything, and see section 6.1 - the default is permissive.

**Before you enrol devices, understand the default:** a brand-new tailnet
ships with an allow-all policy. Every device can reach every other device on
every port. That is convenient for a first test and wrong for anything real.
Section 6 fixes it.

---

## 4. Deploying Tailscale on Windows

### 4.1 Interactive Installation

Pick whichever matches how you manage software.

**Option A - Direct download (simplest):**

Download the MSI from
[https://tailscale.com/download/windows](https://tailscale.com/download/windows)
and run it. Tailscale installs a service plus a system-tray application.

**Option B - winget (built into Windows 10 1809+ and Windows 11):**

```powershell
winget install --id Tailscale.Tailscale
```

**Option C - Chocolatey:**

```powershell
choco install tailscale -y
```

After installation, sign in from the tray icon, or from a terminal:

```powershell
tailscale up
```

This opens a browser window for authentication. Once you approve, the device
appears in the admin console.

### 4.2 Add tailscale.exe to Your PATH

The installer places the CLI at `C:\Program Files\Tailscale\tailscale.exe`.
Recent installers add this to the system PATH, but if `tailscale` is not
recognised in a new terminal, add it:

```powershell
# Check whether it already resolves
Get-Command tailscale -ErrorAction SilentlyContinue

# Add to the system PATH (run as Administrator, then open a NEW terminal)
$tsPath = 'C:\Program Files\Tailscale'
$current = [Environment]::GetEnvironmentVariable('Path', 'Machine')
if ($current -notlike "*$tsPath*") {
    [Environment]::SetEnvironmentVariable('Path', "$current;$tsPath", 'Machine')
}
```

Verify the install:

```powershell
tailscale version
tailscale status
tailscale ip -4
```

### 4.3 Unattended Mode - The Windows Gotcha

**This is the single most important Windows-specific setting.**

By default on Windows, Tailscale runs in the context of the logged-in user.
When that user signs out, **Tailscale disconnects** and the machine drops off
your tailnet. For a laptop that is reasonable. For a server you administer
remotely, it is a disaster: you reboot the box, nobody logs in, and it never
comes back on the tailnet - which is precisely when you need it.

Enable unattended mode on any machine that must stay connected without a user
session:

```powershell
tailscale up --unattended
```

Or tick **"Run unattended"** in the tray icon menu (Preferences).

For fleet deployment, enforce it via Group Policy / registry so it cannot be
turned off locally - see section 4.5.

> **Rule of thumb:** if you would be upset to find the machine offline after a
> reboot, it needs `--unattended`.

### 4.4 Silent and Automated Deployment

For imaging, MDM, or scripted rollout, install silently and enrol with a
pre-authentication key.

**Step 1 - Generate an auth key.** In the admin console under
**Settings → Keys**, create an auth key. For unattended server deployment use:

- **Reusable** - if the same key enrols multiple machines
- **Pre-approved** - if device approval is enabled (section 6.6)
- **Tagged** (e.g. `tag:server`) - so the node is owned by the tag, not by the
  person who generated the key

Auth keys can be issued with a maximum lifetime of 90 days. Treat them as
secrets: anything holding a reusable auth key can join your tailnet.

**Step 2 - Install silently:**

```powershell
# Install the MSI with no UI
msiexec /i "tailscale-setup-latest.msi" /quiet /norestart
```

**Step 3 - Enrol non-interactively:**

```powershell
$authKey = 'tskey-auth-REPLACE-ME'

& 'C:\Program Files\Tailscale\tailscale.exe' up `
    --authkey=$authKey `
    --unattended `
    --hostname=$env:COMPUTERNAME `
    --advertise-tags=tag:server `
    --accept-routes
```

A complete deployment script:

```powershell
#Requires -RunAsAdministrator
<#
    Deploys and enrols Tailscale on a Windows host.
    Pass the auth key in rather than hardcoding it.
#>
param(
    [Parameter(Mandatory = $true)][string]$AuthKey,
    [string]$Tag      = 'tag:server',
    [string]$Hostname = $env:COMPUTERNAME
)

$ErrorActionPreference = 'Stop'
$exe = 'C:\Program Files\Tailscale\tailscale.exe'

if (-not (Test-Path $exe)) {
    Write-Host 'Installing Tailscale...'
    winget install --id Tailscale.Tailscale `
        --silent --accept-package-agreements --accept-source-agreements
}

# Wait for the service to be ready before enrolling
$deadline = (Get-Date).AddSeconds(60)
while ((Get-Date) -lt $deadline) {
    if (Get-Service -Name 'Tailscale*' -ErrorAction SilentlyContinue) { break }
    Start-Sleep -Seconds 2
}

Write-Host 'Enrolling in tailnet...'
& $exe up --authkey=$AuthKey --unattended --hostname=$Hostname --advertise-tags=$Tag

& $exe status
Write-Host "Tailnet IP: $(& $exe ip -4)"
```

Run it:

```powershell
.\Deploy-Tailscale.ps1 -AuthKey 'tskey-auth-REPLACE-ME'
```

> **Note on MSI properties.** Tailscale's MSI supports installer properties for
> some of these settings, and the supported set has grown over releases. Rather
> than relying on a property name that may not exist in the version you are
> deploying, the script above uses the documented CLI path, which is stable.
> Check the current
> [Windows deployment docs](https://tailscale.com/kb/1189/install-windows)
> if you want to fold configuration into the MSI call itself.

### 4.5 Enforcing Settings with Group Policy / Registry

On Windows, Tailscale reads system policy from the registry, which lets you
lock settings so users cannot override them locally. Policies live under:

```
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Tailscale
```

Commonly used values:

| Value name | Type | Example | Effect |
|---|---|---|---|
| `UnattendedMode` | REG_SZ | `always` | Forces unattended mode on |
| `Tailnet` | REG_SZ | `example.com` | Restricts sign-in to one tailnet |
| `LoginURL` | REG_SZ | `https://login.tailscale.com` | Points at a custom control server (e.g. Headscale) |
| `AdminConsole` | REG_SZ | `hide` | Hides the admin console link in the tray menu |
| `UpdateMenu` | REG_SZ | `hide` | Hides the self-update option |
| `ExitNodeID` | REG_SZ | `<node ID>` | Forces traffic through a specific exit node |

Set them with PowerShell:

```powershell
#Requires -RunAsAdministrator
$key = 'HKLM:\SOFTWARE\Policies\Tailscale'
New-Item -Path $key -Force | Out-Null

# Always run unattended, regardless of user session
New-ItemProperty -Path $key -Name 'UnattendedMode' `
    -Value 'always' -PropertyType String -Force | Out-Null

# Restrict sign-in to your tailnet only
New-ItemProperty -Path $key -Name 'Tailnet' `
    -Value 'example.com' -PropertyType String -Force | Out-Null

Get-ItemProperty -Path $key
```

Restart the Tailscale service to pick up changes:

```powershell
Restart-Service -Name 'Tailscale*' -Force
```

> **Verify before you deploy.** Tailscale has added policy keys over time and
> the client silently ignores values it does not recognise - a typo fails
> quietly rather than erroring, exactly like a misspelled config option. Check
> the current
> [Windows policy documentation](https://tailscale.com/kb/1315/mdm-keys)
> and confirm each key took effect on one test machine before rolling out.

### 4.6 Windows Firewall and Incoming Connections

Tailscale creates its own virtual network adapter. Traffic arriving over that
adapter is still subject to Windows Firewall.

The tray menu has an **"Allow incoming connections"** toggle. When it is off,
Tailscale blocks inbound connections from other tailnet nodes - useful for a
laptop that only ever initiates connections, and the correct default for a
device that offers no services.

The equivalent CLI control is shields-up mode:

```powershell
# Block all incoming connections from the tailnet
tailscale up --shields-up

# Allow them again
tailscale up --shields-up=false
```

If you need a specific service reachable over Tailscale but not from your
local LAN, scope the firewall rule to the Tailscale interface:

```powershell
#Requires -RunAsAdministrator
# Allow RDP over the Tailscale adapter only
New-NetFirewallRule -DisplayName 'RDP over Tailscale' `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 3389 `
    -RemoteAddress 100.64.0.0/10 `
    -Action Allow
```

Scoping `-RemoteAddress` to the CGNAT range means the rule only accepts
connections carrying a tailnet source address. Combine this with an ACL
(section 6.1) so the restriction is enforced at both ends.

**This is the pattern that replaces port-forwarding RDP to the internet.**
Close 3389 at your perimeter entirely and reach it over the tailnet instead.

### 4.7 Windows as a Subnet Router

A subnet router advertises routes to devices that cannot run Tailscale
themselves - printers, IP cameras, switch management interfaces, legacy
appliances.

Windows does not forward IP traffic by default. Enable it first:

```powershell
#Requires -RunAsAdministrator
Set-ItemProperty `
    -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters' `
    -Name 'IPEnableRouter' -Value 1

# A reboot is required for this to take effect
Restart-Computer
```

After rebooting, advertise the subnet:

```powershell
tailscale up --advertise-routes=192.168.1.0/24 --unattended
```

Then **approve the route in the admin console** under the machine's route
settings. Advertised routes do nothing until approved - this is a frequent
source of "I advertised it but nothing works". You can automate approval with
`autoApprovers` (section 6.8).

On the client side, accept advertised routes:

```powershell
tailscale up --accept-routes
```

### 4.8 Windows as an Exit Node

An exit node routes a client's entire internet traffic, not just tailnet
traffic - useful for using an untrusted network safely or appearing to
originate from a specific location.

Enable IP forwarding as in section 4.7, then:

```powershell
tailscale up --advertise-exit-node --unattended
```

Approve it in the admin console, then select it from a client:

```powershell
# List available exit nodes
tailscale exit-node list

# Route all traffic through one
tailscale set --exit-node=my-exit-node

# Keep local LAN reachable while using the exit node
tailscale set --exit-node=my-exit-node --exit-node-allow-lan-access

# Stop using an exit node
tailscale set --exit-node=
```

> `--exit-node-allow-lan-access` matters more than it looks. Without it, using
> an exit node cuts you off from your own local network - your printer, your
> NAS, your router's admin page - because all traffic including LAN-destined
> packets goes out the exit node.

### 4.9 Managing the Windows Service

```powershell
# Status
Get-Service -Name 'Tailscale*'

# Restart (fixes a surprising number of transient issues)
Restart-Service -Name 'Tailscale*' -Force

# Ensure it starts at boot
Set-Service -Name 'Tailscale*' -StartupType Automatic

# Disconnect without uninstalling
tailscale down

# Reconnect
tailscale up

# Log out and forget credentials entirely
tailscale logout
```

---

## 5. Essential CLI Commands

These work identically on Windows, Linux, and macOS.

### Status and Identity

```powershell
tailscale status              # All peers, connection type, online state
tailscale status --json       # Machine-readable, for scripting
tailscale ip -4               # This device's IPv4 tailnet address
tailscale ip -6               # IPv6 address
tailscale whois 100.101.102.103   # Which user/device owns an address
tailscale version
```

### Connectivity Testing

```powershell
# Tailscale-aware ping: reports whether the path is direct or relayed
tailscale ping myserver

# Diagnose NAT traversal and DERP latency
tailscale netcheck
```

`tailscale netcheck` is the first thing to run when connections are slow. It
reports your NAT type, whether UDP works, and latency to each DERP region. If
it shows `UDP: false`, your network is blocking UDP and everything will relay.

### Changing Settings

Older guides use `tailscale up` with flags for everything, which re-applies
every setting at once. `tailscale set` changes one setting without disturbing
the rest:

```powershell
tailscale set --accept-routes           # Accept advertised subnet routes
tailscale set --accept-dns=false        # Stop using MagicDNS
tailscale set --exit-node=              # Clear exit node
tailscale set --hostname=web-01         # Rename this node
```

### File Transfer (Taildrop)

```powershell
# Send a file to another node
tailscale file cp .\report.pdf myserver:

# Receive queued files into the current directory
tailscale file get .
```

### Diagnostics

```powershell
# Generate a diagnostic bundle - include the ID when contacting support
tailscale bugreport
```

---

## 6. Better Configuration: Hardening Your Tailnet

Everything up to here gets you connected. This section is what separates a
working tailnet from a defensible one.

### 6.1 Replace the Default Allow-All ACL

**Start here.** A new tailnet ships with a policy equivalent to:

```json
{
  "acls": [
    { "action": "accept", "src": ["*"], "dst": ["*:*"] },
  ],
}
```

That means every device can reach every other device on every port. One
compromised laptop then has unrestricted network access to every server you
have enrolled. Tailscale's value is identity-based segmentation, and this
default throws it away.

Policy is written in **HuJSON** - JSON with comments and trailing commas
allowed - and edited under **Access Controls** in the admin console.

A realistic starting policy:

```json
{
  // Who is allowed to apply each tag to a machine
  "tagOwners": {
    "tag:server": ["autogroup:admin"],
    "tag:ci":     ["autogroup:admin"],
    "tag:router": ["autogroup:admin"],
  },

  "groups": {
    "group:eng": ["alice@example.com", "bob@example.com"],
    "group:ops": ["alice@example.com"],
  },

  "acls": [
    // Ops get full access
    {
      "action": "accept",
      "src":    ["group:ops"],
      "dst":    ["*:*"],
    },

    // Engineers reach servers on SSH and HTTPS only - not RDP, not SMB
    {
      "action": "accept",
      "src":    ["group:eng"],
      "dst":    ["tag:server:22,443"],
    },

    // Every user can always reach their own devices
    {
      "action": "accept",
      "src":    ["autogroup:member"],
      "dst":    ["autogroup:self:*"],
    },

    // CI runners may reach the artifact host, nothing else
    {
      "action": "accept",
      "src":    ["tag:ci"],
      "dst":    ["tag:server:443"],
    },
  ],
}
```

Key ideas:

- **Default deny.** Anything not matched by an `accept` rule is denied. There
  is no explicit deny rule because none is needed.
- **Ports are part of the destination.** `tag:server:22,443` is far tighter
  than `tag:server:*`. Enumerate the ports you actually need.
- **`autogroup:self`** lets users reach their own devices without granting
  access to anyone else's.

### 6.2 Use Tags for Servers, Not User Ownership

When a person enrols a server with their own login, that server is **owned by
that person**. Two consequences bite later:

1. Its node key expires on the user's schedule and the machine drops off.
2. If that person leaves and their account is deprovisioned, the server's
   access goes with them.

Tagging a machine transfers ownership to the tag. Tagged nodes do not have an
owning user, and **their keys do not expire**.

```powershell
tailscale up --advertise-tags=tag:server --unattended
```

The tag must be declared in `tagOwners`, and the user or auth key applying it
must be listed as an owner of that tag.

> Retagging an existing node requires re-authenticating it, so decide tags
> before a large rollout rather than after.

### 6.3 Test Your ACLs Before Applying Them

Tailscale policy supports a `tests` block that runs on save. If a test fails,
the policy is rejected - so a bad edit cannot lock you out.

```json
{
  "acls": [ /* ... as above ... */ ],

  "tests": [
    {
      "src":    "bob@example.com",
      "accept": ["tag:server:443", "tag:server:22"],
      "deny":   ["tag:server:3389"],
    },
    {
      "src":    "tag:ci",
      "accept": ["tag:server:443"],
      "deny":   ["tag:server:22"],
    },
  ],
}
```

Write a test for every rule you care about, and specifically write `deny`
tests for the things that must *never* be reachable. A `deny` test that starts
passing after a policy edit is how you catch an accidental widening.

### 6.4 Manage Key Expiry Deliberately

Node keys expire after **180 days** by default. When a key expires the device
drops off the tailnet until someone re-authenticates it interactively.

This is a genuine trade-off, not a setting to blindly disable:

| Choice | Benefit | Cost |
|---|---|---|
| Leave expiry on | Stale/lost devices fall off automatically | Servers need periodic re-auth |
| Disable for a node | Servers stay up indefinitely | A stolen key stays valid until manually revoked |
| Use tags | Keys do not expire, ownership is clear | Requires tag policy up front |

The right answer for infrastructure is usually **tags** (section 6.2) rather
than manually disabling expiry per device, because it solves ownership at the
same time.

For user laptops, leave expiry enabled. That is the control that removes a
lost device from your network without any manual action.

### 6.5 Auth Key Hygiene

Auth keys are credentials that enrol machines. Treat them accordingly.

| Key type | Use for | Notes |
|---|---|---|
| **One-off** | A single machine | Safest default; expires on use |
| **Reusable** | Imaging, fleet rollout | Anyone holding it can join your tailnet |
| **Ephemeral** | CI runners, autoscaling, containers | Node is removed automatically when it goes offline |
| **Pre-approved** | Enrolment where device approval is on | Skips manual approval |
| **Tagged** | Any unattended machine | Assigns tag ownership at enrolment |

Practical rules:

- Never commit an auth key to a repository or bake it into a golden image
  that is shared or archived.
- Use **ephemeral** keys for anything that scales up and down, or you will
  accumulate hundreds of dead nodes in the console.
- Set the shortest workable expiry; the maximum is 90 days.
- Revoke a key in the admin console the moment a rollout is finished.
- Prefer OAuth clients over long-lived reusable keys for automation - they
  can mint short-lived auth keys on demand.

### 6.6 Enable Device Approval

With device approval on, a newly enrolled machine cannot join the tailnet
until an administrator approves it - even if it presented valid credentials.
This limits the damage of a leaked auth key.

Enable it under **Settings → Device management**. Pair it with pre-approved
auth keys for machines you are deploying deliberately, so automation still
works while opportunistic enrolment does not.

### 6.7 Tailnet Lock

Tailnet lock is the answer to "what if the coordination server itself is
compromised or coerced?"

Normally the coordination server distributes public keys, so a malicious
control plane could in principle hand your devices a key belonging to an
attacker's node. With tailnet lock enabled, node keys must be **signed by
trusted signing keys that you hold**, and your devices reject any node key
that is not properly signed.

```powershell
# Initialise, designating the current node as a trusted signer
tailscale lock init

# Check status
tailscale lock status

# List nodes awaiting signature
tailscale lock signing-key
```

Considerations before enabling:

- Designate **more than one** signing node. If you lose your only signing key
  you cannot add new devices.
- Every new node needs signing by a trusted device, which adds a step to
  enrolment.
- It is most valuable for high-sensitivity tailnets. For a home lab it is
  usually more ceremony than the threat model justifies.

### 6.8 Auto-Approve Routes and Exit Nodes

Manually approving every subnet route and exit node does not scale and tempts
people into over-broad approvals. Declare them in policy instead:

```json
{
  "autoApprovers": {
    "routes": {
      "192.168.1.0/24": ["tag:router"],
      "10.20.0.0/16":   ["tag:router"],
    },
    "exitNode": ["tag:exit"],
  },
}
```

Now any node carrying `tag:router` that advertises `192.168.1.0/24` is
approved automatically - and, importantly, a node advertising a route you did
*not* list still requires manual approval.

### 6.9 DNS and MagicDNS

Under **DNS** in the admin console:

- **MagicDNS** - on by default; gives every node a resolvable name.
- **Global nameservers** - push a DNS server to every node. Useful for
  internal resolution; also a way to enforce a filtering resolver across the
  fleet.
- **Override local DNS** - forces clients to use your nameservers rather than
  those from DHCP.
- **Split DNS** - resolve only specific domains via a specific nameserver.
  This is usually what you want instead of overriding all DNS:

  ```
  corp.example.com  →  100.101.102.103 (internal DNS server)
  everything else   →  the client's normal resolver
  ```

Split DNS keeps internal names working over the tailnet without routing every
public lookup through your infrastructure.

On a Windows client, if MagicDNS names stop resolving:

```powershell
tailscale set --accept-dns=true
ipconfig /flushdns
Restart-Service -Name 'Tailscale*' -Force
```

### 6.10 Tailscale SSH

Tailscale can terminate SSH itself, authenticating with tailnet identity
instead of managing `authorized_keys` across hosts. Access is then governed by
the same policy file as everything else.

```json
{
  "ssh": [
    {
      "action": "check",          // "accept" for no re-auth prompt
      "src":    ["autogroup:member"],
      "dst":    ["autogroup:self"],
      "users":  ["autogroup:nonroot", "root"],
    },
    {
      "action": "check",
      "src":    ["group:ops"],
      "dst":    ["tag:server"],
      "users":  ["autogroup:nonroot"],
    },
  ],
}
```

`"action": "check"` requires periodic re-authentication through your identity
provider - effectively a second factor in front of a shell. `"accept"` skips
that prompt.

Enable on the host (Tailscale SSH is supported on Linux and macOS hosts;
Windows machines act as SSH *clients*):

```bash
tailscale up --ssh
```

Sessions can be recorded to a recorder node for audit purposes - worth
configuring if you need an access trail for compliance.

### 6.11 Serve and Funnel

**Serve** publishes a local service to your tailnet with an HTTPS certificate
and a MagicDNS name:

```powershell
# Expose a local app on port 3000 to the tailnet over HTTPS
tailscale serve --bg 3000

tailscale serve status
tailscale serve --https=443 off
```

**Funnel** publishes it to the **public internet**:

```powershell
tailscale funnel --bg 3000
```

Funnel requires an explicit policy grant, which is a deliberate safety
measure - you cannot expose a service to the internet by accident:

```json
{
  "nodeAttrs": [
    {
      "target": ["tag:web"],
      "attr":   ["funnel"],
    },
  ],
}
```

> Funnel deserves care. It takes a service that was reachable only by
> authenticated tailnet members and makes it reachable by anyone on the
> internet, which inverts the property that made Tailscale useful. Use Serve
> unless you specifically need public access, and never Funnel an
> unauthenticated admin interface.

---

## 7. Monitoring and Audit

### Configuration Audit Logs

The admin console records administrative actions - policy edits, key creation,
device approvals, user role changes. Review them periodically, and especially
after any suspected compromise.

### Network Flow Logs

Tailscale can record which nodes connected to which, useful for verifying that
your ACLs behave as intended and for incident response. Enable under
**Settings → Logs**. Both configuration and network logs can be streamed to an
external SIEM.

### Scripted Health Checks

Because `tailscale status --json` is machine-readable, monitoring is
straightforward:

```powershell
# Alert if this node is not connected to the tailnet
$status = tailscale status --json | ConvertFrom-Json

if (-not $status.Self.Online) {
    Write-Warning "Tailscale node is OFFLINE"
}

# Report peers currently connected via a DERP relay rather than directly
$status.Peer.PSObject.Properties |
    Where-Object { $_.Value.Online -and $_.Value.Relay -ne '' } |
    ForEach-Object {
        "{0} is relayed via {1}" -f $_.Value.HostName, $_.Value.Relay
    }
```

A sudden shift from direct to relayed connections across many peers usually
means a firewall change started blocking UDP.

---

## 8. Troubleshooting on Windows

### Tailscale Disconnects After Logout or Reboot

The unattended-mode issue from section 4.3. Fix:

```powershell
tailscale up --unattended
```

and enforce `UnattendedMode` via registry policy (section 4.5) so it cannot be
switched off locally.

### Connections Are Slow or Always Relayed

```powershell
tailscale netcheck
tailscale ping <peer>
```

If `netcheck` reports `UDP: false`, the network is blocking UDP and all
traffic is being relayed via DERP. Allow outbound UDP 41641 and UDP 3478. A
symmetric NAT on both ends will also force relaying even when UDP is open.

### MagicDNS Names Do Not Resolve

```powershell
tailscale set --accept-dns=true
ipconfig /flushdns
Get-DnsClientServerAddress -InterfaceAlias 'Tailscale*'
```

Third-party VPN clients and some endpoint security software hijack DNS
resolution and break MagicDNS. If names fail only on one machine, that is the
first thing to check.

### Subnet Routes Advertised but Unreachable

Work through in order:

1. Is the route **approved** in the admin console? (Most common cause.)
2. Is IP forwarding enabled on the router node, and was it rebooted?
   ```powershell
   Get-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters' -Name 'IPEnableRouter'
   ```
3. Is the client accepting routes? `tailscale up --accept-routes`
4. Does your **ACL** permit the traffic? An approved route still obeys policy.
5. Does the target device's own firewall allow the tailnet source range?

### Device Shows Offline in the Console

```powershell
Get-Service -Name 'Tailscale*'
Restart-Service -Name 'Tailscale*' -Force
tailscale status
```

If it stays offline, check whether the node key expired (visible in the admin
console) - re-authenticate with `tailscale up`, and consider tagging the
machine so it does not recur.

### Collecting Diagnostics

```powershell
tailscale bugreport
```

This produces an identifier that Tailscale support can correlate with
server-side logs. Include it in any support request.

---

## 9. Security Best Practices Checklist

**Access control**

- [ ] Replaced the default allow-all ACL with least-privilege rules
- [ ] Destinations specify ports, not `*`
- [ ] `tests` block covers both expected access and expected denials
- [ ] Servers are tagged rather than owned by individual users

**Credentials and enrolment**

- [ ] Device approval enabled
- [ ] Auth keys are one-off or ephemeral wherever practical
- [ ] No auth keys committed to source control or baked into shared images
- [ ] Keys revoked once a rollout completes
- [ ] Key expiry left enabled for user laptops

**Windows specifics**

- [ ] Unattended mode enabled on every machine that must survive logout
- [ ] `UnattendedMode` enforced by registry policy on managed fleets
- [ ] Incoming connections disabled on devices that offer no services
- [ ] Firewall rules scoped to `100.64.0.0/10` rather than `Any`
- [ ] RDP closed at the perimeter and reached over the tailnet instead

**Operations**

- [ ] Exit nodes and subnet routes approved deliberately or via `autoApprovers`
- [ ] Network flow logs enabled and streamed somewhere durable
- [ ] Admin console audit logs reviewed periodically
- [ ] Tailnet lock evaluated against your actual threat model
- [ ] Funnel used only where public exposure is genuinely intended

---

## 10. Useful Resources

### Official Documentation

- **Tailscale Docs**: https://tailscale.com/kb
- **Windows Install Guide**: https://tailscale.com/kb/1189/install-windows
- **ACL Syntax Reference**: https://tailscale.com/kb/1018/acls
- **MDM / Policy Keys**: https://tailscale.com/kb/1315/mdm-keys
- **Tailnet Lock**: https://tailscale.com/kb/1226/tailnet-lock
- **Exit Nodes**: https://tailscale.com/kb/1103/exit-nodes
- **Subnet Routers**: https://tailscale.com/kb/1019/subnets

### Background Reading

- **How NAT Traversal Works**: https://tailscale.com/blog/how-nat-traversal-works
- **WireGuard Protocol**: https://www.wireguard.com/protocol/

### Related Tools

- **Headscale** - open-source, self-hosted implementation of the Tailscale
  coordination server, for teams that need to run the control plane themselves
- **WireGuard** - the underlying tunnel protocol, if you want to build the
  same thing manually without the coordination layer
- **Netbird**, **ZeroTier** - alternative mesh VPN implementations

### Related Guides in This Repository

- [Advanced SSH Connectivity](../LinuxWriteUps/EnhancedSSH_Connectivity.md) -
  SSH hardening that pairs naturally with Tailscale SSH
- [Cybersecurity Architecture Notes](Cybersecurity_Architechture_Notes.md) -
  the zero-trust concepts Tailscale implements
- [PowerShelling in Windows](Powershelling_in_Windows.md) - PowerShell
  fundamentals used throughout this guide

---

## Summary

This guide covered:

- How Tailscale separates the control plane from the WireGuard data plane, and
  why that matters for privacy
- NAT traversal, DERP relays, CGNAT addressing, and MagicDNS
- Installing and enrolling Tailscale on Windows, interactively and silently
- Unattended mode, registry policy enforcement, and firewall scoping
- Windows as a subnet router and as an exit node
- Hardening: least-privilege ACLs, tags, ACL tests, key expiry, auth key
  hygiene, device approval, and tailnet lock
- DNS strategy, Tailscale SSH, Serve and Funnel
- Monitoring, audit logging, and Windows-specific troubleshooting

**Key Takeaways:**

- The default allow-all ACL is the first thing to change, not the last
- On Windows, unattended mode is the difference between a server that comes
  back after reboot and one that does not
- Tag your servers - it fixes key expiry and ownership in one step
- Advertised routes and exit nodes do nothing until approved
- Tailscale removes the need to expose RDP or SSH to the internet; make sure
  you actually close those ports once it is working

**Next Steps:**

1. Create a tailnet and enrol two devices to see direct connectivity working
2. Replace the default ACL and add `tests` before enrolling anything further
3. Deploy to Windows with `--unattended` and tagged auth keys
4. Add a subnet router for equipment that cannot run Tailscale
5. Enable network flow logs and review what your policy actually permits

Remember: Tailscale makes remote access easy, which also makes over-permissive
access easy. The convenience is only a security gain if you pair it with an
ACL that reflects who should genuinely reach what - and with closing the
perimeter ports it replaces.
