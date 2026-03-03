# Wazuh Project - Quick Start Guide

## 🎯 You Are Here

If you've just opened this workspace, here's your roadmap to understand the entire Wazuh project.

---

## 📚 Documentation You Have

I've created 4 comprehensive guides in this repository:

| Document | Purpose | Start When |
|----------|---------|-----------|
| **PROJECT_UNDERSTANDING.md** | High-level overview of the entire system | 👈 **START HERE** |
| **ARCHITECTURE_DIAGRAMS.md** | Visual diagrams showing data flow & interactions | You want to visualize how components talk |
| **CODEBASE_NAVIGATION.md** | How to find specific features in the code | You need to modify/debug something |
| **PRACTICAL_EXAMPLES.md** | Real commands, configs, and code examples | You want hands-on reference material |

---

## 🚀 Quick Start Path (30 minutes)

### 1. Understand What Wazuh Does (5 min)
Read: **PROJECT_UNDERSTANDING.md** → "Project Overview" section

**Key takeaway**: Wazuh is a security monitoring platform with agents on endpoints and a central manager that analyzes logs and detects threats.

### 2. Learn the Architecture (10 min)
Read: **ARCHITECTURE_DIAGRAMS.md** → Section 1 & 2

**Key takeaway**: 
- Agents collect data → Manager analyzes via rules → Alerts generated → Visible in dashboard

### 3. Find Key Components (5 min)
Read: **CODEBASE_NAVIGATION.md** → "Quick Module Reference"

**Key takeaway**: Know where to find code for:
- Agent communication: `/src/remoted/`
- Log analysis: `/src/engine/`
- API: `/api/api/`
- Python framework: `/framework/wazuh/`

### 4. See Practical Examples (10 min)
Skim: **PRACTICAL_EXAMPLES.md** → Configuration & Commands sections

**Key takeaway**: How to actually use Wazuh (install, configure, manage agents)

---

## 🎓 Deep Dives

### If you want to understand...

**How agents send data to the manager**
1. Read: ARCHITECTURE_DIAGRAMS.md → Section 2 (Data Flow)
2. Navigate to: `/src/remoted/` and `/src/logcollector/`
3. Check: CODEBASE_NAVIGATION.md → "Daemon Processes" table

**How rules work**
1. Read: ARCHITECTURE_DIAGRAMS.md → Section 2 (Analysis pipeline)
2. Navigate to: `/src/engine/` 
3. Example config: PRACTICAL_EXAMPLES.md → "Manager Configuration"

**File Integrity Monitoring (FIM)**
1. Read: ARCHITECTURE_DIAGRAMS.md → Section 4
2. Navigate to: `/src/syscheckd/` and `/architecture/FIM/`
3. Config example: PRACTICAL_EXAMPLES.md → FIM configuration

**Cloud security & integrations**
1. Read: PROJECT_UNDERSTANDING.md → "Cloud Security" & "Integration Points"
2. Navigate to: `/wodles/aws/`, `/architecture/wm_azure/`, etc.
3. Find data providers: `/architecture/data_provider/`

**API development**
1. Read: PROJECT_UNDERSTANDING.md → "API Layer"
2. Navigate to: `/api/api/controllers/` and `/api/api/spec/spec.yaml`
3. Framework logic: `/framework/wazuh/`

**Building & Compilation**
1. Read: PRACTICAL_EXAMPLES.md → "Build & Installation"
2. Check: `/src/CMakeLists.txt` and `/src/Makefile`
3. Install script: `install.sh` (root level)

---

## 🔍 Find Specific Code

### Use CODEBASE_NAVIGATION.md Table

**Example**: "Where's the code that handles agent registration?"

1. Open CODEBASE_NAVIGATION.md
2. Search for "agent registration"
3. Found: `agent management` → `/src/addagent/` (C) and `/framework/wazuh/agent.py` (Python)

### Use File Search in VS Code

**Example**: Finding where alerts are stored

```
Ctrl+K Ctrl+O (or Cmd+K Cmd+O on Mac)
Type: **/alerts.json
Review: /var/ossec/logs/alerts/alerts.json
```

### Use Grep Search

**Example**: Finding where remoted daemon is configured

```
Search in workspace for: "remoted"
Results show configuration, startup, and usage locations
```

---

## 💡 Key Concepts

### 1. **Agents vs Manager**
- **Agents**: Run on endpoints, collect data (logs, files, processes)
- **Manager**: Central server, analyzes data, generates alerts

### 2. **Data Flow Pipeline**
```
Logs/Events → Parsing → Rules Engine → Alert Generation → Storage
```

