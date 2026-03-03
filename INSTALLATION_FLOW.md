# Wazuh Installation & Initialization Flow

## Complete Step-by-Step Walkthrough

This guide explains **exactly what happens** when you install Wazuh, from start to finish.

---

## 🚀 Phase 1: Pre-Installation Checks

```
User runs: ./install.sh
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ Script Initialization (install.sh)                    │
│                                                       │
│ 1. Detect OS type & version                          │
│    - Check for Linux distro (CentOS, Ubuntu, etc)    │
│    - Check architecture (x86_64, ARM, etc)           │
│    - Detect available shell (bash, zsh, sh)          │
│                                                       │
│ 2. Check available tools & compilers                 │
│    - Verify gcc/clang installed                      │
│    - Check for make/gmake                            │
│    - Check for CMake (must be 3.12.4+)              │
│                                                       │
│ 3. Check existing installation                       │
│    - Is Wazuh already installed?                     │
│    - If yes → upgrade path or reinstall             │
│                                                       │
│ 4. Parse command line arguments                      │
│    - ./install.sh debug          (build w/ debug)    │
│    - ./install.sh binary-install (skip compilation) │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ User Prompts & Configuration Selection               │
│ (if not running in automated mode)                   │
│                                                       │
│ 1. Installation Type:                                │
│    ┌─────────────┬──────────────┬───────────────┐   │
│    │  Manager    │ Agent        │  Hybrid       │   │
│    │ (Central)   │(Endpoint)    │(Both)         │   │
│    └─────────────┴──────────────┴───────────────┘   │
│                                                       │
│ 2. Installation Directory:                           │
│    Default: /var/ossec                              │
│    User can override (e.g., /opt/wazuh)            │
│                                                       │
│ 3. Enable/Disable Features:                          │
│    ☑ File Integrity Monitoring (syscheck)           │
│    ☑ Rootkit Detection (rootcheck)                   │
│    ☑ System Collector (inventory)                    │
│    ☑ Email Alerts (if applicable)                    │
│                                                       │
│ 4. Optional: Pre-registration data                   │
│    - Agent name, ID, manager IP (for agents)        │
│                                                       │
│ 5. Optional: Database support                        │
│    - PostgreSQL, MySQL support                       │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
```

---

## 🏗️ Phase 2: Dependency Download & Compilation

```
┌──────────────────────────────────────────────────────┐
│ Dependency Download (make deps)                       │
│ Location: /src/ directory                            │
│                                                       │
│ Downloads external libraries:                        │
│ - OpenSSL (encryption)                              │
│ - zlib (compression)                                │
│ - SQLite (local database)                           │
│ - cJSON (JSON parsing)                              │
│ - msgpack (serialization)                           │
│ - cpython (optional optimization)                   │
│                                                       │
│ Files saved to: /src/external/                      │
│ Time: ~5-10 minutes (depends on internet)           │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ Source Code Compilation (make build)                 │
│ Location: /src/                                      │
│                                                       │
│ The Makefile compiles different binaries depending  │
│ on target (SERVER or AGENT):                        │
│                                                       │
│ TARGET=server  → Compiles:                          │
│   ✓ wazuh-remoted    (receives agent data)          │
│   ✓ wazuh-analysisd  (rules engine)                 │
│   ✓ wazuh-execd      (active response)              │
│   ✓ wazuh-monitord   (monitoring)                   │
│   ✓ wazuh-logtest    (rule testing)                 │
│   ✓ manage_agents    (agent management)             │
│   + all shared libraries                            │
│                                                       │
│ TARGET=agent   → Compiles:                          │
│   ✓ wazuh-agentd     (agent daemon)                 │
│   ✓ logcollector     (log collection)               │
│   ✓ syscheckd        (file monitoring)              │
│   ✓ rootcheck        (rootkit detection)            │
│   ✓ syscollector     (system inventory)             │
│   + all shared libraries                            │
│                                                       │
│ Process:                                             │
│ .c/.cpp files ──► GCC/Clang ──► Object files (.o)  │
│                       │                              │
│                       ▼                              │
│                  Linker ──► Executables              │
│                                                       │
│ Build flags applied:                                 │
│ - Optimization: -O2 or -O3                          │
│ - Debug: -g (if debug mode)                         │
│ - CPU threads: -j$(nproc) (parallel compilation)    │
│                                                       │
│ Time: ~15-60 minutes (depends on hardware)          │
│                                                       │
│ Output location: /src/build/                        │
│ Compiled binaries are generated here first          │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
```

---

## 📦 Phase 3: Installation (Copy Files)

