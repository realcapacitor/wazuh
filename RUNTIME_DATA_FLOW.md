# Wazuh Runtime Data Flow - After Installation

This document explains **exactly what happens** when Wazuh is running, after successful installation.

---

## 🎯 Runtime Architecture Overview

Once Wazuh is installed and running, here's the complete data flow:

```
┌────────────────────────────────────────────────────────────────────────────┐
│                         AGENT ENDPOINT (Any Linux/Windows PC)              │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                            │
│  System Data Sources:                                                      │
│  ┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐          │
│  │  Log Files       │ │ File System      │ │ Running Processes│          │
│  │ ────────────     │ │ ────────────     │ │ ─────────────── │          │
│  │ /var/log/auth    │ │ /etc/            │ │ ps aux          │          │
│  │ /var/log/syslog  │ │ /usr/bin/        │ │ Process list    │          │
│  │ /var/log/apache  │ │ /var/www/        │ │ Memory usage    │          │
│  │ Application logs │ │ Custom dirs      │ │ CPU usage       │          │
│  │ Windows Events   │ │ File attrs       │ │ Network conns   │          │
│  └──────────────────┘ └──────────────────┘ └──────────────────┘          │
│         │                    │                      │                     │
│         └────────────┬───────┴──────────────────────┘                     │
│                      │                                                    │
│         ┌────────────▼────────────┐                                       │
│         │    WAZUH AGENT          │                                       │
│         │  (wazuh-agentd daemon)  │                                       │
│         │                         │                                       │
│         │ Spawns 4 modules:       │                                       │
│         ├─────────────────────────┤                                       │
│         │ 1. LOGCOLLECTOR         │                                       │
│         │    ────────────────     │                                       │
│         │    Reads log files      │                                       │
│         │    Follows new lines    │                                       │
│         │    Handles rotation     │                                       │
│         │    Reads continuously   │                                       │
│         │    Output: Log events   │                                       │
│         │                         │                                       │
│         │ 2. SYSCHECKD (FIM)      │                                       │
│         │    ─────────────────    │                                       │
│         │    At interval (1hr):   │                                       │
│         │    - Scan directories   │                                       │
│         │    - Hash each file     │                                       │
│         │    - Compare to baseline│                                       │
│         │    - Detect changes     │                                       │
│         │    - Track who changed  │                                       │
│         │    Output: FIM alerts   │                                       │
│         │                         │                                       │
│         │ 3. ROOTCHECK            │                                       │
│         │    ────────────         │                                       │
│         │    Periodic checks:     │                                       │
│         │    - Hidden files       │                                       │
│         │    - Hidden processes   │                                       │
│         │    - Rootkit signatures │                                       │
│         │    - Unregistered ports │                                       │
│         │    - Cloaked processes  │                                       │
│         │    Output: Security     │                                       │
│         │           alerts        │                                       │
│         │                         │                                       │
│         │ 4. SYSCOLLECTOR         │                                       │
│         │    ──────────────       │                                       │
│         │    System inventory:    │                                       │
│         │    - OS info            │                                       │
│         │    - Installed packages │                                       │
│         │    - Network interfaces │                                       │
│         │    - Processes running  │                                       │
│         │    - Hardware config    │                                       │
│         │    Output: Inventory    │                                       │
│         │           data          │                                       │
│         │                         │                                       │
│         └────────────┬────────────┘                                       │
│                      │                                                    │
│    ┌─────────────────┼─────────────────┐                                 │
│    │                 │                 │                                 │
│    ▼                 ▼                 ▼                                 │
│ Logs            FIM Events      Rootkit       Inventory                 │
│ ────             ──────────      Alerts        ─────────                │
│ Raw text        File changes    Anomalies     System info               │
│ Per line        Added/deleted    Trojans      Asset data                │
│ Events          Modified attrs   Backdoors                              │
│                 Permissions      Hidden procs                           │
│                 Ownership        Rootkits                               │
│                                                                          │
│    └─────────────────┬──────────────────────┘                           │
│                      │                                                    │
│         ┌────────────▼──────────────┐                                    │
│         │  DATA AGGREGATION         │                                    │
│         │  (In memory buffer)       │                                    │
│         │                           │                                    │
│         │ Collect all events:       │                                    │
│         │ - Group by timestamp      │                                    │
│         │ - Add agent metadata      │                                    │
│         │ - Add source info         │                                    │
│         │ - Create JSON objects     │                                    │
│         │ - Buffer up to ~500KB     │                                    │
│         │                           │                                    │
│         │ Configuration:            │                                    │
│         │ Agent ID: 001             │                                    │
│         │ Agent name: linux-01      │                                    │
│         │ Manager: 192.168.1.100    │                                    │
│         │                           │                                    │
│         └────────────┬──────────────┘                                    │
│                      │                                                    │
│         ┌────────────▼──────────────┐                                    │
│         │  ENCRYPTION LAYER         │                                    │
│         │                           │                                    │
│         │ Read shared secret:       │                                    │
│         │ from /var/ossec/etc/      │                                    │
│         │     client.keys           │                                    │
│         │                           │                                    │
│         │ Key format:               │                                    │
│         │ 001 linux-01 192.168.1.50 │                                    │
│         │ <256-bit encryption key>  │                                    │
│         │                           │                                    │
│         │ Encryption:               │                                    │
│         │ Plaintext buffer          │                                    │
│         │   ↓ (AES/DES with key)   │                                    │
│         │ Encrypted payload         │                                    │
│         │                           │                                    │
│         └────────────┬──────────────┘                                    │
│                      │                                                    │
│         ┌────────────▼──────────────┐                                    │
│         │  COMPRESSION LAYER        │                                    │
│         │                           │                                    │
│         │ Apply zlib compression    │                                    │
│         │ Encrypted data            │                                    │
│         │   ↓ (compress)            │                                    │
│         │ Compressed payload        │                                    │
│         │ Typically 70-80% smaller  │                                    │
│         │                           │                                    │
│         └────────────┬──────────────┘                                    │
│                      │                                                    │
│         ┌────────────▼──────────────┐                                    │
│         │  NETWORK TRANSMISSION     │                                    │
│         │                           │                                    │
│         │ Destination:              │                                    │
│         │ Manager IP: 192.168.1.100 │                                    │
│         │ Manager Port: 1514        │                                    │
│         │ Protocol: TCP (default)   │                                    │
│         │                           │                                    │
│         │ If manager unreachable:   │                                    │
│         │ - Buffer locally          │                                    │
│         │ - Retry periodically      │                                    │
│         │ - Store to disk queue     │                                    │
│         │ - Resume when online      │                                    │
│         │                           │                                    │
│         └────────────┬──────────────┘                                    │
│                      │                                                    │
│         ┌────────────▼──────────────┐                                    │
│         │  LOCAL BACKUP STORAGE     │                                    │
│         │  /var/ossec/queue/        │                                    │
│         │                           │                                    │
│         │ Ensures no data loss      │                                    │
│         │ Persists until sent       │                                    │
│         │ Survives agent restart    │                                    │
│         │                           │                                    │
│         └────────────┬──────────────┘                                    │
│                      │                                                    │
└──────────────────────┼────────────────────────────────────────────────────┘
                       │
       ════════════════════════════════════════════ (NETWORK) ════════════════
       Encrypted, Compressed Data Stream on Port 1514
       ════════════════════════════════════════════════════════════════════════
                       │
┌──────────────────────▼────────────────────────────────────────────────────┐
│                    WAZUH MANAGER (Central Server)                         │
├──────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  ┌────────────────────────────────────────┐                              │
│  │  REMOTED DAEMON (wazuh-remoted)        │                              │
│  │  ─────────────────────────────────────  │                              │
│  │  Listens on Port 1514 (TCP/UDP)        │                              │
│  │                                         │                              │
│  │  For each received packet:              │                              │
│  │  1. Check source IP (agent list)       │                              │
│  │  2. Extract agent ID from payload      │                              │
│  │  3. Look up key in client.keys         │                              │
│  │  4. Decrypt payload with key           │                              │
│  │  5. Decompress data                    │                              │
│  │  6. Parse decrypted events             │                              │
│  │  7. Add server metadata (timestamp,    │                              │
│  │     server hostname, received time)    │                              │
│  │  8. Queue for analysis                 │                              │
│  │                                         │                              │
│  │  Handles:                              │                              │
│  │  - Multiple agent connections          │                              │
│  │  - Disconnection detection             │                              │
│  │  - Agent registration validation       │                              │
│  │  - Load balancing                      │                              │
│  │  - Queue management                    │                              │
│  │                                         │                              │
│  └────────────────────┬────────────────────┘                              │
│                       │                                                    │
│       ┌───────────────▼───────────────┐                                    │
│       │  IN-MEMORY EVENT QUEUE        │                                    │
│       │                               │                                    │
│       │ Purpose:                      │                                    │
│       │ - Buffer incoming events      │                                    │
│       │ - Decouple reception & analysis
│       │ - Handle traffic spikes       │                                    │
│       │                               │                                    │
│       │ Events waiting in queue:      │                                    │
│       │ - SSH failed login            │                                    │
│       │ - File permission change      │                                    │
│       │ - Process start/stop          │                                    │
│       │ - Network connection          │                                    │
│       │ - Package installed           │                                    │
│       │ - System update               │                                    │
│       │                               │                                    │
│       │ Monitoring:                   │                                    │
│       │ - Queue size checked          │                                    │
│       │ - If too large: drop events   │                                    │
│       │ - If OK: send to analysis     │                                    │
│       │                               │                                    │
│       └───────────────┬───────────────┘                                    │
│                       │                                                    │
│  ┌────────────────────▼─────────────────────┐                             │
│  │  ANALYSIS ENGINE (wazuh-analysisd)       │                             │
│  │  ────────────────────────────────────────  │                             │
│  │  Main threat detection daemon            │                             │
│  │                                          │                             │
│  │  For each event in queue:                │                             │
│  │                                          │                             │
│  │  PHASE 1: DECODING                       │                             │
│  │  ───────────────────                     │                             │
│  │  Match decoders to parse log format      │                             │
│  │                                          │                             │
│  │  Example input log:                      │                             │
│  │  "Jan 15 10:30:42 ssh[1234]: Failed     │                             │
│  │   password for invalid user admin from   │                             │
│  │   192.168.1.100 port 54321"             │                             │
│  │                                          │                             │
│  │  Decoders extract:                       │                             │
│  │  {                                       │                             │
│  │    "timestamp": "Jan 15 10:30:42",       │                             │
│  │    "source": "ssh",                      │                             │
│  │    "log_type": "authentication",         │                             │
│  │    "srcip": "192.168.1.100",             │                             │
│  │    "srcport": "54321",                   │                             │
│  │    "user": "admin",                      │                             │
│  │    "event_type": "login",                │                             │
│  │    "status": "failed"                    │                             │
│  │  }                                       │                             │
│  │                                          │                             │
│  │  PHASE 2: RULE MATCHING                  │                             │
│  │  ─────────────────────                  │                             │
│  │  Check decoded fields against rules      │                             │
│  │                                          │                             │
│  │  Example rule:                           │                             │
│  │  <rule id="5710" level="5">              │                             │
│  │    <if_sid>5700</if_sid>                 │                             │
│  │    <regex>^Failed password for          │                             │
│  │            invalid user</regex>         │                             │
│  │    <description>SSH brute force          │                             │
│  │                 attack</description>    │                             │
│  │  </rule>                                 │                             │
│  │                                          │                             │
│  │  Rule matching: Does log match          │                             │
│  │  this regex? → YES                       │                             │
│  │                                          │                             │
│  │  PHASE 3: FREQUENCY & THRESHOLD CHECKS   │                             │
│  │  ─────────────────────────────────────  │                             │
│  │  Some rules require multiple matches:    │                             │
│  │                                          │                             │
│  │  Rule example:                          │                             │
│  │  <rule id="5711" level="10">             │                             │
│  │    <if_sid>5710</if_sid>                │                             │
│  │    <frequency>5</frequency>              │                             │
│  │    <timeframe>600</timeframe>            │                             │
│  │    <description>SSH brute force         │                             │
│  │                  (5 fails in 10min)     │                             │
│  │    <action>block</action>                │                             │
│  │  </rule>                                 │                             │
│  │                                          │                             │
│  │  Check: Have we seen 5 failures in      │                             │
│  │  last 10 minutes? → If YES, MATCH      │                             │
│  │                                          │                             │
│  │  PHASE 4: POST-CONDITION FILTERING       │                             │
│  │  ──────────────────────────────────    │                             │
│  │  White-list checks:                      │                             │
│  │  - Ignore rule?                          │                             │
│  │  - Override level?                       │                             │
│  │  - Add tags?                             │                             │
│  │  - Correlate with other events?          │                             │
│  │                                          │                             │
│  │  RESULT: If rule matched:                │                             │
│  │  Generate ALERT with all decoded data   │                             │
│  │                                          │                             │
│  │  ┌─────────────────────────────────┐   │                             │
│  │  │ ALERT GENERATED:                │   │                             │
│  │  │ {                               │   │                             │
│  │  │   "id": "1705318245.12345",    │   │                             │
│  │  │   "agent_id": "001",            │   │                             │
│  │  │   "agent_name": "linux-01",     │   │                             │
│  │  │   "timestamp": "2024-01-15      │   │                             │
│  │  │              T10:30:45Z",       │   │                             │
│  │  │   "rule_id": "5710",            │   │                             │
│  │  │   "rule_level": "5",            │   │                             │
│  │  │   "rule_description": "SSH      │   │                             │
│  │  │                 brute force",   │   │                             │
│  │  │   "srcip": "192.168.1.100",     │   │                             │
│  │  │   "user": "admin",              │   │                             │
│  │  │   "full_log": "Jan 15 10:30:42  │   │                             │
│  │  │         ssh[1234]: ...",         │   │                             │
│  │  │   "groups": ["sinlog",          │   │                             │
│  │  │              "auth",            │   │                             │
│  │  │              "sshd"],           │   │                             │
│  │  │   "mitre": {                    │   │                             │
│  │  │     "tactic": "Credential Access"
│  │  │     "technique": "Brute Force"  │   │                             │
│  │  │   }                             │   │                             │
│  │  │ }                               │   │                             │
│  │  └─────────────────────────────────┘   │                             │
│  │                                          │                             │
│  └────────────────────┬─────────────────────┘                             │
│                       │                                                    │
│       ┌───────────────┼────────────────┬──────────────────┐               │
│       │               │                │                  │               │
│       ▼               ▼                ▼                  ▼               │
│                                                                            │
│   Alert Output Paths:                                                    │
│                                                                            │
│   1. Local Storage                2. Elasticsearch       3. Active       │
│      ─────────────                  ───────────────       Response       │
│      /var/ossec/logs/            Indexing                ──────────     │
│      alerts.json                 Full-text search        execd daemon   │
│      JSON format                 Analytics               Triggered if   │
│      Queryable via local API      6-month retention      rule has       │
│      Real-time viewing           Remote backup           action: yes    │
│      Local backup                Grafana dashboards                     │
│                                                                            │
│   4. REST API                      5. Database           6. Email        │
│      ─────────                       ──────────          ─────          │
│      /api/alerts                   /var/ossec/queue/   Sent to admin   │
│      JSON responses                db/alerts.db         upon high       │
│      Real-time queries             SQLite               level alerts    │
│      Integration with              Quick lookups                        │
│      external systems              Agent inventory                      │
│                                                                            │
│   7. File Integrity Alerts        8. Rootkit Alerts                      │
│      ───────────────────          ──────────────────                    │
│      Combined in syscheck DB      Stored separately                     │
│      Per-agent statistics         Higher priority                       │
│      Visual reports               Immediate notification                │
│      Diff view available          Forensic details                      │
│                                                                            │
└──────────────────────────────────────────────────────────────────────────┘
                       │
         ┌─────────────┴────────────────┬──────────────────┐
         │                              │                  │
         ▼                              ▼                  ▼
    ┌─────────────┐          ┌─────────────────┐   ┌──────────────┐
    │ Local Files │          │ Elasticsearch   │   │ Active       │
    │ & Database  │          │ Cluster         │   │ Response     │
    │             │          │                 │   │              │
    │Queryable    │          │ Searchable      │   │ Blocks IPs   │
    │ immediately │          │ Full-featured   │   │ Kills procs  │
    │ Real-time   │          │ Dashboards      │   │ Executes cmd │
    │             │          │ Reports         │   │ Revokes keys │
    └────────┬────┘          └────────┬────────┘   └──────┬───────┘
             │                        │                    │
             └────────────┬───────────┴────────────────────┘
                          │
                  ┌───────▼──────────┐
                  │ Wazuh Dashboard  │
                  │ Visual Analytics │
                  │ Reports          │
                  │ Management UI    │
                  └──────────────────┘
```

