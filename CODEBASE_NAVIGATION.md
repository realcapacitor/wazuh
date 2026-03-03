# Wazuh Codebase Navigation Guide

## Quick Module Reference

### If you're looking to understand/modify...

**Agent-Manager Communication**
- Files: `/src/remoted/`, `/src/os_auth/`
- Language: C/C++
- Purpose: Handles encrypted communication between agents and manager

**Log Collection**
- Files: `/src/logcollector/`
- Language: C/C++
- Purpose: Collects logs from systems on agents
- Config: `/etc/ossec-agent.conf`

**File Integrity Monitoring (FIM)**
- Files: `/src/syscheckd/`, `/architecture/FIM/`
- Language: C/C++ (legacy) / Modern (v2)
- Purpose: Monitors file changes, permissions, attributes
- Config: `<syscheck>` section in ossec.conf

**Malware/Rootkit Detection**
- Files: `/src/rootcheck/`
- Language: C/C++
- Purpose: Detects hidden files, processes, rootkits
- Config: `<rootcheck>` section in ossec.conf

**Detection Rules & Decoders**
- Files: `/src/engine/`
- Language: C/C++
- Purpose: Rule matching engine
- Config: `/var/ossec/ruleset/`

**Alert Storage & Query**
- Files: `/src/wazuh_db/`, `/framework/wazuh/` (database module)
- Language: C/C++ / Python
- Purpose: Store and retrieve security alerts

**API Endpoints**
- Files: `/api/api/controllers/`
- Language: Python
- Purpose: RESTful API implementation
- Spec: `/api/api/spec/spec.yaml`

**Agent Management**
- Files: `/framework/wazuh/agent.py`, `/src/addagent/`
- Language: Python / C
- Purpose: Register agents, manage agent enrollment
- Tools: `/var/ossec/bin/manage_agents`

**Active Response**
- Files: `/src/os_execd/`, `/framework/wazuh/active_response.py`
- Language: C / Python
- Purpose: Execute response actions on triggered rules

**Cloud Integrations**
- Files: `/wodles/aws/`, `/wodles/azure/`, `/architecture/wm_*/`
- Language: Python
- Purpose: Collect data from cloud providers

**System Configuration Assessment**
- Files: `/architecture/sca/`, `/ruleset/sca/`
- Language: Python / Rule files
- Purpose: Check compliance and misconfigurations

**Vulnerability Detection**
- Files: `/architecture/vulnerability-scanner/`
- Language: Python/C
- Purpose: Correlate software inventory with CVE databases

**Manager Dashboard/WUI**
- Related: `/api/` (provides backend)
- Note: Frontend is in separate wazuh-dashboard-plugins repo

---

## Entry Points by Role

### 👤 Security Analyst
- **Main UI**: Wazuh Dashboard
- **Alerts**: Elasticsearch indices
- **Configuration**: `/var/ossec/etc/ossec.conf`

### 👨‍💻 Developer Building Custom Rules
- **Rule Files**: `/var/ossec/ruleset/rules/`
- **Decoders**: `/var/ossec/ruleset/decoders/`
- **References**: `/src/engine/` (rule engine source)

### 🔧 System Administrator
- **Installation**: [install.sh](install.sh)
- **Agent Config**: `/etc/ossec-agent.conf`
- **Manager Config**: `/etc/ossec-manager.conf`
- **Control Script**: `/var/ossec/bin/wazuh-control`

### 🏗️ Core Developer
- **Agent Code**: `/src/client-agent/`
- **Manager Code**: `/src/remoted/`, `/src/monitord/`
- **API Code**: `/api/api/`
- **Framework**: `/framework/wazuh/`

### ☁️ Cloud Engineer
- **AWS Integration**: `/wodles/aws/`
- **Azure Integration**: `/wodles/azure/`
- **GCP Integration**: `/wodles/gcloud/`
- **Generic Data Provider**: `/architecture/data_provider/`

---

## Daemon Processes (Core Services)

| Daemon | Path | Purpose | Linux User |
|--------|------|---------|------------|
| **wazuh-agentd** | `/src/client-agent/` | Agent main process | root/wazuh |
| **wazuh-agentmodule** | `/src/wazuh_modules/` | Agent modules (AWS, Azure, etc) | wazuh |
| **wazuh-remoted** | `/src/remoted/` | Remote communication (manager) | root |
| **wazuh-monitord** | `/src/monitord/` | Monitoring & status | root |
| **wazuh-execd** | `/src/os_execd/` | Active response executor | root |
| **wazuh-analysisd** | `/src/engine/` | Analysis & rule matching | wazuh |
| **wazuh-logcollector** | `/src/logcollector/` | Log collection (agents) | root |
| **wazuh-syscheckd** | `/src/syscheckd/` | File integrity monitor | root |
| **wazuh-rootcheck** | `/src/rootcheck/` | Rootkit detector | root |