```
┌──────────────────────────────────────────────────────┐
│ Create Installation Directory Structure              │
│ Location: /var/ossec/ (or user-specified)          │
│                                                       │
│ Created directories:                                 │
│ /var/ossec/bin/           ← Executables             │
│ /var/ossec/etc/           ← Configuration files     │
│ /var/ossec/lib/           ← Shared libraries        │
│ /var/ossec/queue/         ← Data queues             │
│ /var/ossec/var/           ← Runtime files           │
│ /var/ossec/logs/          ← Log files               │
│ /var/ossec/ruleset/       ← Security rules          │
│ /var/ossec/patches/       ← Updates                 │
│ /var/ossec/backup/        ← Configuration backups   │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ Copy Compiled Binaries                               │
│                                                       │
│ From: /src/build/ or build directories             │
│ To:   /var/ossec/bin/                              │
│                                                       │
│ Manager binaries copied:                            │
│ - wazuh-remoted                                     │
│ - wazuh-analysisd                                   │
│ - wazuh-execd                                       │
│ - wazuh-monitord                                    │
│ - manage_agents                                     │
│ - wazuh-logtest                                     │
│ - wazuh-control                                     │
│                                                       │
│ Agent binaries copied:                              │
│ - wazuh-agentd                                      │
│ - logcollector (integrated in agent)                │
│ - syscheckd (integrated in agent)                   │
│ - rootcheck (integrated in agent)                   │
│ - wazuh-control                                     │
│ - agent-auth (for pre-registration)                │
│                                                       │
│ Permissions set: 750 (rwxr-x---)                    │
│ Owner: root (or wazuh user)                         │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ Copy Configuration & Rule Files                      │
│                                                       │
│ From: /etc/ and /ruleset/ in repo                  │
│ To:   /var/ossec/etc/ and /var/ossec/ruleset/     │
│                                                       │
│ Configuration files copied:                         │
│ - ossec.conf (main config template)                │
│ - internal_options.conf (internal settings)        │
│ - client.keys (for agents, initially empty)        │
│ - local_internal_options.conf                      │
│                                                       │
│ Rules & Decoders copied:                           │
│ - Hundreds of .xml rule files                      │
│ - Decoder files for parsing                        │
│ - SCA (Security Config Assessment) policies        │
│ - Active response scripts                          │
│                                                       │
│ Examples copied:                                    │
│ - Example configurations                           │
│ - Sample setups                                    │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ Create System Users & Groups                         │
│                                                       │
│ System users created:                               │
│                                                       │
│ User: root    (UID: 0)                              │
│ Group: root   (GID: 0)                              │
│ Purpose: File ownership for most Wazuh files       │
│                                                       │
│ User: wazuh   (UID: 10000+ or first available)     │
│ Group: wazuh  (GID: 10000+ or first available)     │
│ Purpose: Run analysisd, logcollector (non-root)    │
│                                                       │
│ Directories owned by:                               │
│ - /var/ossec/bin/          → root:root, 750        │
│ - /var/ossec/etc/          → root:root, 750        │
│ - /var/ossec/queue/        → root:wazuh, 770       │
│ - /var/ossec/logs/         → wazuh:wazuh, 750      │
│ - /var/ossec/var/          → root:root, 750        │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
```

---

## ⚙️ Phase 4: Post-Installation Configuration

