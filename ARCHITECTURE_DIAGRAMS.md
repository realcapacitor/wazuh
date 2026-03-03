# Wazuh System Architecture - Visual Diagrams

## 1. Complete System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        WAZUH MONITORING PLATFORM                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                          ╔════════════════════════════╗                      │
│                          ║   SECURITY ANALYST (WUI)   ║                      │
│                          ║   - View Alerts            ║                      │
│                          ║   - Investigate Events     ║                      │
│                          ║   - Manage Agents          ║                      │
│                          ╚════════════════════════════╝                      │
│                                      │                                       │
│                                      ▼                                       │
│                   ┌──────────────────────────────────────┐                  │
│                   │   WAZUH DASHBOARD (React/Angular)   │                  │
│                   │   Port: 5601 (Kibana) / 443 (Dashboard)               │
│                   └──────────────────┬───────────────────┘                  │
│                                      │                                       │
│        ┌─────────────────────────────┼─────────────────────────────┐       │
│        ▼                             ▼                             ▼       │
│  ┌──────────────┐          ┌──────────────┐          ┌──────────────┐    │
│  │ Elasticsearch│          │  Wazuh API   │          │   Wazuh DB   │    │
│  │(Port 9200)   │          │ (Python)     │          │ (Agent Stats)│    │
│  │- Alerts      │          │ (Port 55000) │          │ - Inventory  │    │
│  │- Dashboards  │          │ - REST calls │          │ - Syscalls   │    │
│  └──────────────┘          └──────┬───────┘          └──────────────┘    │
│        ▲                           │                       ▲               │
│        │                    ┌──────▼──────┐                │               │
│        │                    │   Wazuh     │                │               │
│        │                    │  Framework  │                │               │
│        │                    │   (Python)  │                │               │
│        │                    │             │                │               │
│        │        ┌──────────►│ - Agent.py  │◄──────────┐   │               │
│        │        │           │ - Rules.py  │           │   │               │
│        │        │           │ - Cluster   │           │   │               │
│        │        │           │ - Security  │           │   │               │
│        │        │           │ - RBAC      │           │   │               │
│        │        │           └──────┬──────┘           │   │               │
│        │        │                  │                  │   │               │
│        │        │        ┌─────────▼──────────┐       │   │               │
│        │        │        │  Wazuh Manager     │       │   │               │
│        │        │        │  (Port 1514, 514)  │       │   │               │
│        │        │        │                    │       │   │               │
│        │        │   ┌───►│ - remoted (C/C++)  │       │   │               │
│        │        │   │    │ - analysisd (C/C++)│───┐   │   │               │
│        │        │   │    │ - monitord         │   │   │   │               │
│        │        │   │    │ - execd            │   │   │   │               │
│        │        │   │    │ - logcollector     │   │   │   │               │
│        │        │   │    └────────────────────┘   │   │   │               │
│        │        │   │                            └──┬─┼──►└───────────────┤
│        └────────┼───┘                              │ │                      │
│                 │                                  │ │                      │
│                 │                        ╔═════════▼═▼════════╗            │
│                 │                        ║   ALERT STORAGE    ║            │
│                 │                        ║   - alertsd.json   ║◄───────────┤
│                 │                        ║   - alerts.log     ║            │
│                 │                        ║   - Elasticsearch  ║            │
│                 │                        ║   - Wazuh-DB       ║            │
│                 │                        ╚════════════════════╝            │
│                 │                                  ▲                        │
│                 │                                  │                        │
│                 │                                  │                        │
│       ┌─────────┼──────────────────────────────────┤                       │
│       ▼         ▼                                  │                       │
│   ┌──────────────────────┐  ┌───────────────────┐│                       │
│   │  SECURITY RULES &    │  │  AGENT RESPONSES  ││                       │
│   │  CONFIGURATION       │  │  - Block IP       ││                       │
│   │  - Decoders          │  │  - Kill Process   ││                       │
│   │  - Rules             │  │  - Run Commands   ││                       │
│   │  - SCA Policies      │  │  - Firewall Rules ││                       │
│   │  - Active Response   │  │  - Revoke Access  ││                       │
│   │    Scripts           │  │                   ││                       │
│   └──────────────────────┘  └───────────────────┘│                       │
│                                                   │                       │
│   ════════════════════════════════════════════════════════════════════   │
│                   ENCRYPTED COMMUNICATION CHANNEL                           │
│   ════════════════════════════════════════════════════════════════════   │
│                                                   │                       │
│       ┌───────────────────────────────────────────┤                       │
│       │                                           │                       │
│   ╔═══▼═══════════════════╗  ╔═══════════════════▼══╗                    │
│   ║   WAZUH AGENTS        ║  ║  CLOUD INTEGRATIONS  ║                    │
│   ║   (Multiple systems)  ║  ║  - AWS Integration   ║                    │
│   ║                       ║  ║  - Azure Integration ║                    │
│   ║ Client-side processes:║  ║  - GCP Integration   ║                    │
│   ║ - wazuh-agentd       ║  ║  - GitHub            ║                    │
│   ║ - logcollector       ║  ║  - Office 365        ║                    │
│   ║ - syscheckd (FIM)     ║  ║                      ║                    │
│   ║ - rootcheck          ║  ║  (/wodles/*)         ║                    │
│   ║ - osquery (optional)  ║  ║                      ║                    │
│   ╚═══════════┬═══════════╝  ╚═══════════┬══════════╝                    │
│               │                           │                               │
│   ┌───────────┼───────────────────────────┤                               │
│   │           │                           │                               │
│   ▼           ▼                           ▼                               │
│   
│ ╔═══════════════════╗ ╔════════════════╗ ╔════════════════╗              │
│ ║ MONITORED SOURCES ║ ║ CLOUD SERVICES ║ ║ OTHER SOURCES  ║              │
│ ║ (Agent-side)      ║ ║ (Cloud APIs)   ║ ║                ║              │
│ ║                   ║ ║                ║ ║                ║              │
│ ║ - System Logs     ║ ║ - AWS APIs     ║ ║ - Syslog       ║              │
│ ║ - App Logs        ║ ║ - Azure APIs   ║ ║ - Network      ║              │
│ ║ - File System     ║ ║ - GCP APIs     ║ ║ - Custom APIs  ║              │
│ ║ - Registry        ║ ║ - GitHub API   ║ ║                ║              │
│ ║   (Windows)       ║ ║ - Office 365   ║ ║                ║              │
│ ║ - Network Traffic ║ ║                ║ ║                ║              │
│ ║ - Package Info    ║ ║                ║ ║                ║              │
│ ║ - Processes       ║ ║                ║ ║                ║              │
│ ╚═══════════════════╝ ╚════════════════╝ ╚════════════════╝              │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Data Flow Through Detection Pipeline

```
┌──────────────────────────────────────────────────────────────────┐
│                      AGENT DATA COLLECTION                       │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Multiple Sources (logs, files, registry, system, cloud APIs)   │
│           ▼                                                      │
│  ┌───────────────────────────────────────────┐                 │
│  │      LOG PARSING & NORMALIZATION          │                 │
│  │  (logcollector daemon)                     │                 │
│  │  - Read logs                              │                 │
│  │  - Parse multiline events                 │                 │
│  │  - Normalize timestamps                   │                 │
│  └───────────────────────────────────────────┘                 │
│           ▼                                                      │
│  ┌───────────────────────────────────────────┐                 │
│  │    COMPRESSION & BUFFERING                │                 │
│  │  (agent-side buffer)                      │                 │
│  │  - Compress data                          │                 │
│  │  - Buffer for optimization                │                 │
│  │  - Handle disconnections                  │                 │
│  └───────────────────────────────────────────┘                 │
│           ▼                                                      │
│  ┌───────────────────────────────────────────┐                 │
│  │  ENCRYPTION & AUTHENTICATION              │                 │
│  │  (TLS/SSL + shared secret)                │                 │
│  │  - AES/SSL encryption                     │                 │
│  │  - Key management                         │                 │
│  │  - Agent verification                     │                 │
│  └───────────────────────────────────────────┘                 │
│           ▼                                                      │
│  ┌───────────────────────────────────────────┐                 │
│  │  SECURE TRANSMISSION TO MANAGER           │                 │
│  │  (TCP/UDP Port 1514)                      │                 │
│  │  - Network transmission                   │                 │
│  │  - Failover handling                      │                 │
│  └───────────────────────────────────────────┘                 │
│           ▼                                                      │
└──────────────────────────────────────────────────────────────────┘
                            │
                            │
    ┌───────────────────────▼───────────────────────┐
    │        MANAGER DATA RECEPTION                  │
    ├───────────────────────────────────────────────┤
    │                                               │
    │  ┌──────────────────────────────────────┐   │
    │  │  REMOTED (remoted daemon - C/C++)     │   │
    │  │  - TLS/SSL server                    │   │
    │  │  - Receive encrypted agent data     │   │
    │  │  - Decrypt and decompress           │   │
    │  │  - Verify agent authentication      │   │
    │  │  - Queue events                     │   │
    │  └──────────────────────────────────────┘   │
    │           ▼                                  │
    │  ┌──────────────────────────────────────┐   │
    │  │  EVENT BUFFERING & QUEUING           │   │
    │  │  - Store in memory queue             │   │
    │  │  - Aggregate events                  │   │
    │  │  - Handle burst traffic              │   │
    │  └──────────────────────────────────────┘   │
    │           ▼                                  │
    └──────────────────────────────────────────────┘
                            │
                            │
    ┌───────────────────────▼──────────────────────┐
    │        ANALYSIS & RULE MATCHING               │
    ├───────────────────────────────────────────────┤
    │                                               │
    │  ┌──────────────────────────────────────┐   │
    │  │ 1. DECODING (Extract log fields)     │   │
    │  │    - Apply decoders rules            │   │
    │  │    - Extract: source, destination   │   │
    │  │    - Extract: user, action, status  │   │
    │  │    - Extract: timestamp, severity   │   │
    │  └──────────────────────────────────────┘   │
    │           ▼                                  │
    │  ┌──────────────────────────────────────┐   │
    │  │ 2. RULE MATCHING (Detection Engine)  │   │
    │  │    - Match against rules             │   │
    │  │    - Check conditions/thresholds     │   │
    │  │    - Calculate rule level (priority) │   │
    │  │    - Find matching rule ID           │   │
    │  │    - Apply frequency/timeframe rules │   │
    │  └──────────────────────────────────────┘   │
    │           ▼                                  │
    │  ┌──────────────────────────────────────┐   │
    │  │ 3. RULE CHAIN (Rules can trigger     │   │
    │  │    other rules)                      │   │
    │  │    - Pass matched data to next rule  │   │
    │  │    - Correlation                     │   │
    │  │    - Complex threat detection        │   │
    │  └──────────────────────────────────────┘   │
    │           ▼                                  │
    │  ┌──────────────────────────────────────┐   │
    │  │ 4. POSTCONDITION TESTS               │   │
    │  │    - Ignore patterns                 │   │
    │  │    - Whitelist checks                │   │
    │  │    - Context evaluation              │   │
    │  └──────────────────────────────────────┘   │
    │           ▼                                  │
    └──────────────────────────────────────────────┘
                            │
                            │ ( If rule match )
                            │
    ┌───────────────────────▼─────────────────────┐
    │        ALERT GENERATION & STORAGE            │
    ├──────────────────────────────────────────────┤
    │                                              │
    │  ┌─────────────────────────────────────┐   │
    │  │ Create Alert Object with:           │   │
    │  │ - Timestamp                         │   │
    │  │ - Agent ID                          │   │
    │  │ - Rule ID & Name                    │   │
    │  │ - Log source                        │   │
    │  │ - Severity/Level                    │   │
    │  │ - Decoded fields                    │   │
    │  │ - Matched rule chain                │   │
    │  │ - Full log text                     │   │
    │  │ - Source & Destination IPs/Users    │   │
    │  └─────────────────────────────────────┘   │
    │           ▼                                 │
    │  ┌─────────────────────────────────────┐   │
    │  │  STORE IN LOCAL DATABASE            │   │
    │  │  (Wazuh-DB)                        │   │
    │  │  - alerts.json                      │   │
    │  │  - agents.db (inventory)            │   │
    │  └─────────────────────────────────────┘   │
    │           ▼                                 │
    │  ┌─────────────────────────────────────┐   │
    │  │  SEND TO ELASTICSEARCH              │   │
    │  │  - Remote Elasticsearch cluster    │   │
    │  │  - Indexed for fast searching       │   │
    │  │  - Retention policies               │   │
    │  └─────────────────────────────────────┘   │
    │           ▼                                 │
    │  ┌─────────────────────────────────────┐   │
    │  │  CHECK ACTIVE RESPONSE              │   │
    │  │  - Rule has action?                 │   │
    │  │  - Response enabled?                │   │
    │  │  - Thresholds met?                  │   │
    │  └─────────────────────────────────────┘   │
    │           ▼                                 │
    └──────────────────────────────────────────────┘
                            │
                            │ ( If active response triggered )
                            │
    ┌───────────────────────▼──────────────────────┐
    │        ACTIVE RESPONSE EXECUTION              │
    ├──────────────────────────────────────────────┤
    │                                              │
    │  ┌─────────────────────────────────────┐   │
    │  │ execd Daemon (os_execd)             │   │
    │  │ - Determine target (agent/manager)  │   │
    │  │ - Execute response script           │   │
    │  │ - Possible actions:                 │   │
    │  │   • Block source IP                 │   │
    │  │   • Allow source IP                 │   │
    │  │   • Kill process                    │   │
    │  │   • Restart service                 │   │
    │  │   • Run custom command              │   │
    │  │   • Firewall rules                  │   │
    │  │   • File operations                 │   │
    │  └─────────────────────────────────────┘   │
    │           ▼                                 │
    │  ┌─────────────────────────────────────┐   │
    │  │ Agent Executes Response             │   │
    │  │ - Receives command from manager     │   │
    │  │ - Executes on local system          │   │
    │  │ - Logs response action              │   │
    │  │ - Reports back to manager           │   │
    │  └─────────────────────────────────────┘   │
    │           ▼                                 │
    │  ┌─────────────────────────────────────┐   │
    │  │ Response Logged as Alert             │   │
    │  │ - Active response triggered          │   │
    │  │ - Action taken                       │   │
    │  │ - Status recorded                    │   │
    │  └─────────────────────────────────────┘   │
    │                                              │
    └──────────────────────────────────────────────┘
                            │
                            │
    ┌───────────────────────▼──────────────────────┐
    │        VISUALIZATION & ANALYSIS               │
    ├──────────────────────────────────────────────┤
    │                                              │
    │  Alerts available in:                       │
    │  1. Wazuh Dashboard (Web UI)                │
    │  2. Elasticsearch Kibana                    │
    │  3. Wazuh API queries                       │
    │  4. Local alert database                    │
    │  5. Custom integrations (SIEM, etc)         │
    │                                              │
    └──────────────────────────────────────────────┘
```

---

## 3. Component Interaction Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    WAZUH CORE ARCHITECTURE                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│                   [Web UI / Dashboard]                              │
│                     (Kibana/React)                                  │
│                           │                                         │
│                           ▼                                         │
│   ┌────────────────────────────────────────────────────┐          │
│   │           WAZUH API Layer (Python)                 │          │
│   │  /api/agents, /api/rules, /api/alerts, etc...     │          │
│   │                                                    │          │
│   │  Authentication → RBAC → Request Processing       │          │
│   └────────────────┬──────────────────────────────────┘          │
│                    │                                              │
│                    ▼                                              │
│   ┌────────────────────────────────────────────────────┐          │
│   │      WAZUH FRAMEWORK (Python Layer)                │          │
│   │                                                    │          │
│   │  ┌──────────────────────────────────────────┐    │          │
│   │  │ Manager Operations                       │    │          │
│   │  │ - agent.py (agent management)            │    │          │
│   │  │ - manager.py (manager functions)         │    │          │
│   │  │ - cluster.py (clustering)                │    │          │
│   │  │ - security.py (auth/RBAC)                │    │          │
│   │  │ - syscheck.py (FIM queries)              │    │          │
│   │  │ - rootcheck.py (rootkit info)            │    │          │
│   │  │ - active_response.py (AR execution)      │    │          │
│   │  │ - mitre.py (MITRE ATT&CK mappings)       │    │          │
│   │  └──────────────────────────────────────────┘    │          │
│   │                     │                             │          │
│   │                     ▼                             │          │
│   │  ┌──────────────────────────────────────────┐    │          │
│   │  │ Core C/C++ Daemons                       │    │          │
│   │  │                                          │    │          │
│   │  └──────────────────────────────────────────┘    │          │
│   │                                                    │          │
│   └────────────────────────────────────────────────────┘          │
│                    │                                              │
│    ┌───────────────┼────────────────┬──────────────┐             │
│    │               │                │              │             │
│    ▼               ▼                ▼              ▼             │
│ ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐      │
│ │ remoted  │  │  exec    │  │monitored │  │   Analysis  │      │
│ │(C/C++)   │  │   d      │  │(C/C++)   │  │  Daemon(C++)│      │
│ │          │  │(C/C++)   │  │          │  │             │      │
│ │Port 1514 │  │          │  │Monitor   │  │- Decoding   │      │
│ │          │  │Executes  │  │services  │  │- Rule Match │      │
│ │- Receive │  │active    │  │- Status  │  │- Alerts     │      │
│ │  agents  │  │response  │  │- Startup │  │- Events     │      │
│ │- Decrypt │  │scripts   │  │- Health  │  │             │      │
│ │- Queue   │  │- Block   │  │- Control │  └──────────────┘      │
│ │  events  │  │  IP      │  │ loop     │                        │
│ │- Store   │  │- Kill    │  │          │                        │
│ │  queue   │  │  proc    │  └──────────┘                        │
│ └──────────┘  └──────────┘                                      │
│    │                                                             │
│    └─────────────────────┬──────────────────────────────────    │
│                          ▼                                       │
│        ┌──────────────────────────────┐                         │
│        │  Database Layer              │                         │
│        │  (Wazuh-DB / SQLite)         │                         │
│        │                              │                         │
│        │ - alerts.json                │                         │
│        │ - agents.db                  │                         │
│        │ - metadata                   │                         │
│        │ - inventory                  │                         │
│        └──────────────────────────────┘                         │
│                    ▲                                             │
│                    │                                             │
│        ┌───────────┴──────────────┐                             │
│        ▼                          ▼                             │
│  ┌──────────────┐        ┌──────────────────┐                 │
│  │ Local SQL DB │        │ Elasticsearch    │                 │
│  │              │        │ (Remote)         │                 │
│  │ SQLite/MySQL │        │                  │                 │
│  │ PostgreSQL   │        │ Alerts indexed   │                 │
│  │              │        │ for searching    │                 │
│  │ Fast local   │        │ & analytics      │                 │
│  │ queries      │        │                  │                 │
│  └──────────────┘        └──────────────────┘                 │
│        ▲                          ▲                             │
│        │                          │                             │
│        └───────┬──────────────────┘                             │
│               /│\                                               │
│              / │ \                 ← Queries, Searches         │
│             /  │  \                                             │
│            /   │   \                                            │
│           /    │    \ Agent Communication (Port 1514)           │
│          /     │     \   ╔─────────────────────╗               │
│         /      │      └──║  WAZUH AGENTS       ║               │
│        /       │         ║                     ║               │
│       /        │         ║ - logcollector      ║               │
│      /         │         ║ - syscheckd         ║               │
│     /          │         ║ - rootcheck         ║               │
│    /           │         ║ - wazuh-agentd      ║               │
│   /            │         ║                     ║               │
│  /             │         ║ (on all endpoints)  ║               │
│ /              │         ╚─────────────────────╝               │
│/               │                                                │
│               │                                                 │
│        Agents report:                                           │
│        - Logs → logcollector                                    │
│        - File changes → syscheckd (FIM)                        │
│        - System status → rootcheck                             │
│        - Inventory → syscollector                              │
│        - Each sends to manager via encrypted channel            │
│                                                                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. File Integrity Monitoring (FIM) Pipeline

```
┌──────────────────────────────────────────────────────┐
│           FILE INTEGRITY MONITORING (FIM)             │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────────────────────────────┐          │
│  │ AGENT SIDE (syscheckd daemon)        │          │
│  │                                      │          │
│  │ Config: <syscheck> sections in       │          │
│  │         ossec-agent.conf             │          │
│  │                                      │          │
│  │ ┌──────────────────────────────┐   │          │
│  │ │ 1. Scan Configuration        │   │          │
│  │ │    - Files to monitor        │   │          │
│  │ │    - Directories to watch    │   │          │
│  │ │    - Scan interval (hours)   │   │          │
│  │ │    - Real-time vs scheduled  │   │          │
│  │ │    - Ignore patterns         │   │          │
│  │ └──────────────────────────────┘   │          │
│  │           ▼                         │          │
│  │ ┌──────────────────────────────┐   │          │
│  │ │ 2. File System Scan          │   │          │
│  │ │    - Recursively scan dirs   │   │          │
│  │ │    - Real-time monitoring    │   │          │
│  │ │    - Calculate file hashes   │   │          │
│  │ │      (MD5, SHA1, SHA256)     │   │          │
│  │ │    - Record metadata:        │   │          │
│  │ │      • Size                  │   │          │
│  │ │      • Permissions           │   │          │
│  │ │      • Owner/Group           │   │          │
│  │ │      • Modified date         │   │          │
│  │ │      • Links                 │   │          │
│  │ │      • Inode                 │   │          │
│  │ └──────────────────────────────┘   │          │
│  │           ▼                         │          │
│  │ ┌──────────────────────────────┐   │          │
│  │ │ 3. Change Detection          │   │          │
│  │ │    - Compare with baseline   │   │          │
│  │ │    - Identify changes:       │   │          │
│  │ │      • Hash change           │   │          │
│  │ │      • Permissions change    │   │          │
│  │ │      • Ownership change      │   │          │
│  │ │      • Size change           │   │          │
│  │ │      • New file              │   │          │
│  │ │      • Deleted file          │   │          │
│  │ │    - Determine users/procs   │   │          │
│  │ │      who modified files      │   │          │
│  │ └──────────────────────────────┘   │          │
│  │           ▼                         │          │
│  │ ┌──────────────────────────────┐   │          │
│  │ │ 4. Generate FIM Events       │   │          │
│  │ │    Format:                   │   │          │
│  │ │    "Modified:" + filepath    │   │          │
│  │ │    Size changed from X to Y  │   │          │
│  │ │    Permissions: 755 → 777    │   │          │
│  │ │    MD5 hash changed          │   │          │
│  │ │    User change info (if)     │   │          │
│  │ └──────────────────────────────┘   │          │
│  │           ▼                         │          │
│  │ ┌──────────────────────────────┐   │          │
│  │ │ 5. Send to Manager           │   │          │
│  │ │    (via remoted)             │   │          │
│  │ └──────────────────────────────┘   │          │
│  │                                      │          │
│  └──────────────────────────────────────┘          │
│                                                      │
└──────────────────────────────────────────────────────┘
                    │
                    │ (FIM Events)
                    ▼
┌──────────────────────────────────────────────────────┐
│         MANAGER SIDE (Analysis)                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────────────────────────────┐          │
│  │ FIM Event Received                   │          │
│  │ Example: "Modified: /etc/passwd"     │          │
│  │          "Size: 1234 → 1240"         │          │
│  │          "Hash changed"              │          │
│  │          "User: root"                │          │
│  │          "Process: vipw"             │          │
│  └──────────────────────────────────────┘          │
│                    ▼                                │
│  ┌──────────────────────────────────────┐          │
│  │ Decode FIM Event                     │          │
│  │ - File path                          │          │
│  │ - Change type                        │          │
│  │ - Size delta                         │          │
│  │ - Hash values                        │          │
│  │ - User who made change               │          │
│  │ - Process involved                   │          │
│  │ - Timestamp                          │          │
│  └──────────────────────────────────────┘          │
│                    ▼                                │
│  ┌──────────────────────────────────────┐          │
│  │ Apply FIM Detection Rules            │          │
│  │                                      │          │
│  │ Rule examples:                       │          │
│  │ - File permission change on /etc/    │          │
│  │ - Binary file modified               │          │
│  │ - System library changed             │          │
│  │ - Unauthorized user modification     │          │
│  │ - Critical symlink changed           │          │
│  │ - Unauthorized deletion              │          │
│  │                                      │          │
│  │ Check against:                       │          │
│  │ - FIM rules (SCA, MITRE)             │          │
│  │ - Baseline                           │          │
│  │ - Threat intelligence                │          │
│  │ - Anomaly detection                  │          │
│  └──────────────────────────────────────┘          │
│                    ▼                                │
│  ┌──────────────────────────────────────┐          │
│  │ Generate Alert (if rule matched)     │          │
│  │                                      │          │
│  │ Alert fields:                        │          │
│  │ - Rule ID (e.g., 550011)             │          │
│  │ - Severity (Low to 15)               │          │
│  │ - File path                          │          │
│  │ - Change details                     │          │
│  │ - User/process info                  │          │
│  │ - Agent info                         │          │
│  │ - Timestamp                          │          │
│  │ - Events correlated                  │          │
│  └──────────────────────────────────────┘          │
│                    ▼                                │
│  ┌──────────────────────────────────────┐          │
│  │ Store FIM Alert                      │          │
│  │ - Local DB (alerts)                  │          │
│  │ - Elasticsearch (indexing)           │          │
│  │ - Available in Dashboard             │          │
│  └──────────────────────────────────────┘          │
│                                                      │
└──────────────────────────────────────────────────────┘
```

---

## 5. Agent Registration & Key Exchange

```
┌──────────────────────────────────────────────────────┐
│     AGENT REGISTRATION & ENROLLMENT PROCESS          │
├──────────────────────────────────────────────────────┤
│                                                      │
│  STEP 1: Request Agent Registration                │
│                                                      │
│  ┌──────────┐      Manager Tool (manage_agents)   │
│  │New Agent │                                      │
│  │  Client  │      $ manage_agents -a              │
│  │          │                                      │
│  │Hostname: │  ──────► Prompt user for:            │
│  │myhost01  │          - Agent name                │
│  │          │          - Agent IP/hostname         │
│  │IP: 1.2...│          - Optional: agent_id        │
│  └──────────┘                                      │
│                          ▼                          │
│              ┌─────────────────────┐              │
│              │ Wazuh Manager       │              │
│              │ (os_auth daemon)    │              │
│              │                     │              │
│              │ - Generate agent_id │              │
│              │ - Generate key      │              │
│              │ - Store in database │              │
│              └─────────────────────┘              │
│                          ▼                          │
│  ┌──────────────────────────────────┐            │
│  │ Manager Output                   │            │
│  │                                  │            │
│  │ Agent ID: 002                    │            │
│  │ Agent name: myhost01             │            │
│  │ Key: qeH45k[+XD5gLhLSzHLdU....  │            │
│  │ (shared secret)                  │            │
│  │                                  │            │
│  │ → Save in /var/ossec/etc/       │            │
│  │   client.keys          (Manager) │            │
│  └──────────────────────────────────┘            │
│                                                    │
│  STEP 2: Configure Agent                          │
│                                                    │
│  On agent system:                                 │
│  /var/ossec/etc/ossec.conf                        │
│                                                    │
│  <client>                                         │
│    <server-ip>manager.example.com</server-ip>    │
│    <config-profile>generic, linux</config-profile│
│    <enrollment>                                   │
│      <agent_name>myhost01</agent_name>           │
│      <manager_address>1.2.3.4</manager_address>  │
│      <authorization_pass_phrase>SecretPass123    │
│    </enrollment>                                  │
│  </client>                                        │
│                                                    │
│  → OR copy key manually/programmatically         │
│                                                    │
│  STEP 3: Start Agent                             │
│  $ /var/ossec/bin/wazuh-control start            │
│                                                    │
│  Agent generates:                                │
│  - Agent key file: /var/ossec/etc/client.keys   │
│  - Startup log: /var/ossec/logs/ossec.log       │
│                                                    │
│  ┌──────────────────────────────────┐            │
│  │ client.keys (on agent)           │            │
│  │                                  │            │
│  │ 002 myhost01 1.2.3.4             │            │
│  │ qeH45k[+XD5gLh...GCwKELj3mxNwF   │            │
│  │ (same as manager's copy)         │            │
│  └──────────────────────────────────┘            │
│                          ▼                         │
│      STEP 4: Initialize Communication             │
│                                                    │
│  ┌──────────┐        Encrypted TLS               │
│  │  Agent   │◄────────────────────────►┌────────┐│
│  │  Client  │ Port 1514 (TCP/UDP)     │Manager ││
│  │  (run)   │ Shared secret validation│(remote)││
│  │          │ Key exchange            │daemon  ││
│  └──────────┘                         └────────┘│
│       │                                    ▲     │
│       │                                    │     │
│       ├─► Agent ID: 002                   │     │
│       ├─► Timestamp                       │     │
│       ├─► Encrypted payload               │     │
│       │   (using shared secret)           │     │
│       │                                    │     │
│       └──────────────────────────────────►│     │
│                                           │     │
│   Manager validates:                       │     │
│   ✓ Agent ID known                         │     │
│   ✓ Shared secret matches                 │     │
│   ✓ Decrypt successful                    │     │
│   ✓ Agent status active                   │     │
│                                           │     │
│   Manager responds:                        │     │
│   ✓ ACK (acknowledgment)                  │     │
│   └─────────────────────────────────────►│     │
│                                                    │
│  STEP 5: Start Data Collection                     │
│                                                    │
│  Agent now:                                       │
│  ✓ Collects logs (logcollector)                  │
│  ✓ Monitors files (syscheckd)                    │
│  ✓ Scans for rootkits (rootcheck)               │
│  ✓ Gathers system info (syscollector)            │
│  ✓ Sends data to manager periodically            │
│                                                    │
│  Data flow:                                       │
│  Events → Buffer → Encrypt → Compress            │
│  → Send via port 1514 → Manager processes        │
│                                                    │
└──────────────────────────────────────────────────────┘
```

---

These diagrams give you a visual understanding of how Wazuh's different components interact and process data through the entire security monitoring pipeline. For additional details, refer to the main project documentation.