---

## 📊 Detailed Event Lifecycle

Here's what happens to a **single log event** from start to finish:

### 1. **Event Generation on Agent** (0 ms)
```
Log file: /var/log/auth.log
New line written:
"Jan 15 10:30:42 myhost sshd[1234]: Failed password for invalid user 
 admin from 192.168.1.100 port 54321 ssh2"

Logcollector immediately detects new line
```

### 2. **Event Collection** (0-1 ms)
```
Logcollector module reads the line
Extracts:
- Source file: /var/log/auth.log
- Log content: Full text
- Timestamp: Jan 15 10:30:42
- Adds context:
  - Agent ID: 001
  - Agent Name: linux-01
  - Type: ssh
  - Category: authentication
```

### 3. **Buffering on Agent** (1-10 seconds)
```
Events collected in memory buffer:
- SSH failed login (this event)
- Apache access log entry (another line)
- File change notification (FIM)
- Process spawn event (syscollector)
- System update (package management)

Buffer aggregation:
- Group by 2-10 second interval
- Prepare as JSON objects
- Current buffer size: ~300KB
```

### 4. **Encryption on Agent** (10-11 seconds)
```
Read shared secret from: /var/ossec/etc/client.keys
Key for agent 001: qeH45k[+XD5gLhLSzHLdUKGFn0...

Encrypt buffer:
- Algorithm: AES encryption
- Key: from client.keys
- Mode: CBC with HMAC
- Plaintext: 300KB of events
- Ciphertext: 300KB encrypted data
```