```
┌──────────────────────────────────────────────────────┐
│ Generate Configuration Files                         │
│                                                       │
│ For MANAGER:                                         │
│ ┌────────────────────────────────────────────────┐  │
│ │ /var/ossec/etc/ossec.conf                      │  │
│ │  <ossec_config>                                │  │
│ │    <global>                                    │  │
│ │      <email_from>wazuh@yourcompany.com</email>│  │
│ │      <smtp_server>localhost</smtp_server>      │  │
│ │    </global>                                  │  │
│ │                                                │  │
│ │    <remote>                                   │  │
│ │      <connection>secure</connection>          │  │
│ │      <port>1514</port>                        │  │
│ │      <protocol>tcp</protocol>                 │  │
│ │    </remote>                                  │  │
│ │                                                │  │
│ │    <rules>                                    │  │
│ │      (references to all rule files)           │  │
│ │    </rules>                                   │  │
│ │   </ossec_config>                             │  │
│ └────────────────────────────────────────────────┘  │
│                                                       │
│ For AGENT:                                           │
│ ┌────────────────────────────────────────────────┐  │
│ │ /var/ossec/etc/ossec.conf                      │  │
│ │  <ossec_config>                                │  │
│ │    <client>                                    │  │
│ │      <server-ip>192.168.1.100</server-ip>     │  │
│ │      <server-hostname>wazuh-manager</s...>    │  │
│ │      <port>1514</port>                        │  │
│ │      <protocol>tcp</protocol>                 │  │
│ │    </client>                                  │  │
│ │                                                │  │
│ │    <syscheck>                                 │  │
│ │      <frequency>3600</frequency>              │  │
│ │      (directories to monitor)                 │  │
│ │    </syscheck>                                │  │
│ │   </ossec_config>                             │  │
│ └────────────────────────────────────────────────┘  │
│                                                       │
│ Files are customizable - user can edit before start │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ Create Optional: Init Script for Auto-Start         │
│                                                       │
│ Location depends on OS:                             │
│                                                       │
│ Linux (systemd):                                    │
│ /etc/systemd/system/wazuh-manager.service          │
│ /etc/systemd/system/wazuh-agent.service            │
│                                                       │
│ Linux (SysV init):                                  │
│ /etc/init.d/wazuh                                  │
│                                                       │
│ Script enables auto-start on boot:                 │
│ systemctl enable wazuh-manager                     │
│ chkconfig wazuh on   (older systems)               │
│                                                       │
│ This means Wazuh automatically starts when OS boots │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ Manager-Only: Initialize Alert Database             │
│                                                       │
│ wazuh-db is initialized:                            │
│ Location: /var/ossec/queue/db/                     │
│                                                       │
│ Creates SQLite databases:                           │
│ - alerts.db      (alert storage)                    │
│ - agents.db      (agent registry + inventory)       │
│ - wazuh.db       (global metadata)                  │
│                                                       │
│ Initial structure created:                          │
│ - Tables for alerts                                 │
│ - Tables for agent data                            │
│ - Tables for inventory                             │
│ - Indexes for fast querying                        │
│                                                       │
│ Database is empty until first data arrives         │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ Optional: Pre-Register Agent with Manager           │
│                                                       │
│ On MANAGER:                                         │
│ $ /var/ossec/bin/manage_agents -a                  │
│                                                       │
│ This generates:                                     │
│ ┌────────────────────────────────────────────────┐  │
│ │ /var/ossec/etc/client.keys                     │  │
│ │                                                │  │
│ │ Format: ID NAME IP KEY                         │  │
│ │ 001 linux-agent 192.168.1.50                   │  │
│ │ qeH45k[+XD5gLhLSzHLdUKGFn0jnHPrFwN...         │  │
│ │                                                │  │
│ │ 002 windows-pc 10.0.0.25                       │  │
│ │ AbC123[+xYZ9mNoPqRsT...                       │  │
│ │                                                │  │
│ │ The KEY is shared secret used for encryption  │  │
│ └────────────────────────────────────────────────┘  │
│                                                       │
│ This key must be copied to agent before startup    │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
```

---

## 🚀 Phase 5: Service Startup & Initial Operation

