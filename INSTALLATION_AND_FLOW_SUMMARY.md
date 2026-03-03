# Wazuh Installation & Flow - Complete Visual Summary

This document provides a **complete visual overview** of the entire Wazuh flow from installation to runtime operation.

---

## 🟢 Step 1: Pre-Installation (You are here with the code)

```
┌────────────────────────────────────────────────────────────┐
│  Your machine with Wazuh source code downloaded            │
│  Location: /Users/shivam/wazuh/                            │
│                                                             │
│  ├─ /src/              ← C/C++ source code                 │
│  ├─ /api/              ← Python API                        │
│  ├─ /framework/        ← Python framework                  │
│  ├─ /etc/              ← Config templates                  │
│  ├─ /ruleset/          ← Detection rules                   │
│  ├─ install.sh         ← Installation script               │
│  ├─ CMakeLists.txt     ← Build config                      │
│  └─ README.md          ← Documentation                     │
│                                                             │
│  Status: Ready to install                                  │
└────────────────────────────────────────────────────────────┘
                            │
                            │ Run: ./install.sh
                            ▼
```

---

## 🟡 Step 2: Installation Phase (15-90 minutes)

```
┌───────────────────────────────────────────────────────────┐
│ INSTALLATION PROCESS                                       │
├───────────────────────────────────────────────────────────┤
│                                                             │
│  ./install.sh                                              │
│         │                                                  │
│         ├─► Detect OS, check dependencies                 │
│         │   └─ CentOS? Ubuntu? Check gcc, CMake           │
│         │                                                  │
│         ├─► Ask user: Manager or Agent?                   │
│         │   └─ Response: "Manager"                        │
│         │                                                  │
│         ├─► Ask features: FIM? Rootcheck?                 │
│         │   └─ Response: "Yes to all"                     │
│         │                                                  │
│         ├─► Ask installation path                         │
│         │   └─ Response: "/var/ossec"                    │
│         │                                                  │
│         └─► Start compilation                             │
│              │                                             │
│              ├─ make deps                                  │
│              │  └─ Download: OpenSSL, zlib, SQLite        │
│              │     Time: 5-10 min                         │
│              │     Size: ~500MB                           │
│              │                                             │
│              └─ make build TARGET=server                  │
│                 └─ Compile C/C++ source                   │
│                    Time: 15-80 min                        │
│                    Output: Binaries in /src/build/        │
│                                                             │
└───────────────────────────────────────────────────────────┘
                            │
                            ▼
```

---

## 🔵 Step 3: File Installation Phase (2 minutes)

```
┌───────────────────────────────────────────────────────────┐
│ FILE COPYING & CONFIGURATION                               │
├───────────────────────────────────────────────────────────┤
│                                                             │
│  Copy compiled binaries:                                   │
│  /src/build/   ────────► /var/ossec/bin/                 │
│  ├─ wazuh-remoted                                         │
│  ├─ wazuh-analysisd                                       │
│  ├─ wazuh-execd                                           │
│  ├─ wazuh-monitord                                        │
│  └─ (utilities)                                           │
│                                                             │
│  Copy configuration templates:                            │
│  /etc/   ────────► /var/ossec/etc/                       │
│  ├─ ossec.conf (customize as needed)                      │
│  ├─ internal_options.conf                                 │
│  └─ templates/                                            │
│                                                             │
│  Copy rules & decoders:                                    │
│  /ruleset/   ────────► /var/ossec/ruleset/              │
│  ├─ rules/ (4500+ rule files)                            │
│  ├─ decoders/ (100+ decoder files)                       │
│  ├─ sca/ (compliance policies)                           │
│  └─ lists/ (IOC lists)                                   │
│                                                             │
│  Create directory structure:                              │
│  /var/ossec/                                              │
│  ├─ bin/          ┌─ Binaries                             │
│  ├─ etc/          ├─ Configuration                        │
│  ├─ lib/          ├─ Libraries                            │
│  ├─ queue/        ├─ Data, alerts, inventory              │
│  ├─ logs/         ├─ Log files                            │
│  ├─ ruleset/      ├─ Rules & decoders                     │
│  └─ var/          └─ Runtime data                         │
│                                                             │
│  Create system users:                                      │
│  useradd wazuh (UID: 10000)                               │
│  groupadd wazuh (GID: 10000)                              │
│                                                             │
│  Set permissions:                                          │
│  chown root:root /var/ossec/bin/ (750)                    │
│  chown root:wazuh /var/ossec/queue/ (770)                 │
│  chown wazuh:wazuh /var/ossec/logs/ (750)                │
│                                                             │
│  Initialize databases:                                     │
│  mkdir /var/ossec/queue/db/                               │
│  sqlite3 init:                                             │
│  ├─ alerts.db (alert storage)                             │
│  ├─ agents.db (agent registry)                            │
│  └─ wazuh.db (metadata)                                   │
│                                                             │
│  Create init scripts (for auto-start):                     │
│  /etc/systemd/system/wazuh-manager.service                │
│  systemctl enable wazuh-manager                           │
│                                                             │
└───────────────────────────────────────────────────────────┘
                            │
                            ▼
                 Installation Complete!
                            │
                            │ Next: Configure & Start
                            ▼
```

