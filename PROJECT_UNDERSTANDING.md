# Wazuh Project Understanding

## 📋 Project Overview

**Wazuh** is a free and open-source security monitoring platform used for **threat prevention, detection, and response**. It protects workloads across on-premises, virtualized, containerized, and cloud-based environments.

**Current Version**: 5.0.0 (alpha0)

**License**: GPLv2 (GNU General Public License)

---

## 🏗️ Architecture

### High-Level Components

```
┌─────────────────────────────────────────────────────────────┐
│                    Wazuh Ecosystem                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────┐      ┌──────────────────┐            │
│  │   Wazuh Agent   │◄────►│  Wazuh Manager   │            │
│  │  (Endpoints)    │      │   (Central)      │            │
│  └─────────────────┘      └──────────────────┘            │
│                                    │                       │
│                                    ▼                       │
│                         ┌──────────────────┐              │
│                         │   Elastic Stack  │              │
│                         │ (Search & Visual)│              │
│                         └──────────────────┘              │
│                                    │                       │
│                                    ▼                       │
│                         ┌──────────────────┐              │
│                         │  Wazuh Dashboard │              │
│                         │    (Web UI)      │              │
│                         └──────────────────┘              │
└─────────────────────────────────────────────────────────────┘
```

### Two-Tier Deployment Model

1. **Wazuh Agent** - Deployed on monitored systems (endpoints)
   - Collects security data
   - Performs file integrity monitoring
   - Detects threats at endpoint level
   - Lightweight and supports multiple platforms

2. **Wazuh Manager** - Central server
   - Collects and analyzes data from agents
   - Executes security rules
   - Stores alerts and events
   - Provides command and control capabilities

---

## 📁 Directory Structure & Purpose

### `/api` - REST API Layer
- **Purpose**: Provides RESTful API for interaction with Wazuh manager
- **Key Files**:
  - `api/specification/spec.yaml` - OpenAPI specification
  - `authentication.py` - API authentication/authorization
  - `controllers/` - API endpoint handlers
  - `models/` - Data models for API
- **Features**: Add agents, restart managers, query data, manage configuration

### `/framework` - Python Framework & Business Logic
- **Purpose**: Core Python framework for Wazuh manager functionality
- **Key Modules**:
  - `wazuh/agent.py` - Agent management
  - `wazuh/manager.py` - Manager operations
  - `wazuh/cluster.py` - Clustering support
  - `wazuh/security.py` - Security authentication/authorization
  - `wazuh/rbac/` - Role-Based Access Control
  - `wazuh/syscheck.py` - File Integrity Monitoring
  - `wazuh/rootcheck.py` - Rootkit detection
  - `wazuh/active_response.py` - Active response actions

### `/src` - C/C++ Core Engine
- **Purpose**: High-performance compiled components (agents & manager)
- **Key Modules**:

| Module | Purpose |
|--------|---------|
| `logcollector/` | Collects logs from systems |
| `syscheckd/` | File integrity monitoring daemon |
| `rootcheck/` | Rootkit detection |
| `remoted/` | Remote communication daemon (agent ↔ manager) |
| `monitord/` | Monitoring and daemon management |
| `os_auth/` | Authentication (key exchange, agent registration) |
| `os_execd/` | Command execution daemon |
| `wazuh_modules/` | Plugin modules (Azure, AWS, GCP, Office365, etc.) |
| `wazuh_db/` | Database management (alerts, inventories) |
| `shared/` | Shared utilities and libraries |
| `engine/` | Core detection engine |
| `config/` | Configuration parsing |

### `/architecture` - Modern Architecture Components
- **Purpose**: Next-generation modular architecture
- **Key Modules**:
  - `data_provider/` - Data collection framework
  - `dbsync/` - Database synchronization
  - `FIM/` - File Integrity Monitoring (v2)
  - `router-pubsub/` - Message routing
  - `syscollector/` - System inventory collection
  - `vulnerability-scanner/` - CVE detection
  - `centralized_configuration/` - Config management
  - `wm_azure/`, `wm_gcp/`, `wm_github/`, `wm_office365/` - Cloud integrations

### `/etc` - Configuration Files
- **Purpose**: Default configuration templates
- **Key Files**:
  - `ossec-manager.conf` - Manager configuration
  - `ossec-agent.conf` - Agent configuration
  - `internal_options.conf` - Internal settings

### `/wodles` - Wazuh Modules (Python)
- **Purpose**: Plugin modules written in Python
- **Modules**: AWS, Azure, Docker-listener, GCloud integrations

### `/ruleset` - Detection Rules
- **Purpose**: Security rules and compliance checks
- **Contents**:
  - MITRE ATT&CK mappings
  - SCA (Security Configuration Assessment) rules

### `/tests` - Testing Infrastructure
- **Purpose**: Integration tests and test suites

### `/docs` - Documentation
- **Purpose**: User and developer documentation

---