### 3. **Security Rules**
- **Decoders**: Parse/extract fields from logs
- **Rules**: Match patterns, trigger on specific conditions
- **Active Response**: Execute countermeasures when rules trigger

### 4. **Communication**
- Agents → Manager: Encrypted TLS/SSL on port 1514
- Uses shared secrets (encryption keys) for authentication

### 5. **Storage**
- Local: SQLite database in `/var/ossec/queue/`
- Remote: Elasticsearch for long-term storage & analytics

---

## 🎯 Common Tasks

### "I want to understand how X works"

**Step 1**: Find it in **ARCHITECTURE_DIAGRAMS.md** - see visual flow
**Step 2**: Find source code in **CODEBASE_NAVIGATION.md** - know the file location  
**Step 3**: Read actual file in `/src/` or `/framework/` - understand implementation
**Step 4**: Check tests in `/src/unit_tests/` or `/framework/wazuh/tests/` - see usage patterns

### "I want to modify/add a feature"

**Step 1**: Understand the component (use diagrams)
**Step 2**: Locate the code
**Step 3**: Write/modify code
**Step 4**: Check related tests
**Step 5**: Rebuild with `make` targets
**Step 6**: Test changes

### "I want to debug an issue"

**Step 1**: Check logs: `/var/ossec/logs/ossec.log`
**Step 2**: Enable debug mode in **PRACTICAL_EXAMPLES.md** → "Debugging Tips"
**Step 3**: Reproduce issue
**Step 4**: Analyze debug output
**Step 5**: Find relevant code files
**Step 6**: Add debug logic if needed

---

## 📖 File Organization

```
/Users/shivam/wazuh/                           ← You are here

📄 Documentation (NEW - I created these)
├── PROJECT_UNDERSTANDING.md          ← System overview
├── ARCHITECTURE_DIAGRAMS.md          ← Visual diagrams
├── CODEBASE_NAVIGATION.md            ← Where to find things
└── PRACTICAL_EXAMPLES.md             ← How to use Wazuh

📁 Source Code Structure
├── /src/                             ← C/C++ core (agents & manager)
├── /api/                             ← REST API (Python)
├── /framework/                       ← Python business logic
├── /architecture/                    ← Modern modular components
├── /wodles/                          ← Python integration modules
├── /etc/                             ← Config templates
├── /docs/                            ← Official documentation
├── /tests/                           ← Test suites
├── /tools/                           ← Utility scripts
└── /packages/                        ← Package building

🔧 Build & Install
├── install.sh                        ← Main installer
├── upgrade.sh                        ← Upgrade script
├── VERSION.json                      ← Current version
├── INSTALL                           ← Installation guide  
├── README.md                         ← Project overview
└── CHANGELOG.md                      ← Release notes
```

---

## 🎓 Learning Progression

### **Level 1: Basics (Today)**
- [ ] Read: PROJECT_UNDERSTANDING.md (overview section)
- [ ] Read: ARCHITECTURE_DIAGRAMS.md (section 1 only)
- [ ] Run: `ls -la /Users/shivam/wazuh/` and explore the structure
- **Outcome**: Understand what Wazuh does and major components

### **Level 2: Architecture (Day 1-2)**
- [ ] Read: All of ARCHITECTURE_DIAGRAMS.md
- [ ] Read: CODEBASE_NAVIGATION.md (all sections)
- [ ] Explore: Key directories in VS Code sidebar
- [ ] Check: CMakeLists.txt and Makefile to understand build
- **Outcome**: Know how components interact and where code lives

### **Level 3: Code Exploration (Day 2-3)**
- [ ] Open: `/src/remoted/` - understand agent communication
- [ ] Open: `/src/engine/` - understand rule matching
- [ ] Open: `/framework/wazuh/` - understand Python framework
- [ ] Open: `/api/api/controllers/` - understand API design
- **Outcome**: Can navigate and read actual source code

### **Level 4: Hands-On (Day 3+)**
- [ ] Read: PRACTICAL_EXAMPLES.md (build & installation)
- [ ] Try: Building Wazuh from source
- [ ] Try: Installing locally or in VM
- [ ] Try: Running actual commands from PRACTICAL_EXAMPLES.md
- [ ] Try: Modifying a simple rule and testing
- **Outcome**: Can build, deploy, and test Wazuh

---

## 🔑 Quick Reference - Key Directories