---

## 🟣 Step 4: Configuration Phase (5-30 minutes)

```
┌───────────────────────────────────────────────────────────┐
│ CONFIGURING WAZUH FOR YOUR ENVIRONMENT                     │
├───────────────────────────────────────────────────────────┤
│                                                             │
│  Edit: /var/ossec/etc/ossec.conf                          │
│                                                             │
│  MANAGER Configuration:                                    │
│  ┌──────────────────────────────────────────────────┐     │
│  │ <global>                                         │     │
│  │   <email_from>wazuh@yourcompany.com</email_from>│     │
│  │   <smtp_server>localhost</smtp_server>           │     │
│  │   <email_notification>yes</notification>        │     │
│  │ <global>                                         │     │
│  │                                                  │     │
│  │ <remote>                                         │     │
│  │   <connection>secure</connection>                │     │
│  │   <port>1514</port>                              │     │
│  │   <protocol>tcp</protocol>                       │     │
│  │   <max_clients>10000</max_clients>               │     │
│  │ </remote>                                        │     │
│  │                                                  │     │
│  │ <rules>                                          │     │
│  │   <include>rules_config.xml</include>            │     │
│  │   <!-- References all rule files -->             │     │
│  │ </rules>                                         │     │
│  │                                                  │     │
│  │ <logging>                                        │     │
│  │   <log_format>json</log_format>                  │     │
│  │   <max_log_size>1000000</max_log_size>           │     │
│  │ </logging>                                       │     │
│  │                                                  │     │
│  │ <syscheck>  <!-- FIM Configuration -->           │     │
│  │   <frequency>86400</frequency> <!-- 1 day -->    │     │
│  │   <directories check_all="yes">                  │     │
│  │     /etc,/usr/bin,/usr/sbin                      │     │
│  │   </directories>                                 │     │
│  │   <directories check_all="yes">                  │     │
│  │     /bin,/sbin                                   │     │
│  │   </directories>                                 │     │
│  │   <ignore>/etc/mtab</ignore>                     │     │
│  │   <ignore>/etc/resolv.conf</ignore>              │     │
│  │ </syscheck>                                      │     │
│  │                                                  │     │
│  │ <rootcheck>                                      │     │
│  │   <disabled>no</disabled>                        │     │
│  │   <check_files>yes</check_files>                 │     │
│  │   <check_trojans>yes</check_trojans>             │     │
│  │   <check_dev>yes</check_dev>                     │     │
│  │ </rootcheck>                                     │     │
│  │                                                  │     │
│  │ <osquery>  <!-- Osquery integration (opt) -->    │     │
│  │   <disabled>yes</disabled>                       │     │
│  │ </osquery>                                       │     │
│  │                                                  │     │
│  │ </ossec_config>                                  │     │
│  └──────────────────────────────────────────────────┘     │
│                                                             │
│  OPTIONAL: Register Agents                                │
│  ┌──────────────────────────────────────┐                │
│  │ $ /var/ossec/bin/manage_agents -a    │                │
│  │                                      │                │
│  │ Output:                              │                │
│  │ Agent ID: 001                        │                │
│  │ Name: linux-web-server               │                │
│  │ IP: 192.168.1.100                    │                │
│  │ Key: qeH45k[+XD5gLhLSzHLdU...       │                │
│  │                                      │                │
│  │ (Copy this key to agent system)      │                │
│  └──────────────────────────────────────┘                │
│                                                             │
└───────────────────────────────────────────────────────────┘
                            │
                            ▼
                  Ready to Start Services
```

