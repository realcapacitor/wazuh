# Wazuh Project - Key Files & Examples

## 📌 Critical Files to Know

### Project Metadata
```
/Users/shivam/wazuh/
├── README.md                 ← Project overview
├── INSTALL                   ← Installation instructions
├── install.sh               ← Main installer script (ENTRY POINT)
├── VERSION.json             ← Current version (5.0.0-alpha0)
├── CHANGELOG.md             ← Version history
├── LICENSE                  ← GPLv2 license
├── CONTRIBUTORS             ← Contribution guidelines
└── SECURITY.md              ← Security policy
```

### Configuration Templates
```
/etc/
├── ossec-manager.conf       ← Manager configuration template
├── ossec-agent.conf         ← Agent configuration template
├── agent.conf               ← Deprecated (use ossec-agent.conf)
├── internal_options.conf    ← Internal settings (debug, timers, etc)
└── templates/               ← Additional config templates
```

### Source Code (C/C++ Core)
```
/src/
├── CMakeLists.txt           ← CMake build configuration
├── Makefile                 ← Make targets
├── build.py                 ← Build script
├── remoted/                 ← Agent-Manager communication daemon
├── logcollector/            ← Log collection daemon
├── syscheckd/               ← File integrity monitoring daemon
├── rootcheck/               ← Rootkit detection daemon
├── os_auth/                 ← Authentication & key management
├── os_execd/                ← Active response executor
├── monitord/                ← Manager monitoring daemon
├── engine/                  ← Detection engine (rule matching)
├── wazuh_modules/           ← Plugin modules (AWS, Azure, etc)
├── wazuh_db/                ← Alert database management
├── shared/                  ← Shared utilities
└── config/                  ← Configuration parsing
```

### Python Framework
```
/framework/
├── setup.py                 ← Framework package setup
├── requirements.txt         ← Python dependencies
├── Makefile                 ← Build/test targets
└── wazuh/                   ← Main framework package
    ├── agent.py             ← Agent management API
    ├── manager.py           ← Manager API
    ├── cluster.py           ← Clustering support
    ├── security.py          ← RBAC & authentication
    ├── core/                ← Core utilities
    ├── rbac/                ← Role-based access control
    └── tests/               ← Test suite
```

### REST API
```
/api/
├── setup.py                 ← API package setup
├── api/
│   ├── authentication.py    ← API authentication
│   ├── configuration.py     ← API config
│   ├── controllers/         ← Endpoint handlers
│   ├── models/              ← Data models
│   ├── spec/spec.yaml       ← OpenAPI specification
│   └── test/                ← API tests
└── scripts/                 ← Utility scripts
```

### Integration Modules (Python)
```
/wodles/
├── aws/                     ← AWS integration
├── azure/                   ← Azure integration
├── docker-listener/         ← Docker integration
└── gcloud/                  ← Google Cloud integration
```

### Modern Architecture (Next-gen)
```
/architecture/
├── data_provider/           ← Data collection framework
├── dbsync/                  ← Database synchronization
├── FIM/                     ← File Integrity Monitoring v2
├── vulnerability-scanner/   ← Vulnerability detection
├── syscollector/            ← System inventory collector
├── router-pubsub/           ← Message routing
├── wm_azure/                ← Azure module (modern)
├── wm_gcp/                  ← GCP module (modern)
├── wm_github/               ← GitHub module (modern)
├── wm_office365/            ← Office365 module (modern)
└── wm_sca/                  ← Security Config Assessment (modern)
```

---

## 📄 Important Configuration Examples

