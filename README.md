# End-to-End SOC Automation using Open-Source SOAR Tools

## 📘 Overview
This repository contains an **automated Security Operations Center (SOC)** implementation using open-source security tools — **Wazuh**, **TheHive**, **Cortex**, **MISP**, **Kibana**, and **OpenSearch Dashboard**. The project demonstrates comprehensive automation in threat detection, incident response, and case management.

### Key Capabilities
- **Threat Detection:** Wazuh monitors endpoints and generates security alerts
- **Alert Automation:** Custom integration scripts forward Wazuh alerts to TheHive
- **Incident Management:** Centralized case creation and tracking in TheHive
- **Threat Intelligence:** MISP integration for IOC sharing and enrichment
- **Automated Analysis:** Cortex performs dynamic malware analysis on artifacts
- **Visualization:** Real-time dashboards via Wazuh Dashboard and OpenSearch
- **Orchestrated Deployment:** Complete Docker Compose stack for rapid deployment

## 👥 Team Members
- Abdul Rafay  
- Kamil Saeed  

**Institution:** FAST National University of Computer and Emerging Sciences (FAST NUCES)  
**Department:** Cyber Security  
**Course:** Network Security

---

## 📁 Repository Structure

```
.
├── Code/                          # Integration and configuration scripts
│   ├── custom-w2thive.py         # Wazuh-to-TheHive integration script
│   ├── custom-w2thive/           # Integration package
│   ├── ossec.conf                # Wazuh integration configuration template
│   └── README.md                 # Integration documentation
│
├── setup-dockers/                # Docker Compose configurations
│   ├── wazuh_dockers/            # Single-node Wazuh deployment
│   │   ├── docker-compose.yml    # Wazuh stack orchestration
│   │   ├── generate-indexer-certs.yml
│   │   ├── config/               # Wazuh configuration files
│   │   └── README.md
│   │
│   └── cortex_misp_thehive/      # TheHive, Cortex, MISP stack
│       ├── docker-compose.yml    # Full SOC stack orchestration
│       └── README.md
│
├── docs/                          # Project documentation
│   ├── FinalReport.pdf           # Comprehensive project report
│   ├── Proposal.pdf              # Project proposal
│   └── Architectural_Diagram.png # System architecture visualization
│
├── Presentation/                  # Project presentation materials
│   └── screenshots/              # Deployment and integration screenshots
│       ├── initital_setup/
│       ├── integration/
│       └── simulation/
│
└── README.md                      # This file
```

---

## 🔧 Core Components

### 1. **Wazuh** - Security Monitoring & Detection
- **Role:** Endpoint detection and response (EDR), log analysis, threat detection
- **Deployment:** Single-node configuration via Docker Compose
- **Features:**
  - Real-time threat detection with 3000+ compliance rules
  - Agent-based monitoring across endpoints
  - Integrity monitoring and vulnerability assessment
  - Log aggregation and analysis
  - OpenSearch Dashboard & Wazuh Dashboard visualization

### 2. **TheHive** - Incident Response Platform
- **Role:** Centralized case management and incident investigation
- **Deployment:** Docker Compose with Cassandra backend & MinIO storage
- **Features:**
  - Automated alert to case conversion
  - Artifact tracking and IOC management
  - Collaboration and case workflow management
  - Observable analysis and timeline tracking
  - REST API for integrations (v1)

### 3. **Cortex** - Automated Threat Analysis
- **Role:** Dynamic malware analysis and threat intelligence enrichment
- **Deployment:** Docker container with Elasticsearch backend
- **Features:**
  - Automated artifact analysis (files, domains, IPs, URLs)
  - Integration with threat intelligence services
  - Custom analyzer support
  - Analysis pipeline orchestration

### 4. **MISP** - Threat Intelligence Sharing
- **Role:** Centralized threat intelligence platform
- **Deployment:** Docker Compose with MySQL backend
- **Features:**
  - IOC (Indicator of Compromise) management
  - Threat event correlation
  - Sharing and distribution of threat intelligence
  - Event creation and enrichment

### 5. **OpenSearch & Elasticsearch** - Analytics & Indexing
- Real-time log indexing and search capabilities
- Dashboard creation and visualization
- Alert aggregation and analytics

---

## 🚀 Quick Start

### Prerequisites
- **Docker & Docker Compose** installed on your system
- **Minimum 8GB RAM** and 20GB disk space
- **Linux/MacOS/Windows** with Docker Engine support

### Deploy Wazuh Stack
```bash
cd setup-dockers/wazuh_dockers/

# Generate SSL certificates
docker-compose -f generate-indexer-certs.yml run --rm generator

# Start Wazuh services
docker-compose up -d
```

**Access Wazuh Dashboard:** `https://localhost:443`  
**Default Credentials:** `admin:SecretPassword` (change on first login)

### Deploy TheHive, Cortex & MISP
```bash
cd setup-dockers/cortex_misp_thehive/

# Start SOC stack
docker-compose up -d
```

**Access Endpoints:**
- TheHive: `http://localhost:9000`
- Cortex: `http://localhost:9001`
- MISP: `http://localhost:8081`

---

## 🔗 Integration Flow