---

## 🟣 Step 5: Service Startup (30 seconds)

```
┌────────────────────────────────────────────────────────────┐
│ STARTING WAZUH SERVICES                                    │
├────────────────────────────────────────────────────────────┤
│                                                             │
│ $ /var/ossec/bin/wazuh-control start                       │
│                                                             │
│ MANAGER DAEMONS START (in order):                          │
│                                                             │
│ 1st: wazuh-remoted ─────► Listening on Port 1514           │
│      └─ Status: Ready for agent connections                │
│         Loaded keys from /var/ossec/etc/client.keys        │
│                                                             │
│ 2nd: wazuh-analysisd ──► Loading 4500+ rules              │
│      └─ Status: Ready to process events                    │
│         Memory: Decoders, Rules, Patterns                 │
│                                                             │
│ 3rd: wazuh-execd ──────► Loaded response scripts           │
│      └─ Status: Ready to execute countermeasures            │
│         Scripts: firewall-drop.sh, block-account.sh        │
│                                                             │
│ 4th: wazuh-monitord ──► Monitoring system health           │
│      └─ Status: Running daemon health checks               │
│         Check intervals: Every 5 seconds                   │
│                                                             │
│ Optional: wazuh-clusterd ─► (if clustering enabled)        │
│ Optional: wazuh-apid ────► (Python API on port 55000)      │
│                                                             │
│ All daemons started successfully!                          │
│ $ ps aux | grep wazuh                                       │
│ wazuh-remote  - listening 192.168.1.x:1514 tcp  LISTEN    │
│ wazuh-analysi - analyzing events              RUNNING     │
│ wazuh-execd   - ready for responses           RUNNING     │
│ wazuh-monitor - health checking               RUNNING     │
│                                                             │
│ Logs:                                                       │
│ Check: /var/ossec/logs/ossec.log                           │
│ $ tail -f /var/ossec/logs/ossec.log                         │
│ 2024-01-15 10:30:42 [*] System launched                    │
│ 2024-01-15 10:30:45 [*] Loaded X rules                     │
│ 2024-01-15 10:30:48 [*] Remoted daemon started             │
│                                                             │
└────────────────────────────────────────────────────────────┘
                            │
                            ▼
              ✓ MANAGER READY FOR AGENTS
                            │
                            │ Now start agents (on other machines)
                            ▼
```

---

## 🟢 Step 6: Agent Configuration & Start (on agent systems)