### 5. **Compression** (11-12 seconds)
```
Apply zlib compression:
Encrypted buffer: 300KB
Compressed: ~85KB (28% of original)
Ready to send
```

### 6. **Network Transmission** (12-13 seconds)
```
Send to manager:
- Destination: 192.168.1.100:1514
- Protocol: TCP
- Packet size: ~85KB
- One packet transmission

If manager offline:
- Store to disk: /var/ossec/queue/
- Retry in 2-5 seconds
- No data lost
- Resume when online
```

### 7. **Reception on Manager** (13 seconds)
```
remoted daemon receives packet from 192.168.1.100:port12345
- Source IP: 192.168.1.100 (matches agent network)
- Decrypt using agent 001's key from client.keys
- Decompress (85KB → 300KB)
- Parse JSON events
- Queue for analysis
```

### 8. **Queue Wait** (13-14 seconds)
```
Event joins analysis queue:
Queue state:
- Current size: 150 events queued
- Processing rate: 100 events/sec
- Wait time: ~1.5 seconds
- Event priority: Normal (level < 7)
```

### 9. **Analysis Phase 1: Decoding** (14-14.1 seconds)
```
Get event from queue:
Raw event:
{
  "agent_id": "001",
  "source": "/var/log/auth.log",
  "log": "Failed password for invalid user admin from 192.168.1.100 
          port 54321"
}

Apply SSH decoder:
{
  "timestamp": "Jan 15 10:30:42",
  "service": "sshd",
  "event_type": "authentication_failure",
  "srcip": "192.168.1.100",
  "srcport": "54321",
  "user": "admin",
  "action": "failed_login",
  "protocol": "ssh2"
}
```