### Manager Configuration (simplified)
```xml
<!-- /var/ossec/etc/ossec.conf on manager -->
<ossec_config>
  <global>
    <email_notification>yes</email_notification>
    <email_from>wazuh@example.com</email_from>
    <smtp_server>localhost</smtp_server>
    <email_maxperhour>12</email_maxperhour>
    <agents_disconnection_time>10m</agents_disconnection_time>
    <agents_disconnection_alert_time>0</agents_disconnection_alert_time>
  </global>

  <remote>
    <connection>syslog</connection>
    <port>514</port>
    <protocol>tcp,udp</protocol>
  </remote>

  <alerts>
    <log_alert_level>2</log_alert_level>
    <email_alert_level>7</email_alert_level>
  </alerts>

  <logging>
    <log_format>json</log_format>
  </logging>

  <rules>
    <include>rules_config.xml</include>
    <include>pam_rules.xml</include>
    <include>sshd_rules.xml</include>
    <!-- ... many more rule files -->
  </rules>

  <syscheck>
    <frequency>79200</frequency> <!-- 22 hours -->
    <directories check_all="yes" />/etc,/usr/bin,/usr/sbin</directories>
    <directories check_all="yes" />/bin,/sbin</directories>
    <ignore>/etc/mtab</ignore>
    <ignore>/etc/hosts.allow</ignore>
    <ignore>/etc/hosts.deny</ignore>
    <auto_ignore>yes</auto_ignore>
  </syscheck>

  <rootcheck>
    <disabled>no</disabled>
    <check_files>yes</check_files>
    <check_trojans>yes</check_trojans>
    <check_dev>yes</check_dev>
    <check_sys>yes</check_sys>
    <check_pids>yes</check_pids>
  </rootcheck>
</ossec_config>
```

### Agent Configuration (simplified)
```xml
<!-- /var/ossec/etc/ossec.conf on agent -->
<ossec_config>
  <client>
    <server-ip>192.168.1.100</server-ip>
    <server-hostname>wazuh-manager</server-hostname>
    <port>1514</port>
    <protocol>tcp</protocol>
    <config-profile>linux</config-profile>
  </client>

  <logging>
    <log_format>json</log_format>
  </logging>

  <syscheck>
    <frequency>3600</frequency> <!-- 1 hour -->
    <directories check_all="yes">/etc,/usr/bin,/usr/sbin</directories>
    <directories check_all="yes">/bin,/sbin</directories>
    <ignore>/etc/mtab</ignore>
    <ignore type="sregex">^/proc</ignore>
    <ignore type="sregex">^/sys</ignore>
    <monitored-files>/etc/passwd,/etc/shadow</monitored-files>
  </syscheck>

  <rootcheck>
    <disabled>no</disabled>
    <check_files>yes</check_files>
    <check_trojans>yes</check_trojans>
  </rootcheck>

  <wodle name="osquery">
    <disabled>yes</disabled>
  </wodle>
</ossec_config>
```

---

## 🔧 Build & Installation

### installation Process Flow
```
1. User runs: ./install.sh

   ↓

2. Script checks:
   - OS detection (detect.sh)
   - Compiler availability (gcc/clang)
   - Required dependencies
   - Existing installation

   ↓

3. Prompts user for:
   - Installation type (server/agent/hybrid)
   - Installation directory (default: /var/ossec)
   - Enable/disable features
   - Initial configuration options

   ↓

4. Download external dependencies:
   make deps TARGET=<server|agent>

   ↓

5. Compile source:
   cd src && make TARGET=<server|agent> build

   ↓

6. Build artifacts created:
   - Executable binaries
   - Shared libraries
   - Configuration files

   ↓

7. Installation phase:
   - Create directories
   - Copy files to /var/ossec
   - Set permissions
   - Create users/groups
   - Generate init scripts

   ↓

8. Post-installation:
   - Create key for agent registration
   - Test connectivity (if network accessible)
   - Generate initial configuration
   - Create start script

   ↓

9. Start services (optional):
   /var/ossec/bin/wazuh-control start
```

### Building from Source
```bash
# Full installation
cd /Users/shivam/wazuh
./install.sh

# Or specific build targets
cd src

# Download external deps (sqlite, OpenSSL, etc)
make deps TARGET=server

# Build manager
make TARGET=server INSTALLDIR=/var/ossec -j4 build

# Build agent  
make TARGET=agent INSTALLDIR=/var/ossec -j4 build

# Build Windows agent
make TARGET=winagent -j4 build

# With optimization
make TARGET=server INSTALLDIR=/var/ossec OPTIMIZE_CPYTHON=yes build

# With database support
make TARGET=server INSTALLDIR=/var/ossec DATABASE=pgsql build

# Clean build artifacts
make clean
```

---

## 🚀 Starting and Controlling Services