```
┌────────────────────────────────────────────────────────────┐
│ AGENT INSTALLATION (on each endpoint)                      │
├────────────────────────────────────────────────────────────┤
│                                                             │
│ Same installation as manager, but:                          │
│ $ ./install.sh                                              │
│ → Select: "Agent" (not Manager)                            │
│                                                             │
│ Edit: /var/ossec/etc/ossec.conf                            │
│                                                             │
│ AGENT Configuration:                                       │
│ ┌──────────────────────────────────────────────────┐      │
│ │ <client>                                         │      │
│ │   <server-ip>192.168.1.100</server-ip>         │      │
│ │   <server-hostname>wazuh-manager</server-host...│      │
│ │   <port>1514</port>                             │      │
│ │   <protocol>tcp</protocol>                      │      │
│ │   <config-profile>linux</config-profile>        │      │
│ │   <agent_name>linux-web-01</agent_name>         │      │
│ │   <agent_id>001</agent_id>  <!-- from manager -->       │
│ │ </client>                                        │      │
│ │                                                  │      │
│ │ <syscheck>  <!-- What to monitor -->            │      │
│ │   <frequency>3600</frequency> <!-- 1 hour -->   │      │
│ │   <directories check_all="yes">                 │      │
│ │     /etc,/usr/bin,/usr/sbin                     │      │
│ │   </directories>                                │      │
│ │ </syscheck>                                     │      │
│ │                                                  │      │
│ │ <rootcheck>                                     │      │
│ │   <disabled>no</disabled>                       │      │
│ │ </rootcheck>                                    │      │
│ │                                                  │      │
│ │ <logging>                                       │      │
│ │   <log_format>json</log_format>                 │      │
│ │ </logging>                                      │      │
│ │                                                  │      │
│ │ <localfile>  <!-- Which logs to collect -->    │      │
│ │   <log_format>syslog</log_format>              │      │
│ │   <location>/var/log/auth.log</location>        │      │
│ │ </localfile>                                    │      │
│ │                                                  │      │
│ │ <localfile>                                    │      │
│ │   <log_format>syslog</log_format>              │      │
│ │   <location>/var/log/syslog</location>          │      │
│ │ </localfile>                                    │      │
│ │                                                  │      │
│ │ (Add more log files as needed)                  │      │
│ │                                                  │      │
│ │ </ossec_config>                                 │      │
│ └──────────────────────────────────────────────────┘      │
│                                                             │
│ Copy agent key from manager:                               │
│ ┌──────────────────────────────────────────────────┐      │
│ │ 1. Get key from manager:                         │      │
│ │    /var/ossec/bin/manage_agents -e 001          │      │
│ │    Output: qeH45k[+XD5gLhLSzHLdU...            │      │
│ │                                                  │      │
│ │ 2. Paste into agent:                            │      │
│ │    /var/ossec/etc/client.keys (on agent system) │      │
│ │                                                  │      │
│ │ 3. Format:                                      │      │
│ │    001 linux-web-01 192.168.1.100              │      │
│ │    qeH45k[+XD5gLhLSzHLdUKGFn0jnHPrFwN...       │      │
│ │                                                  │      │
│ │ 4. Set permissions:                             │      │
│ │    chmod 640 /var/ossec/etc/client.keys         │      │
│ │    chown root:wazuh /var/ossec/etc/client.keys │      │
│ └──────────────────────────────────────────────────┘      │
│                                                             │
│ Start agent:                                                │
│ $ /var/ossec/bin/wazuh-control start                       │
│                                                             │
│ AGENT STARTUP:                                              │
│                                                             │
│ wazuh-agentd starts and:                                   │
│ ├─ Reads client.keys (agent ID + encryption key)           │
│ ├─ Connects to manager on 192.168.1.100:1514               │
│ ├─ Sends: Agent ID "001"                                   │
│ ├─ Manager validates key matches                           │
│ ├─ Manager sends: ACK                                      │
│ ├─ Connection established (encrypted TLS)                  │
│ │                                                             │
│ ├─ Spawns logcollector:                                    │
│ │  └─ Starts reading: /var/log/auth.log, /var/log/syslog  │
│ │                                                             │
│ ├─ Spawns syscheckd:                                       │
│ │  └─ Creates baseline of /etc/, /usr/bin/ files           │
│ │  └─ Ready to detect changes                              │
│ │                                                             │
│ ├─ Spawns rootcheck:                                       │
│ │  └─ Ready to scan for malware                            │
│ │                                                             │
│ └─ Spawns syscollector:                                    │
│    └─ Ready to collect system inventory                    │
│                                                             │
│ Status: Agent connected to manager                         │
│ $ /var/ossec/bin/wazuh-control status                      │
│ [*] Wazuh agent is running                                 │
│     Agent name: linux-web-01                               │
│     Agent ID  : 001                                        │
│     Manager IP: 192.168.1.100:1514                         │
│     Status    : Connected                                  │
│                                                             │
└────────────────────────────────────────────────────────────┘
                            │
                            ▼
          ✓ AGENT CONNECTED & SENDING DATA
```

---

## 🟠 Step 7: Continuous Operation (24/7)