```
User runs: /var/ossec/bin/wazuh-control start
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ wazuh-control Script (Daemon Manager)                │
│ Location: /var/ossec/bin/wazuh-control             │
│                                                       │
│ This script:                                         │
│ - Reads configuration from ossec.conf              │
│ - Reads installation type (manager/agent)          │
│ - Determines which daemons to start                │
│ - Starts processes in correct order                │
│ - Monitors daemon health                           │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ MANAGER STARTUP SEQUENCE                             │
│                                                       │
│ 1. Start wazuh-remoted (Daemon 1)                    │
│    Purpose: Receive encrypted data from agents      │
│    Binds to: Port 1514 (TCP/UDP)                    │
│    ┌────────────────────────────────────┐           │
│    │ $ /var/ossec/bin/wazuh-remoted -f  │           │
│    │ [*] Remote daemon started            │           │
│    │ Listening on port 1514              │           │
│    │ Max agents: 14000                    │           │
│    └────────────────────────────────────┘           │
│    Loads: /var/ossec/etc/client.keys               │
│    Status: Waiting for agent connections           │
│             Ready to receive data                   │
│                                                       │
│ 2. Start wazuh-analysisd (Daemon 2)                 │
│    Purpose: Rule matching & alert generation        │
│    Reads: All rule files from /var/ossec/ruleset/  │
│    ┌────────────────────────────────────┐           │
│    │ $ /var/ossec/bin/wazuh-analysisd -f│           │
│    │ [*] Analysis daemon started          │           │
│    │ Loaded 4500+ rules                   │           │
│    │ Max events/sec: 1000                │           │
│    └────────────────────────────────────┘           │
│    Loads entire ruleset into memory:                │
│    - Decoders: Parse log structures                 │
│    - Rules: Pattern matching rules                  │
│    - Pre-processors: Output formatting              │
│    Status: Ready to analyze events                  │
│                                                       │
│ 3. Start wazuh-execd (Daemon 3)                     │
│    Purpose: Execute active response scripts         │
│    ┌────────────────────────────────────┐           │
│    │ $ /var/ossec/bin/wazuh-execd -f    │           │
│    │ [*] Exec daemon started              │           │
│    │ Loaded response scripts               │           │
│    └────────────────────────────────────┘           │
│    Loads: Active response scripts from              │
│    /var/ossec/active-response/bin/                 │
│    Examples:                                        │
│    - firewall-drop.sh (block attacker IP)          │
│    - disable-account.sh (lock compromised user)    │
│    - host-deny.sh (edit /etc/hosts.deny)           │
│    Status: Ready to execute responses               │
│                                                       │
│ 4. Start wazuh-monitord (Daemon 4)                  │
│    Purpose: Monitor other daemons & overall system  │
│    ┌────────────────────────────────────┐           │
│    │ $ /var/ossec/bin/wazuh-monitord -f │           │
│    │ [*] Monitor daemon started           │           │
│    │ Checking: remoted, analysisd, execd │           │
│    └────────────────────────────────────┘           │
│    Tasks:                                            │
│    - Check if other daemons are running            │
│    - Restart dead daemons automatically            │
│    - Monitor disk space                            │
│    - Rotate logs                                   │
│    - Clean old files                               │
│    Status: Monitoring system health                 │
│                                                       │
│ All manager daemons running:                        │
│ ✓ wazuh-remoted                                     │
│ ✓ wazuh-analysisd                                   │
│ ✓ wazuh-execd                                       │
│ ✓ wazuh-monitord                                    │
│ ✓ (optionally) wazuh-apid (Python API)             │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────────────────────────────────────┐
│ AGENT STARTUP SEQUENCE                               │
│                                                       │
│ Prerequisites:                                       │
│ - /var/ossec/etc/client.keys must exist            │
│ - Must contain key received from manager           │
│ - ossec.conf must have manager IP/hostname        │
│ - ossec.conf must have agent name/ID              │
│                                                       │
│ 1. Start wazuh-agentd (Main Agent Process)          │
│    Purpose: Manage all agent operations             │
│    ┌────────────────────────────────────┐           │
│    │ $ /var/ossec/bin/wazuh-agentd -f   │           │
│    │ [*] Wazuh agent started             │           │
│    │ Agent name: linux-agent             │           │
│    │ Agent ID: 001                       │           │
│    │ Manager: 192.168.1.100:1514         │           │
│    └────────────────────────────────────┘           │
│                                                       │
│    Integrated sub-processes:                        │
│                                                       │
│    a) Logcollector Module                          │
│       Purpose: Collect logs from system files      │
│       Monitors: (configured in ossec.conf)         │
│       - /var/log/auth.log (Linux)                  │
│       - /var/log/syslog                           │
│       - /var/log/apache2/access.log               │
│       - /var/log/mysql/error.log                  │
│       - Windows Event Log (on Windows agents)     │
│       - JSON logs from apps                        │
│                                                       │
│    b) Syscheckd Module (FIM)                       │
│       Purpose: Monitor file integrity              │
│       Scans: (configured in ossec.conf)            │
│       - /etc/  (system files)                      │
│       - /usr/bin/  (binaries)                      │
│       - Custom paths                               │
│       Creates MD5/SHA256 hashes of files          │
│       Detects any file changes                     │
│                                                       │
│    c) Rootcheck Module                             │
│       Purpose: Detect rootkits & hidden processes  │
│       Checks:                                       │
│       - /dev/ for hidden files                     │
│       - Process list for hidden processes          │
│       - System calls for anomalies                  │
│       - Registry (Windows)                         │
│       - Open ports for unregistered listeners      │
│                                                       │
│    d) Syscollector Module                          │
│       Purpose: Collect system inventory            │
│       Gathers:                                      │
│       - OS info (kernel, version)                  │
│       - Installed packages                        │
│       - Network interfaces                        │
│       - Running processes                         │
│       - Hardware info                             │
│       Sends periodically (useful for Asset Mgmt)   │
│                                                       │
│ 2. Establish Connection to Manager                 │
│    ┌────────────────────────────────────┐           │
│    │ Encrypting with shared secret...    │           │
│    │ Connecting to 192.168.1.100:1514   │           │
│    │ (waiting for connection acceptance) │           │
│    │ Connection established!             │           │
│    │ Authentication accepted             │           │
│    └────────────────────────────────────┘           │
│                                                       │
│    Connection process:                              │
│    - Agent reads key from /var/ossec/etc/client.keys
│    - Extracts: Agent ID, encryption key            │
│    - Creates TLS/SSL socket to manager             │
│    - Sends agent identification                    │
│    - Manager validates key matches                 │
│    - Manager sends ACK                             │
│    - Connection encrypted established              │
│                                                       │
│ Agent now running:                                  │
│ ✓ Collecting logs continuously                     │
│ ✓ Monitoring file changes (at intervals)           │
│ ✓ Scanning for rootkits (at intervals)             │
│ ✓ Collecting system inventory                      │
│ ✓ Buffering events locally                         │
│ ✓ Encrypting and sending to manager               │
│                                                       │
└──────────────────────────────────────────────────────┘
           │
           ▼
```