### Manager Services
```bash
# Start all manager services
/var/ossec/bin/wazuh-control start

# Start specific service
/var/ossec/bin/wazuh-control start wazuh-remoted
/var/ossec/bin/wazuh-control start wazuh-analysisd
/var/ossec/bin/wazuh-control start wazuh-execd

# Stop services
/var/ossec/bin/wazuh-control stop

# Check status
/var/ossec/bin/wazuh-control status

# Get manager info
/var/ossec/bin/wazuh-control info

# Restart services
/var/ossec/bin/wazuh-control restart
```

### Agent Services
```bash
# Start agent
/var/ossec/bin/wazuh-control start

# Status
/var/ossec/bin/wazuh-control status

# Check connectivity to manager
/var/ossec/bin/wazuh-control info
```

---

## 👥 Agent Management

### Add Agent
```bash
/var/ossec/bin/manage_agents -a

# Prompts for:
# - Agent name
# - Agent IP (or hostname)
# - Optional: Agent ID

# Output:
# Agent ID: 002
# Agent name: hostname01
# IP address: 192.168.1.50
# Key: qeH45k[+XD5gLhLSzHLdUKGFn0....
```

### List Agents
```bash
/var/ossec/bin/manage_agents -l

# Output:
# ID: 000, Name: wazuh-manager, IP: 127.0.0.1, Status: Active
# ID: 001, Name: linux-agent, IP: 192.168.1.50, Status: Active
# ID: 002, Name: windows-pc, IP: 10.0.0.5, Status: Disconnected
```

### Remove Agent
```bash
/var/ossec/bin/manage_agents -r 002

# Confirmation prompt:
# Do you want to remove it? (y/n): y
```

### Import Keys
```bash
# Paste key and agent info into /var/ossec/etc/client.keys
# Format: ID NAME IP KEY
# Example:
echo "002 hostname01 192.168.1.50 qeH45k[+XD5gLh..." >> /var/ossec/etc/client.keys
chown wazuh:wazuh /var/ossec/etc/client.keys
chmod 640 /var/ossec/etc/client.keys
```

---

## 📋 Testing & Debugging

### Rule Testing
```bash
# Test a log line against rules
echo "2024/01/15 10:30:45 Failed password for invalid user admin from 192.168.1.100 port 54321 ssh2" \
  | /var/ossec/bin/wazuh-logtest -j

# Output: JSON with matched rule ID, severity, etc.
```

### Configuration Validation
```bash
# Validate ossec.conf syntax
/var/ossec/bin/wazuh-control verify-agent-conf

# Check if syntax is correct
```

### Log Testing
```bash
# Interactive rule tester
/var/ossec/bin/wazuh-logtest

# Feed logs and see which rules match
```

### Debug Logs
```bash
# Enable debug logging in internal_options.conf
vi /var/ossec/etc/local_internal_options.conf

# Set debug levels:
logcollector.debug=2
syscheck.debug=2
remoted.debug=2
analysisd.debug=2

# Restart services
/var/ossec/bin/wazuh-control restart

# Check debug output
tail -f /var/ossec/logs/ossec.log
```

---

## 🔐 Security & Encryption

### Key Files
```
/var/ossec/etc/client.keys
- Contains agent keys
- Format: ID NAME IP KEY
- Permissions: 640 (owner: root, group: wazuh)
- Must be identical on manager and agent
```

### SSL/TLS Certificates
```bash
# Generate self-signed certificate (if not using provided certs)
openssl req -x509 -newkey rsa:2048 -keyout server.key -out server.cert -days 365

# Location in Wazuh:
/var/ossec/etc/sslmanager/ca.cert
/var/ossec/etc/sslmanager/server.pem
/var/ossec/etc/sslmanager/server.key
```

---

## 📊 Alert Storage & Querying

### Alert Database Format
```bash
# Main alert file (JSON lines)
/var/ossec/logs/alerts/alerts.json

# Example alert:
{
  "timestamp": "2024-01-15T10:30:45.123456+0000",
  "agent": {
    "id": "001",
    "name": "linux-agent",
    "ip": "192.168.1.50"
  },
  "manager": {
    "name": "wazuh-manager"
  },
  "id": "1705318245.12345",
  "version": 1,
  "rule": {
    "level": 5,
    "description": "Failed sshd login",
    "id": "5710"
  },
  "full_log": "2024/01/15 10:30:45 Failed password for invalid user admin...",
  "srcip": "192.168.1.100",
  "srcport": "54321",
  "dstip": "192.168.1.50",
  "action": "alert"
}
```