### 10. **Analysis Phase 2: Rule Matching** (14.1-14.2 seconds)
```
Decoders output: 
{srcip: 192.168.1.100, user: admin, action: failed_login}

Load ruleset into memory (done at startup):
- 4500+ rules loaded
- Organized by ID
- Pre-compiled patterns

Check rules in order:
Rule 5700: "SSH login attempt"
  - Match condition: NOT application ("sshd")
  - Result: SKIP (we have app "sshd")

Rule 5710: "SSH failed login"
  - Match condition: Log contains "Failed password"
  - Result: MATCH! ✓

Rule 5711: "SSH brute force attempt"
  - Match condition: 5 failures in 10 minutes
  - Previous matches in 10min window: 2
  - Result: NO MATCH YET (need 5 total)

Rule 31190: "SSH authentication success"
  - Match condition: Log contains "Accepted"
  - Result: SKIP (no match)
```

### 11. **Generate Alert** (14.2-14.3 seconds)
```
Rule 5710 matched!

Create Alert JSON:
{
  "timestamp": "2024-01-15T10:30:42+0000",
  "agent": {
    "id": "001",
    "name": "linux-01",
    "ip": "192.168.1.50"
  },
  "manager": {
    "name": "wazuh-manager"
  },
  "id": "1705318242.12345",
  "rule": {
    "level": 5,
    "description": "SSH failed login",
    "id": "5710",
    "groups": ["sinlog", "auth", "sshd"],
    "pci_dss": ["11.4"],
    "cis": ["6.2.1"]
  },
  "full_log": "Jan 15 10:30:42 sshd[1234]: Failed password for 
              invalid user admin from 192.168.1.100 port 54321 ssh2",
  "srcip": "192.168.1.100",
  "srcport": "54321",
  "user": "admin",
  "dstip": "192.168.1.50",
  "action": "alert",
  "mitre": {
    "tactic": "Credential Access",
    "technique": "Brute Force",
    "tactic_id": "TA0006",
    "technique_id": "T1110"
  }
}
```