## 🔄 Data Flow Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    AGENT SIDE                                 │
│                                                               │
│  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ Log Files   │  │File Integrity│  │   Registry   │       │
│  │ System Info │  │  (Syscheck)  │  │  (Windows)   │       │
│  └──────┬──────┘  └──────┬───────┘  └──────┬───────┘       │
│         │                │                  │                │
│         └────────────────┼──────────────────┘                │
│                          ▼                                   │
│            ┌──────────────────────────┐                     │
│            │  Log Collector Daemon    │                     │
│            │  (logcollector)          │                     │
│            └──────────┬───────────────┘                     │
│                       ▼                                     │
│            ┌──────────────────────────┐                     │
│            │ Encryption + Compression │                     │
│            │ (Agent Buffer)           │                     │
│            └──────────┬───────────────┘                     │
│                       ▼                                     │
│            ┌──────────────────────────┐                     │
│            │  Secure Communication    │                     │
│            │   (TLS/SSL)              │                     │
│            └──────────┬───────────────┘                     │
└─────────────────────────┼──────────────────────────────────┘
                          │
         (Secure Channel) │
                          │
┌─────────────────────────▼──────────────────────────────────┐
│                    MANAGER SIDE                             │
│                                                             │
│            ┌──────────────────────────┐                   │
│            │  Remote Daemon (remoted) │                   │
│            │  (Receives Data)         │                   │
│            └──────────┬───────────────┘                   │
│                       ▼                                   │
│            ┌──────────────────────────┐                   │
│            │  Data Decryption         │                   │
│            └──────────┬───────────────┘                   │
│                       ▼                                   │
│            ┌──────────────────────────┐                   │
│            │  Alert/Event Processing  │                   │
│            │  (Decoders & Rules)      │                   │
│            └──────────┬───────────────┘                   │
│                       ▼                                   │
│            ┌──────────────────────────┐                   │
│            │  Detection Engine        │                   │
│            │  (Rule Matching)         │                   │
│            └──────────┬───────────────┘                   │
│                       ▼                                   │
│            ┌──────────────────────────┐                   │
│            │  Alert Storage           │                   │
│            │  (Wazuh-DB)              │                   │
│            └──────────┬───────────────┘                   │
│                       ▼                                   │
│            ┌──────────────────────────┐                   │
│            │  Elasticsearch/Storage   │                   │
│            └──────────────────────────┘                   │
│                       │                                   │
└───────────────────────┼───────────────────────────────────┘
                        │
        (Dashboards/Queries)
                        │
                        ▼
             ┌──────────────────────┐
             │  Wazuh Dashboard     │
             │  (Web UI / Analytics)│
             └──────────────────────┘
```

---

## 🎯 Key Features & Capabilities

### 1. **Intrusion Detection** 🔍
- Malware and rootkit detection
- Suspicious anomaly identification
- Hidden files and cloaked processes detection
- Unregistered network listener detection
- System call response inconsistencies

### 2. **Log Data Analysis** 📊
- Centralized log collection
- Log parsing and decoding
- Rule-based analysis with regex engine
- Multiple log source support (syslog, APIs, etc.)
- Detection of: errors, misconfigurations, malicious activities, policy violations

### 3. **File Integrity Monitoring (FIM)** 🔐
- Real-time file system monitoring
- Tracks: content, permissions, ownership, attributes
- User and process identification
- CVE correlation
- PCI DSS compliance support

### 4. **Vulnerability Detection** 🛡️
- Software inventory collection
- CVE database correlation
- Automated vulnerability assessment
- CVE updates from security databases

### 5. **Configuration Assessment** ⚙️
- System and application config monitoring
- Compliance checking (PCI DSS, CIS, GPG13, GDPR)
- Security policy enforcement
- Remediation recommendations

### 6. **Active Response** ⚡
- Automated threat countermeasures
- IP blocking capabilities
- Rate limiting
- Process termination
- Custom response scripts

### 7. **Cloud Security** ☁️
- AWS, Azure, Google Cloud integration
- Cloud API-level monitoring
- Cloud configuration assessment
- Instance-level monitoring

### 8. **Container Security** 🐳
- Docker integration
- Container behavior monitoring
- Image and volume security
- Network settings monitoring
- Privileged container detection

---

## 🔧 Build & Installation

### Build System
- **Language**: C/C++ (core) + Python (framework/API)
- **Build Tool**: CMake + Make
- **Build File**: `/src/CMakeLists.txt`, `/src/Makefile`

### Installation Methods
1. **Fast Installation**: `./install.sh` (recommended for most users)
2. **Binary Installation**: `./install.sh binary-install` (uses pre-compiled binaries)
3. **Source Build**: Manual compilation for custom configurations

### Default Installation Path
- Installation directory: `/var/ossec`
- Configuration: `/var/ossec/etc/`
- Binaries: `/var/ossec/bin/`
- Data/Logs: `/var/ossec/queue/`, `/var/ossec/logs/`

---

## 🔌 Integration Points

### 1. **API Layer** (`/api`)
- RESTful API for external integrations
- Authentication (API keys, OAuth)
- RBAC support
- OpenAPI specification

### 2. **Cloud Integrations** (`/wodles`, `/architecture/wm_*`)
- AWS (EC2, Lambda, S3, CloudTrail)
- Azure (VMs, SQL, App Services)
- Google Cloud
- GitHub
- Office 365

### 3. **Data Providers** (`/architecture/data_provider`)
- Structured data collection framework
- Plugin architecture for new data sources

### 4. **Message Bus** (`/architecture/router-pubsub`)
- Inter-component communication
- Pub/Sub pattern
- Event routing

---

## 📦 Module Interactions

```
┌─────────────────────────────────────────────┐
│         API / Web Interface Layer           │
│  (/api, /framework/wazuh)                   │
└────────────────┬────────────────────────────┘
                 │
        ┌────────┴────────┐
        ▼                 ▼
   ┌─────────┐      ┌──────────┐
   │Framework │      │   API    │
   │ Layer    │      │Controller│
   └────┬────┘      └────┬─────┘
        │                 │
        └────────┬────────┘
                 ▼
      ┌──────────────────┐
      │  Database Layer  │
      │  (wazuh-db)      │
      └────────┬─────────┘
               │
        ┌──────┴──────┬──────────┬─────────┐
        ▼             ▼          ▼         ▼
   ┌────────┐  ┌─────────┐ ┌────────┐ ┌───────┐
   │ remoted│  │ os_auth │ │logcoll.│ │syschkd│
   │(C/C++) │  │(C/C++)  │ │(C/C++) │ │(C/C++)│
   └────────┘  └─────────┘ └────────┘ └───────┘
        │             │         │          │
        │ ┌───────────┼─────────┘          │
        │ │           │                    │
        ▼ ▼           ▼                    ▼
    ┌──────────────────────┐      ┌──────────────┐
    │  Detection Engine    │      │Other Daemons │
    │  (Rules + Decoders)  │      │(rootcheck, ..)
    └──────┬───────────────┘      └──────────────┘
           │
           ▼
    ┌──────────────────┐
    │ Alert Storage    │
    │(Elasticsearch)   │
    └──────────────────┘