| Directory | Language | Purpose | When to Visit |
|-----------|----------|---------|---------------|
| `/src/remoted/` | C/C++ | Agent-Manager communication | Understanding data transmission |
| `/src/engine/` | C/C++ | Rule matching engine | Understanding threat detection |
| `/src/logcollector/` | C/C++ | Log collection daemon | Understanding log processing |
| `/framework/wazuh/` | Python | Manager business logic | Understanding API & operations |
| `/api/api/` | Python | REST API implementation | Developing API clients |
| `/wodles/aws/` | Python | AWS integration | Cloud monitoring |
| `/architecture/data_provider/` | C++ | Data collection framework | New data source integration |
| `/etc/` | Config | Configuration templates | Understanding configuration options |
| `/ruleset/` | Rules | Detection rules | Creating/modifying security rules |

---

## 🎯 My Recommendations

### Start With These Files (in this order)

1. **Read** [PROJECT_UNDERSTANDING.md](PROJECT_UNDERSTANDING.md) - 15 minutes
   - Get the big picture
   
2. **Look at** [ARCHITECTURE_DIAGRAMS.md](ARCHITECTURE_DIAGRAMS.md) - 10 minutes
   - See how it works visually

3. **Reference** [CODEBASE_NAVIGATION.md](CODEBASE_NAVIGATION.md) - As needed
   - Find code locations

4. **Use** [PRACTICAL_EXAMPLES.md](PRACTICAL_EXAMPLES.md) - When doing hands-on work
   - Commands, configs, API calls

### Then Explore The Repo

```
In VS Code:
1. Expand /src/ → Find remoted/ and engine/ → Read some .c/.h files
2. Expand /framework/wazuh/ → Open agent.py, manager.py
3. Expand /api/api/ → Open controllers/ and see endpoint implementations
4. Open install.sh → See the build process
5. Open CMakeLists.txt → Understand compilation
```

---

## ❓ FAQ

**Q: Is this a monolithic or microservices architecture?**  
A: Hybrid. `/src/` is monolithic C/C++ (legacy). `/architecture/` shows transition to modular components.

**Q: What's written in C vs Python?**  
A: C/C++ = high-performance core (agents, managers daemons). Python = framework, API, integrations, management tools.

**Q: Can I run Wazuh without compiling?**  
A: Yes, use pre-built packages. For development, you'll want to compile.

**Q: What's the easiest component to understand?**  
A: The Python framework in `/framework/wazuh/` - it's readable and well-organized.

**Q: What's the hardest component?**  
A: The rule matching engine in `/src/engine/` - it's complex C/C++ with lots of optimization.

**Q: Where do the security rules live?**  
A: `/ruleset/` (included in repo) and `/var/ossec/ruleset/` (after installation).

**Q: How do I contribute?**  
A: Check [CONTRIBUTORS](CONTRIBUTORS) file. Typical flow: fork → branch → test → PR.

---

## 🚀 Next Steps

### Short Term
1. Read all 4 documentation files I created ✓
2. Explore the directory structure in VS Code
3. Pick one component and deep-dive into its code

### Medium Term  
1. Build Wazuh from source (see PRACTICAL_EXAMPLES.md)
2. Install it locally or in a VM
3. Run some actual commands and see outputs
4. Create a test rule and verify it works

### Long Term
1. Understand each major component deeply
2. Identify potential improvements
3. Make a contribution back to the project
4. Customize Wazuh for your organization

---

## 📞 Need Help?

All answers are in the 4 guides I created:

- **Looking for code?** → [CODEBASE_NAVIGATION.md](CODEBASE_NAVIGATION.md)
- **Want visual diagrams?** → [ARCHITECTURE_DIAGRAMS.md](ARCHITECTURE_DIAGRAMS.md)
- **Need commands/configs?** → [PRACTICAL_EXAMPLES.md](PRACTICAL_EXAMPLES.md)
- **Want big picture?** → [PROJECT_UNDERSTANDING.md](PROJECT_UNDERSTANDING.md)

---

## 📊 Project Stats

| Metric | Value |
|--------|-------|
| **Version** | 5.0.0-alpha0 |
| **License** | GPLv2 |
| **Language** | C/C++ (core) + Python (framework/tools) |
| **Main Directories** | 15+ major components |
| **Installation Path** | `/var/ossec/` (default) |
| **Key Ports** | 1514 (agent-manager), 55000 (API), 9200 (Elasticsearch) |
| **Lines of Code** | ~1 million (estimated) |

---

## ✅ You're Ready!

You now have:
- ✅ Complete project overview
- ✅ Visual architecture diagrams
- ✅ Navigation guide to codebase
- ✅ Practical examples and commands
- ✅ This quick-start checklist

**Next: Pick a component and start exploring!**

---

**Created**: March 2026  
**For**: Understanding the Wazuh Security Monitoring Platform  
**Questions?**: Refer to the 4 documentation files created in this workspace