```
┌────────────────────────────────────────────────────────────┐
│ SYSTEM RUNNING - REAL-TIME SECURITY MONITORING              │
├────────────────────────────────────────────────────────────┤
│                                                             │
│ EVERY 1-10 SECONDS (Agent):                               │
│ • Logcollector reads new log lines                         │
│ • Buffer accumulates events                                │
│ • Send encrypted payload to manager                        │
│                                                             │
│ EVERY 10-30 SECONDS (Agent):                              │
│ • New buffer cycle starts                                  │
│ • Continue collecting logs                                 │
│                                                             │
│ EVERY 1 HOUR (Agent):                                      │
│ • syscheckd scans monitored directories                    │
│ • Compares file hashes to baseline                         │
│ • Detects any file changes                                 │
│ • Sends FIM alerts for modifications                       │
│                                                             │
│ EVERY 12+ HOURS (Agent):                                   │
│ • rootcheck scans for trojans/malware                      │
│ • Checks for rootkits                                      │
│ • Looks for suspicious processes                           │
│                                                             │
│ REAL-TIME (Manager):                                       │
│ • remoted receives agent packets                           │
│ • Decrypts and decompresses                                │
│ • Queues for analysis                                      │
│                                                             │
│ CONTINUOUS (Manager):                                      │
│ • analysisd processes events from queue                    │
│ • Decodes log formats (SSH, Apache, etc)                   │
│ • Matches patterns against 4500+ rules                    │
│ • Generates alerts for matches                             │
│ • Sends alerts to storage (local + remote)                 │
│                                                             │
│ EVERY 5 SECONDS (Manager):                                │
│ • monitord checks all daemon health                        │
│ • Restarts any dead processes                              │
│ • Monitors disk space                                      │
│ • Rotates log files if too large                           │
│                                                             │
│ CONTINUOUS OUTPUT:                                          │
│ ┌────────────────────────────────────────┐               │
│ │ Alerts Generated to:                   │               │
│ │ ├─ /var/ossec/logs/alerts/alerts.json │               │
│ │ │  (Local file)                        │               │
│ │ ├─ /var/ossec/queue/db/alerts.db      │               │
│ │ │  (SQLite database)                   │               │
│ │ ├─ Elasticsearch (if configured)       │               │
│ │ │  (Remote indexing & search)          │               │
│ │ ├─ REST API (/api/agents/*/events)    │               │
│ │ │  (Programmatic access)               │               │
│ │ └─ Wazuh Dashboard                     │               │
│ │    (Web visualization)                 │               │
│ │                                        │               │
│ │ Example Alert:                         │               │
│ │ {                                      │               │
│ │   "timestamp": "2024-01-15T10:30:42",  │               │
│ │   "agent_id": "001",                   │               │
│ │   "rule_id": "5710",                   │               │
│ │   "rule_level": "5",                   │               │
│ │   "description": "SSH brute force",     │               │
│ │   "srcip": "192.168.1.100",             │               │
│ │   "user": "admin",                      │               │
│ │   "action": "alert"                     │               │
│ │ }                                      │               │
│ └────────────────────────────────────────┘               │
│                                                             │
└────────────────────────────────────────────────────────────┘
              │
              ▼
   ┌──────────────────────────────────────┐
   │  SECURITY TEAM VISIBILITY            │
   ├──────────────────────────────────────┤
   │                                      │
   │ Dashboard Views:                      │
   │ ├─ Real-time Alert Stream            │
   │ ├─ Risk Indicators                   │
   │ ├─ Top Agents by Alert Count         │
   │ ├─ Rule Patterns & Trends            │
   │ ├─ Compliance Scores                 │
   │ ├─ Threat Timeline                   │
   │ ├─ Geographic Heat Maps              │
   │ └─ MITRE ATT&CK Mapping              │
   │                                      │
   │ Actions Available:                    │
   │ ├─ Drill down to full alert details  │
   │ ├─ Search across all alerts          │
   │ ├─ Generate compliance reports       │
   │ ├─ Track agent status                │
   │ ├─ Create custom rules                │
   │ ├─ Execute active responses manually │
   │ └─ Configure agent settings          │
   │                                      │
   │ Alerts with Actions:                  │
   │ ├─ High severity: Email notification │
   │ ├─ Brute force: Block IP (automatic) │
   │ ├─ Rootkit: Alert & isolate agent    │
   │ ├─ File change: Create incident      │
   │ └─ Config drift: Restore baseline    │
   │                                      │
   └──────────────────────────────────────┘
```

---

## 📊 Complete Lifecycle Summary

```
┌─────────────────────────────────────────────────────────┐
│         WAZUH COMPLETE LIFECYCLE                        │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Phase          Time              Action                │
│  ────────────────────────────────────────────────        │
│                                                          │
│  Phase 1:       Once              Installation          │
│  Install        (1-3 hours)       • Compile source      │
│                                   • Copy files          │
│                                   • Create dirs         │
│                                   • Init databases      │
│                 ✓ Success                               │
│                                                          │
│  Phase 2:       Once              Configuration         │
│  Configure      (5-30 min)        • Edit ossec.conf    │
│                                   • Register agents    │
│                                   • Copy keys          │
│                 ✓ Success                               │
│                                                          │
│  Phase 3:       Once              Startup               │
│  Start Services (1 min)           • Manager daemons    │
│                                   • Agent daemons      │
│                                   • Test connectivity  │
│                 ✓ Success                               │
│                                                          │
│  Phase 4:       Continuous        Operation             │
│  Monitor        (24/7)            • Collect data       │
│  Events         Forever           • Analyze logs       │
│                                   • Generate alerts    │
│                                   • Store results      │
│                                   • Show dashboard     │
│                 ✓ Always Running                        │
│                                                          │
│  Phase 5:       On demand         Maintenance           │
│  Maintenance    (As needed)       • Update rules       │
│                                   • Add agents         │
│                                   • Tune settings      │
│                                   • Upgrade version    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 🎯 Example Workflow: "Failed SSH Login"

```
Scenario:
Attacker tries to SSH into linux-web-01 with wrong password