---

## 📊 Phase 6: Normal Operation (After Startup)

```
┌──────────────────────────────────────────────────────┐
│ CONTINUOUS DATA FLOW                                 │
│                                                       │
│ AGENT SIDE:                                          │
│ Every 2-10 seconds (configurable):                  │
│ 1. Collect events from monitored sources            │
│ 2. Add metadata (agent ID, timestamp, source)       │
│ 3. Buffer in memory                                 │
│ 4. Encrypt using shared secret                      │
│ 5. Compress                                         │
│ 6. Send to manager on port 1514                     │
│ 7. Buffer locally if manager offline                │
│                                                       │
│ MANAGER SIDE:                                        │
│ 1. Receive encrypted data (remoted)                 │
│ 2. Decrypt using client.keys                        │
│ 3. Decompress                                       │
│ 4. Parse events                                     │
│ 5. Queue for analysis                               │
│ 6. Apply decoders and rules (analysisd)            │
│ 7. Generate alerts if matched                       │
│ 8. Store in local DB and Elasticsearch             │
│ 9. Trigger active response if needed (execd)       │
│ 10. Monitoring daemon watches all processes        │
│                                                       │
└──────────────────────────────────────────────────────┘
```

---

## 🔄 Complete Data Flow Diagram

```
Agent System                Manager System
    │                            │
    ├─ Logs          ┌─────────────────────────┐
    ├─ Files    ┌────┤ wazuh-remoted           │
    ├─ Procs    │    │ (Port 1514, encrypted)  │
    └─ Sysinfo  │    └─────────────────────────┘
               │              │
               │ Encrypted    ├─ Decrypt
               │ Channel      ├─ Decompress
               │              │
               │              ▼
       ┌───────┴──────┐ ┌─────────────────────────┐
       │              │ │ Event Queue             │
       ▼              └─┤ (buffering)             │
    wazuh-agentd        └─────────────────────────┘
    ├─logcollector              │
    ├─syscheckd          ┌───────▼──────────┐
    ├─rootcheck          │ wazuh-analysisd  │
    └─syscollector       │ (Rule Engine)    │
                         │ - Decoders       │
                         │ - Rules          │
                         │ - Alerts         │
                         └───────┬──────────┘
                                 │
                      ┌──────────┼──────────┐
                      │          │          │
                      ▼          ▼          ▼
                   Local DB  Elasticsearch execd
                  (SQLite)   (Long-term)  (Response)
                      │          │          │
                      │          ▼          │
                      │      Dashboard      │
                      │    (Visualization)  │
                      │                     │
                      └──────────┬──────────┘
                                 │
                            API (REST)
```

---

## 📋 Installation Flow Summary Chart

```
START
  │
  ├─► Detect OS, check dependencies
  │
  ├─► User selects installation type (Manager/Agent)
  │
  ├─► Download external dependencies (make deps)
  │
  ├─► Compile source code (make build)
  │
  ├─► Create /var/ossec/ directory structure
  │
  ├─► Copy compiled binaries
  │
  ├─► Copy configuration & rule files
  │
  ├─► Create system users (wazuh, root)
  │
  ├─► Initialize databases (manager only)
  │
  ├─► Pre-register agents (optional)
  │
  ├─► Create auto-start scripts
  │
  ├─► Installation COMPLETE
  │
  ├─► Start services: /var/ossec/bin/wazuh-control start
  │
  ├─► MANAGER: remoted, analysisd, execd, monitord started
  │
  ├─► AGENT: wazuh-agentd started with modules
  │
  └─► SYSTEM OPERATIONAL: Data flowing, alerts generating
```

---

This flow explains the entire installation and startup process from first command to full operation!