### 12. **Alert Storage** (14.3-14.4 seconds)
```
Store locally:
- Write to: /var/ossec/logs/alerts/alerts.json
- Format: Newline-delimited JSON
- File size growing

Also store in local DB:
- Query: INSERT INTO alerts (...)
- DB: /var/ossec/queue/db/alerts.db
- Indexed for fast searches
```

### 13. **Forward to Elasticsearch** (14.4-14.5 seconds)
```
If configured, send to Elasticsearch cluster:
POST https://elasticsearch:9200/wazuh-alerts-2024.01.15/_doc

Elasticsearch benefits:
- Full-text searching
- Faceting & aggregations
- Grafana dashboards
- Analytics
- Long-term retention (6 months+)
- Distributed backup
```

### 14. **Dashboard Visualization** (14.5+ seconds)
```
User views Wazuh Dashboard (in browser):

Dashboard queries:
- Elasticsearch API call
- Filter: agent_id = "001"
- Sort: timestamp desc
- Limit: 50 latest alerts

Results appear in UI:
- Alert card shows:
  * Rule: SSH failed login (level 5)
  * Source IP: 192.168.1.100
  * User: admin
  * Time: 10:30:42
  * Agent: linux-01
  * Full log text on hover

User can:
- Click to see full details
- View similar alerts
- Check agent info
- Review rule details
- See MITRE mapping
```