┌─ 10:30:42 - ATTACK ATTEMPT
├─ SSH server rejects login
├─ Writes to /var/log/auth.log:
│  "Jan 15 10:30:42 ssh[1234]: Failed password for 
│   invalid user admin from 192.168.1.100 port 54321"
│
├─ [Agent: linux-web-01]
├─ 10:30:42.100 - logcollector reads new line
├─ Adds metadata: agent_id=001, source=/var/log/auth.log
├─ Buffers event in memory
├─ Waits for buffer to reach threshold (or 10 seconds)
│
├─ 10:30:52 - Buffer ready (10 seconds elapsed)
├─ Encrypt: AES with key from client.keys
├─ Compress: zlib (85% smaller)
├─ Send: 192.168.1.100:1514 (TCP encrypted)
├─ Clear buffer, start new cycle
│
├─ [Manager Network]
├─ 10:30:52.050 - Packet received (network latency)
│
├─ [Manager: wazuh-remoted]
├─ Receive encrypted packet from agent 001
├─ Lookup key for agent 001 in /var/ossec/etc/client.keys
├─ Decrypt payload
├─ Decompress data
├─ Extract events
├─ Add server metadata (received_at, server_hostname)
├─ Queue event for analysis
│
├─ [Manager: wazuh-analysisd]
├─ 10:30:52.100 - Get event from queue
├─ Apply SSH decoder:
│  Decode: srcip, user, action, status, protocol
├─ Check against rules:
│  Rule 5710: "SSH failed login" - MATCH!
├─ Check if action needed:
│  Is there a frequency rule for multiple failures?
│  Current count in last 10 min: 1 (not triggered yet)
├─ Generate Alert:
│  {
│    "id": "1705318252.12345",
│    "timestamp": "2024-01-15T10:30:52Z",
│    "agent_id": "001",
│    "agent_name": "linux-web-01",
│    "rule_id": "5710",
│    "rule_level": "5",
│    "description": "SSH failed login",
│    "srcip": "192.168.1.100",
│    "user": "admin",
│    "full_log": "Jan 15 10:30:42 ssh[1234]: ...",
│    "action": "alert"
│  }
├─ Store alert:
│  Write to: /var/ossec/logs/alerts/alerts.json
│  Insert in: /var/ossec/queue/db/alerts.db
├─ Send to Elasticsearch (if configured)
├─ Check active response: None configured for level 5
│
├─ 10:30:52.200 - SECURITY TEAM
├─ Dashboard automatically updates
├─ Alert appears in "Real-time Alerts" widget
├─ Rule: SSH failed login - Count: 1
├─ Source IP: 192.168.1.100
├─ User: admin
├─ Agent: linux-web-01
│
├─ 10:31:00 - ATTACKER TRIES AGAIN (3 more times in next 30 sec)
├─ 4 more failed login attempts recorded
├─ Each sent to manager
├─ Each triggers rule 5710 alert
├─ frequencycount now: 4
│
├─ 10:31:30 - ONE MORE FAILURE (5th in 10 minutes)
├─ Alert for rule 5710 generated
├─ ALSO triggers rule 5711: "SSH brute force"
│  Condition: 5 failures in 10 minutes
│  Level: 10 (HIGH PRIORITY)
│
├─ 10:31:30.500 - CRITICAL ALERT!
├─ Dashboard shows HIGH severity in red
├─ Email sent to security@yourcompany.com
├─ "ALERT: SSH Brute Force Attack on linux-web-01"
├─ Includes: 5 failed attempts from 192.168.1.100
│
├─ 10:31:30.600 - ACTIVE RESPONSE
├─ Rule 5711 has configured action:
│  <active-response>yes</active-response>
│  <command>firewall-block-ip.sh</command>
├─ execd daemon executes:
│  firewall-block-ip.sh -a add -i 192.168.1.100
├─ Script blocks IP at firewall level
├─ All packets from 192.168.1.100 DROP
│
├─ 10:31:31 - INCIDENT CREATED
├─ SOAR system auto-creates ticket
├─ Assigns to security team
├─ Title: "Brute Force Attack Detected"
├─ Details: Linked to linux-web-01, rule 5711
│
├─ 10:31:35 - ATTACKER BLOCKED
├─ Next SSH attempt fails immediately
├─ Connection refused (blocked by firewall)
├─ No more alerts generated
│
├─ 10:32:00 - INVESTIGATION
├─ Security team logs in to dashboard
├─ Sees 5 failed login attempts from 192.168.1.100
├─ Clicks "View in Elasticsearch"
├─ Searches for more context
├─ Checks if same IP attacked other agents
├─ Reviews pattern of attacks
│
├─ 10:35:00 - REMEDIATION
├─ Security team confirms blocking
├─ Updates firewall rule to : rule permanently
├─ Creates YARA rule to block malware
├─ Notifies incident response team
├─ Opens official security incident
│
├─ 24:00:00 - REPORTING
├─ Alert remains in database forever
├─ Appears in security reports
├─ Included in compliance audit (PCI DSS)
├─ Shows on dashboard "Threat Timeline"
├─ Part of historical analysis
│
└─ EVENT COMPLETE: Detected, Alerted, Responded, Documented
```

---

## 🏁 Final Summary

```
┌─────────────────────────────────────────────────────────┐
│           WAZUH: FROM CODE TO SECURITY                  │
├─────────────────────────────────────────────────────────┤
│                                                          │
│ 1. INSTALL     (1-3 hours, one-time)                   │
│    - Compile C/C++ source code                         │
│    - Create directory structure                         │
│    - Initialize configuration                          │
│    - Ready to run                                      │
│                                                          │
│ 2. CONFIGURE   (10-30 minutes, one-time)               │
│    - Customize for your environment                     │
│    - Register agents                                    │
│    - Set up integrations                                │
│    - Ready to start                                    │
│                                                          │
│ 3. START       (1 minute, once then auto)              │
│    - Manager daemons listen for agents                 │
│    - Agents connect and authenticate                   │
│    - Data flow begins                                  │
│    - Continuously running                              │
│                                                          │
│ 4. MONITOR     (24/7 continuous)                       │
│    ├─ Collection: Agents send data                     │
│    ├─ Analysis: Manager processes events               │
│    ├─ Detection: Rules trigger alerts                  │
│    ├─ Response: Automatic countermeasures              │
│    └─ Visibility: Dashboard shows threats              │
│                                                          │
│ 5. INVESTIGATE (On demand)                             │
│    - Review specific alerts                             │
│    - Search threat intelligence                         │
│    - Create incident tickets                           │
│    - Perform forensic analysis                         │
│                                                          │
│ 6. RESPOND     (Automated + Manual)                     │
│    ├─ Auto: Block IPs, kill processes                  │
│    ├─ Manual: Execute commands, restore config         │
│    ├─ Escalate: Create tickets, notify teams           │
│    └─ Document: Store forensic data                    │
│                                                          │
│ 7. IMPROVE     (Continuous)                            │
│    ├─ Update rules based on threats                     │
│    ├─ Fine-tune alert thresholds                       │
│    ├─ Add new agents                                   │
│    ├─ Expand monitoring                                │
│    └─ Integrate new systems                            │
│                                                          │
│ RESULT:                                                  │
│ 24/7 automated security monitoring platform providing:   │
│ ✓ Threat detection                                      │
│ ✓ Incident response                                     │
│ ✓ Compliance verification                              │
│ ✓ Forensic investigation                               │
│ ✓ Risk assessment                                       │
│ ✓ Real-time alerts                                     │
│ ✓ Historical analysis                                  │
│ ✓ Threat intelligence                                  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

**Now you understand the complete flow of Wazuh from installation through continuous operation!**

For implementation details, refer to:
- [INSTALLATION_FLOW.md](INSTALLATION_FLOW.md) - Deep dive on installation phases
- [RUNTIME_DATA_FLOW.md](RUNTIME_DATA_FLOW.md) - Deep dive on operational data flow
- [PRACTICAL_EXAMPLES.md](PRACTICAL_EXAMPLES.md) - Actual commands and configurations