### Query Elasticsearch
```bash
# Search alerts from specific agent
curl -s "http://localhost:9200/wazuh-alerts-4.x-*/_search?pretty" \
  -H 'Content-Type: application/json' \
  -d '{
    "query": {
      "match": {
        "agent.id": "001"
      }
    }
  }'

# Search by rule level (severity)
curl -s "http://localhost:9200/wazuh-alerts-4.x-*/_search?pretty" \
  -H 'Content-Type: application/json' \
  -d '{
    "query": {
      "range": {
        "rule.level": {"gte": 7}
      }
    }
  }'
```

---

## 🔌 API Examples

### Query Agents
```bash
# List all agents
curl -X GET "https://localhost:55000/api/agents?pretty=true" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  --insecure

# Get specific agent
curl -X GET "https://localhost:55000/api/agents/001?pretty=true" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  --insecure
```

### Query Alerts
```bash
# Get alerts for specific agent
curl -X GET "https://localhost:55000/api/agents/001/events" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  --insecure

# Get alerts by rule ID
curl -X GET "https://localhost:55000/api/overview/rules?rule_ids=5710" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  --insecure
```

### Manage Agents (API)
```bash
# Add agent
curl -X POST "https://localhost:55000/api/agents" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "new-agent",
    "ip": "192.168.1.100"
  }' \
  --insecure

# Restart agent
curl -X POST "https://localhost:55000/api/agents/001/restart" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  --insecure
```

---

## 📈 Monitoring Wazuh Health

### Check Manager Status
```bash
# Manager health
ps aux | grep wazuh

# Should show these processes:
# - wazuh-remoted
# - wazuh-analysisd
# - wazuh-execd
# - wazuh-monitord

# Check disk usage
du -sh /var/ossec/

# Check logs for errors
grep -i error /var/ossec/logs/ossec.log | tail -20

# Restart if issues
/var/ossec/bin/wazuh-control restart
```

### Check Agent Status
```bash
# On agent:
systemctl status wazuh-agent
# or
/var/ossec/bin/wazuh-control status

# Check connectivity
netstat -tlnp | grep 1514  # Should show connection to manager

# View agent logs
tail -f /var/ossec/logs/agent.log
```

---

## 📦 File Structure After Installation

```
/var/ossec/
├── bin/                          # Executables
│   ├── wazuh-control
│   ├── manage_agents
│   ├── wazuh-logtest
│   ├── wazuh-agentd
│   └── ... (more daemons)
├── etc/                          # Configuration
│   ├── ossec.conf               # Main config
│   ├── client.keys              # Agent keys
│   ├── local_internal_options.conf
│   ├── sslmanager/              # SSL certs
│   └── ruleset/
│       ├── rules/               # Security rules
│       ├── decoders/            # Log decoders
│       └── sca/                 # SCA policies
├── queue/                        # Data queues
│   ├── agents/                  # Agent data
│   ├── alerts/                  # Alert files
│   ├── diff/                    # FIM diffs
│   └── agentless/               # Agentless data
├── logs/                         # Log files
│   ├── ossec.log                # Manager log
│   ├── alerts/                  # Alert storage
│   └── ...
├── var/                          # Runtime data
│   ├── db/                      # Databases
│   ├── run/                     # PID files
│   └── ...
├── lib/                          # Shared libraries
├── active-response/              # Response scripts
│   ├── bin/
│   │   ├── disable-account.sh
│   │   ├── firewall-drop.sh
│   │   ├── host-deny.sh
│   │   └── ...
│   └── scripts/
├── tmp/                          # Temporary files
└── agentless/                    # Agentless monitoring
```

---

## 🎯 Summary

This guide covers:
- **Installation**: How to build and install Wazuh
- **Configuration**: Key configuration files and examples
- **Management**: Managing agents and services
- **API**: REST API examples
- **Troubleshooting**: Debugging and monitoring
- **Architecture**: How components interact

For complete details, check the official [Wazuh Documentation](https://documentation.wazuh.com).