### 15. **Active Response Check** (14.5 seconds, parallel)
```
Did rule have an action?
Rule 5710 has: 
  <action>firewall-drop</action> (custom)
  <active-response>no</active-response>

Result: No active response triggered
(Some rules trigger AR automatically)

If triggered, execd daemon would:
- Execute: /var/ossec/active-response/bin/firewall-drop.sh
- Pass: srcip=192.168.1.100, agent_id=001
- Script would: Block IP in firewall
- Log: Response action taken
- Create: New alert for the response
```

### 16. **Complete Journey**
```
Total time from log write to dashboard: ~0.5 seconds
Total time from log write to storage: ~14.5 seconds (network latency)

Alert now:
✓ Visible in dashboard
✓ Stored locally
✓ Stored in Elasticsearch
✓ Queryable via API
✓ Available in reports
✓ Correlated with other events
✓ Analyzed for patterns
```

---

## 🔄 Multi-Agent Flow

When running many agents:

```
Agent 1 ─┐
Agent 2 ──┼──► remoted daemon (handles all agents in parallel)
Agent 3 ──┼──► Decrypts each agent's data
...      ├──► Queues events
Agent 50 ─┤
          │
          └──► Analysis Queue
               │
               ├─ Event from Agent 1
               ├─ Event from Agent 25
               ├─ Event from Agent 3
               └─ (150+ events waiting)
               
               ▼
          
          analysisd processes queue
          (Can do thousands of events/sec)
          
          Generated alerts from multiple agents
          → All stored together
          → Can correlate across agents
          → Create multi-agent rules
```