```

---

## 📊 Technology Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Core Engine** | C/C++ | High-performance data collection & processing |
| **Framework** | Python 3 | Manager logic, business rules, API implementation |
| **API** | Python + Flask/FastAPI | RESTful API endpoints |
| **Database** | SQLite/MySQL/PostgreSQL | Local alerts and metadata |
| **Search** | Elasticsearch | Alert indexing and analytics |
| **Dashboard** | React/Angular | Web UI visualization |
| **IPC** | Unix sockets, TLS | Agent-Manager communication |
| **Build** | CMake, Make | Compilation and installation |

---

## 🔐 Security Architecture

### Authentication
- Agent registration with key exchange
- API key and OAuth support
- RBAC with rule-based access control

### Encryption
- TLS/SSL for agent-manager communication
- Data encryption at rest (optional)
- Secure key management

### Data Protection
- Message authentication codes
- Compressed and encrypted payloads
- Secure agent enrollment process

---

## 📈 Deployment Models

1. **Agent-Manager**: Traditional centralized (most common)
2. **Clustered**: Multiple managers for high availability
3. **Distributed**: Multi-tier architecture for scale
4. **Cloud Native**: For containerized/Kubernetes environments

---

## 🛠️ Development Areas

### Active Development Features
- Vulnerability Scanner (new)
- Centralized Configuration
- Enhanced DBSync
- New Data Provider Framework
- Cloud security enhancements
- Container security improvements

### In This Codebase (v5.0.0-alpha0)
- Next-gen architecture in `/architecture/`
- Legacy components in `/src/`
- Python framework modernization
- API enhancements

---

## 🚀 Typical Workflow

```
1. Agent Installation
   └─► Configure agent.conf
       └─► Register with manager
           └─► Receive encryption keys

2. Data Collection (Agent)
   └─► Collect logs, file changes, system info
       └─► Buffer data locally
           └─► Encrypt and send to manager

3. Data Processing (Manager)
   └─► Decrypt and decompress
       └─► Parse with decoders
           └─► Apply security rules

4. Alert Generation
   └─► Rule match triggers alert
       └─► Store in alerts database
           └─► Send to Elasticsearch

5. Visualization
   └─► Dashboard queries alerts
       └─► Display to security team
           └─► Trigger active responses if needed
```

---

## 📚 Key Documentation Files

- [README.md](README.md) - Project overview
- [INSTALL](INSTALL) - Installation guide
- [install.sh](install.sh) - Automated installer
- [CHANGELOG.md](CHANGELOG.md) - Version history
- [VERSION.json](VERSION.json) - Current version
- [docs/](docs/) - Complete documentation

---

## 🎓 Summary

Wazuh is a sophisticated security monitoring platform with:
- **Distributed architecture** (agents + manager)
- **Multi-layer security** (detection, prevention, response)
- **Cloud-native capabilities** (container, cloud monitoring)
- **Enterprise features** (clustering, RBAC, compliance)
- **Performance-optimized** (C/C++ core + Python framework)
- **Extensible** (API, plugins, cloud integrations)

The codebase shows a transition from traditional security monitoring to a modern, modular architecture supporting cloud-native deployments and advanced threat detection.