---

## Build & Compilation Targets

```bash
# From /src directory:
make TARGET=server              # Build manager
make TARGET=agent               # Build agent
make TARGET=winagent            # Build Windows agent
make deps                        # Download external dependencies
make clean                       # Clean build artifacts
```

See [INSTALL](INSTALL) for dependencies required.

---

## Testing

**Test Locations**:
- `/src/unit_tests/` - Unit tests for core components
- `/api/test/` - API tests
- `/framework/wazuh/tests/` - Framework tests
- `/tests/integration/` - Integration tests

**Test File Format**: pytest, unittest

**Running Tests**:
```bash
# Framework tests
cd /framework && python -m pytest

# API tests
cd /api && python -m pytest

# Unit tests
cd /src && make test
```

---

## Configuration Hierarchy

```
1. Compiled defaults in source code
   ↓
2. /var/ossec/etc/ossec.conf (manager) or agent.conf (agent)
   ↓
3. /var/ossec/etc/local_internal_options.conf
   ↓
4. Environment variables (some configs)
```

---

## Key Files Explained

| File | Purpose |
|------|---------|
| [install.sh](install.sh) | Main installation script - START HERE |
| [README.md](README.md) | Project overview |
| [INSTALL](INSTALL) | Detailed installation instructions |
| [VERSION.json](VERSION.json) | Current version info |
| [CHANGELOG.md](CHANGELOG.md) | Release notes & history |
| `/src/CMakeLists.txt` | CMake build configuration |
| `/src/Makefile` | Main make rules |
| `/etc/ossec-manager.conf` | Manager config template |
| `/etc/ossec-agent.conf` | Agent config template |

---

## Common Commands

```bash
# Control services
/var/ossec/bin/wazuh-control start
/var/ossec/bin/wazuh-control stop
/var/ossec/bin/wazuh-control status

# Manage agents
/var/ossec/bin/manage_agents -l                    # List agents
/var/ossec/bin/manage_agents -a                    # Add agent
/var/ossec/bin/manage_agents -r <agent_id>       # Remove agent

# Query alerts
/var/ossec/bin/wazuh-logtest -r <rule_id>        # Test rule
/var/ossec/bin/wazuh-logtest -j                   # JSON output

# Check configuration
/var/ossec/bin/wazuh-control info                 # Manager info
/var/ossec/bin/wazuh-control verify-agent-conf    # Verify agent config
```

---

## Architecture Evolution

**Legacy (v4.x)**: `/src/` - Monolithic C/C++ codebase
- Mature, stable
- Hard to extend
- Tightly coupled

**Modern (v5.0.0-alpha)**: `/architecture/` - Modular architecture
- Plugin-based components
- Better separation of concerns
- Cloud-native design
- Gradual migration from legacy code

The codebase shows both old and new approaches - legacy components in `/src/` are being gradually replaced/modernized with designs in `/architecture/`

---

## Debugging Tips

**Log Files**:
- Manager: `/var/ossec/logs/`
- Agent: `/var/ossec/logs/` (on agent)
- Startup logs: `ossec.log` or `start.log`

**Enable Debug Mode**:
```bash
./install.sh debug              # Build with debug symbols
```

or edit `/var/ossec/etc/local_internal_options.conf`:
```
logcollector.debug=2
syscheck.debug=2
remoted.debug=2
```

**Compile Flag Reference**: `/src/build.py`, `/src/Makefile`

---

## Key Concepts

**Agent Registration**
- Each agent gets unique key from manager
- Key = agent_id + shared_secret
- Used for encryption/authentication

**Ruleset**
- Decoders: Parse/extract data from logs
- Rules: Match patterns in parsed data
- Output: Generate alerts

**Active Response**
- Triggered by rule with action tag
- Scripts in `/var/ossec/active-response/bin/`
- Can block IPs, kill processes, etc.

**Alert** 
- Generated when rule matches
- Stored in alert database
- Sent to Elasticsearch
- Contains: timestamp, agent, rule_id, log_data, etc.

---

## Resource Files

- **Rules**: `/var/ossec/ruleset/rules/`
- **Decoders**: `/var/ossec/ruleset/decoders/`
- **SCA Policies**: `/var/ossec/ruleset/sca/`
- **Active Response Scripts**: `/var/ossec/active-response/bin/`
- **Configurations**: `/var/ossec/etc/`
- **Logs**: `/var/ossec/logs/`
- **Alerts Database**: `/var/ossec/queue/alerts/`

---

## Contributing Back to Project

Check [CONTRIBUTORS](CONTRIBUTORS) file for guidelines.

Typical workflow:
1. Fork repository
2. Create branch for feature/fix
3. Add tests
4. Submit pull request
5. Address review comments

---

This guide should help you navigate efficiently. For specific components, check the individual README files in each directory.