---

## 📈 Performance Metrics

When Wazuh is operational:

| Metric | Typical Value | Max Value |
|--------|---------------|-----------|
| **Agents connected** | 10-100 | 14,000 |
| **Events per second** | 100-500 | 2,000+ |
| **Analysis latency** | 100-500 ms | 5 seconds |
| **Alert generation rate** | 10-50 alerts/min | Depending on rules |
| **Memory usage** (manager) | 500MB - 2GB | 10GB+ |
| **Disk I/O** | 10-50 MB/sec | 200+ MB/sec |
| **Network bandwidth** | Varies by log volume | 100+ Mbps |
| **Elasticsearch indexing** | Real-time | 5+ minute lag if overloaded |

---

## 🎯 Summary of Runtime Flow

```
┌─────────────────────────────────────────────────────────────┐
│ AGENT CONTINUOUS OPERATIONS                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│ Every 1-2 seconds:                                           │
│ 1. Logcollector: Read new log lines                         │
│ 2. Add to memory buffer                                     │
│ 3. Continue reading other sources                          │
│                                                              │
│ Every 10-30 seconds:                                         │
│ 4. Encrypt buffer with shared secret                        │
│ 5. Compress                                                 │
│ 6. Send to manager (port 1514)                              │
│ 7. Clear buffer, start new cycle                           │
│                                                              │
│ Every 1 hour (configurable):                                 │
│ 8. Syscheckd: Scan files and create baseline                │
│ 9. Detect any changes since last scan                      │
│ 10. Send FIM alerts to manager                              │
│                                                              │
│ Every 12 hours (configurable):                               │
│ 11. Rootcheck: Scan for trojans/rootkits                   │
│ 12. Check for suspicious files/processes                   │
│ 13. Send alerts if anything found                          │
│                                                              │
│ Every 24 hours (configurable):                               │
│ 14. Syscollector: Update system inventory                   │
│ 15. Send package list, running processes, etc.             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
         │
         │ (Continuous encrypted stream)
         ▼
┌─────────────────────────────────────────────────────────────┐
│ MANAGER CONTINUOUS OPERATIONS                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│ Real-time (non-stop):                                       │
│ 1. remoted: Listen for agent packets on port 1514          │
│ 2. Receive, decrypt, decompress                            │
│ 3. Queue events for analysis                               │
│                                                              │
│ Real-time (continuous):                                     │
│ 4. analysisd: Process events from queue                     │
│ 5. Decode log patterns                                      │
│ 6. Match against rules database                            │
│ 7. Generate alerts for matches                             │
│ 8. Store alerts (local DB + Elasticsearch)                 │
│ 9. Execute active responses if needed                       │
│                                                              │
│ Every 5 seconds:                                            │
│ 10. monitord: Check all daemons are alive                  │
│ 11. If any died, restart it                                │
│ 12. Monitor disk space                                     │
│ 13. Rotate old logs if needed                              │
│                                                              │
│ Every minute:                                               │
│ 14. Database maintenance                                   │
│ 15. Prune old entries                                      │
│ 16. Update statistics                                      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│ DATA VISIBILITY                                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│ Real-time Sources:                                          │
│ - Local alerts file: /var/ossec/logs/alerts/alerts.json    │
│ - SQLite DB: /var/ossec/queue/db/alerts.db                 │
│ - REST API: /api/agents/001/events                         │
│                                                              │
│ Dashboard Visualizations:                                   │
│ - Full event list                                          │
│ - Threat level indicator                                   │
│ - Timeline graph                                           │
│ - Top rules triggered                                      │
│ - Top agents by alert count                                │
│ - Security compliance scores                               │
│ - Vulnerability correlation                                │
│                                                              │
│ Elasticsearch (if enabled):                                 │
│ - 6 month data retention                                   │
│ - Full-text search                                         │
│ - Advanced analytics                                       │
│ - Custom dashboards (Kibana/Grafana)                       │
│ - Report generation                                        │
│ - Archive & compliance reports                             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

This is the complete runtime flow of Wazuh after installation and during normal continuous operation!