```
┌─────────────────────────────────────────────────────────────┐
│                     Detection & Response                     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Endpoint/Server  →  Wazuh Agent  →  Wazuh Manager          │
│     (Events)         (Collect)       (Correlate)            │
│                                          ↓                   │
│                                  Alert Rules Triggered       │
│                                          ↓                   │
│                          custom-w2thive.py Integration       │
│                        (Parse, Extract, Format Alert)        │
│                                          ↓                   │
│                                 TheHive API (v1)             │
│                            POST /api/alert (Create)          │
│                                          ↓                   │
│  TheHive Dashboard  ←  Alert Created  ←  TheHive Backend    │
│  (Case Pending)         (Visible)        (Cassandra/MinIO)   │
│       ↓                                       ↓               │
│  Analyst Reviews   ↔  Artifacts Extracted  ←  IOC Detection  │
│  Case Details          (IPs, URLs, Domains, Files)          │
│       ↓                                                      │
│  Send to Cortex                                              │
│       ↓                                                      │
│  Automated Analysis  →  Threat Intelligence  →  MISP        │
│                         (Enrichment)           (Sharing)     │
│       ↓                                                      │
│  Analysis Results  →  Update Case  →  Close/Escalate        │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 📋 Integration Details

### Wazuh-to-TheHive Integration (`custom-w2thive.py`)

**Functionality:**
- Parses Wazuh alert JSON payloads
- Extracts artifacts (IPs, URLs, domains) using regex pattern matching
- Applies configurable severity thresholds:
  - Wazuh alerts: Threshold level 0 (default)
  - Suricata alerts: Threshold level 3 (default)
- Creates TheHive alerts with formatted descriptions, tags, and artifacts
- Sets TLP (Traffic Light Protocol) level = Amber

**Configurable Parameters:**
```python
lvl_threshold = 0              # Wazuh rule level threshold
suricata_lvl_threshold = 3     # Suricata rule level threshold
debug_enabled = False          # Enable debug logging
info_enabled = True            # Enable info logging
```

**Alert Tags Generated:**
- `wazuh` - Source platform identifier
- `rule=<id>` - Wazuh rule ID
- `agent_name=<name>` - Originating agent name
- `agent_id=<id>` - Wazuh agent ID
- `agent_ip=<ip>` - Agent IP address

**Dependencies:** `thehive4py` (TheHive Python API client)

### Wazuh Integration Configuration (`ossec.conf`)

```xml
<integration>
  <name>custom-w2thive</name>
  <hook_url>http://thehive-server:9000</hook_url>
  <api_key>YOUR_THEHIVE_API_KEY</api_key>
  <alert_format>json</alert_format>
</integration>
```

---

## ✅ Project Status

**Completed:**
- ✅ Wazuh single-node Docker deployment
- ✅ TheHive 5.2 with Cortex and MISP Docker stack
- ✅ Custom Wazuh-to-TheHive integration script
- ✅ Alert parsing, artifact extraction, and filtering logic
- ✅ Comprehensive documentation and setup guides
- ✅ System architecture diagrams
- ✅ Demonstration screenshots and walkthroughs

**Features Implemented:**
- Real-time alert forwarding from Wazuh to TheHive
- Automated artifact detection (IPs, URLs, domains)
- Severity-based alert filtering
- TheHive case creation with enriched metadata
- Integration with Cortex for automated analysis
- MISP integration for threat intelligence

---

## 📚 Documentation

- **FinalReport.pdf** - Comprehensive project report with architecture, implementation details, testing results, and conclusions
- **Proposal.pdf** - Original project proposal and requirements
- **Code/README.md** - Detailed integration script documentation
- **setup-dockers/README.md** - Docker deployment instructions
- **Architectural_Diagram.png** - System architecture visualization

---

## 🔐 Security Notes

- Store TheHive API keys securely (use environment variables in production)
- Restrict access to Wazuh and TheHive dashboards with strong authentication
- Use HTTPS/TLS for all inter-service communication
- Implement network segmentation and firewall rules
- Regularly update containers and dependencies for security patches
- Monitor logs for suspicious activities and integration errors

---

## 🚧 Future Enhancements

Potential improvements for extended functionality:
- Multi-rule threshold configuration per rule ID
- Support for additional artifact types (CVEs, email addresses, file hashes)
- Webhook endpoints for reverse notifications
- Batch alert processing for high-volume scenarios
- Machine learning-based alert correlation and deduplication
- Automated response playbooks and remediation actions
- Enhanced compliance reporting (HIPAA, PCI-DSS, SOC2)

---

## 📖 References & Documentation

- [Wazuh Official Documentation](https://documentation.wazuh.com/)
- [TheHive4py GitHub Repository](https://github.com/TheHive-Project/TheHive4py)
- [TheHive API Documentation](https://docs.thehive-project.org/api/)
- [Cortex Documentation](https://github.com/TheHive-Project/Cortex)
- [MISP Documentation](https://misp.github.io/)
- [Docker Documentation](https://docs.docker.com/)

---

## 📄 License

This project is licensed under the **MIT License** - see the LICENSE file for details.

---

## 📞 Support & Contact

For questions, issues, or contributions, please refer to the project documentation or contact the team members listed above.
