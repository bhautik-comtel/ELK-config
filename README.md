# 🚀 ELK Stack Multi-Tier Architecture

![Architecture Diagram](./architecture-diagram.png)

> Centralized log collection and monitoring architecture using Elasticsearch, Logstash, Kibana, Fleet Server, and Elastic Agents.

---

# 📑 Table of Contents

- [Objective](#-objective)
- [Architecture Overview](#-architecture-overview)
- [Architecture Flow Diagram](#-architecture-flow-diagram)
- [Data Flow](#-data-flow)
- [Fleet Server Communication](#-fleet-server-communication)
- [Port Reference](#-port-reference)
- [Logstash Configurations](#-logstash-configurations)
- [Firewall Rules](#-firewall-rules)
- [Security Recommendations](#-security-recommendations)
- [Index Lifecycle Management](#-index-lifecycle-management)
- [Monitoring Recommendations](#-monitoring-recommendations)
- [Architecture Benefits](#-architecture-benefits)
- [Deployment Notes](#-deployment-notes)
- [Troubleshooting](#-troubleshooting)
- [Repository Structure](#-repository-structure)

---

# 🎯 Objective

This architecture is designed to:

- Centralize logs from distributed environments
- Improve security visibility
- Enable SIEM monitoring
- Provide scalable log ingestion
- Simplify troubleshooting
- Support Elastic Agent deployments

---

# 📖 Architecture Overview

This solution provides a centralized logging platform using:

- Logstash Edge Collectors
- Logstash Central Processing
- Elasticsearch Cluster
- Kibana Dashboards
- Fleet Server
- Elastic Agents

The architecture separates the **Client Network** and **Comtel Network** using secure firewall boundaries and NAT translation.

---

# 🏗️ Architecture Flow Diagram

```mermaid
flowchart LR

%% =========================
%% CLIENT NETWORK
%% =========================
subgraph CLIENT["🏢 CLIENT NETWORK"]

    KFW["Known Firewalls"]
    LNX["Linux Servers With Fleet Agent"]
    WIN["Windows Servers With Fleet Agent"]
    SW["Switches"]
    RTR["Routers"]
    UFW["Unknown Firewalls"]
    OTH["Other Devices"]

    EDGE["📥 Logstash Edge Collector"]

    KFW -->|"Agent Syslog 5514"| EDGE
    LNX -->|"Beats 5044"| EDGE
    WIN -->|"Beats 5044"| EDGE
    SW -->|"Syslog 514"| EDGE
    RTR -->|"Syslog 514"| EDGE
    UFW -->|"Syslog 514"| EDGE
    OTH -->|"Syslog 514"| EDGE

    CFW["🛡️ Client Firewall"]

    EDGE -->|"Edge to Central 20004"| CFW

    LNX -.->|"Fleet 20003"| CFW
    WIN -.->|"Fleet 20003"| CFW
end

%% =========================
%% COMTEL NETWORK
%% =========================
subgraph COMTEL["☁️ COMTEL NETWORK"]

    COMFW["🛡️ Comtel Firewall"]

    CENTRAL["📊 Logstash Central"]

    FLEET["⚙️ Fleet Server"]

    ES["🗄️ Elasticsearch"]

    KB["📈 Kibana"]

    COMFW -->|"9800"| CENTRAL

    CENTRAL -->|"9200 / HTTPS"| ES

    ES -->|"5601 / HTTPS"| KB

    COMFW -.->|"8220"| FLEET
end

CFW -->|"NAT 20004 → 9800"| COMFW

CFW -.->|"20003 → 8220"| COMFW
```

---

# 🔄 Data Flow

## Phase 1 — Log Collection

Client devices send logs to Logstash Edge Collector.

| Source | Protocol | Port |
|---|---|---|
| Firewalls | Syslog | 514 |
| Linux Servers | Beats | 5044 |
| Windows Servers | Beats | 5044 |
| Switches | Syslog | 514 |
| Routers | Syslog | 514 |
| Unknown Firewalls | Syslog | 514 |
| Known Firewalls | Agent Syslog | 5514 |

---

## Phase 2 — Edge to Central Forwarding

```text
Logstash Edge
    ↓
Client Firewall
    ↓
Comtel Firewall
    ↓
Logstash Central
```

Using:

```text
TCP 20004 → NAT → 9800
```

---

## Phase 3 — Elasticsearch Processing

```text
Logstash Central → Elasticsearch
```

Port:

```text
9200 / HTTPS
```

Functions:

- Parsing
- Filtering
- Enrichment
- Indexing

---

## Phase 4 — Visualization

```text
Elasticsearch → Kibana
```

Port:

```text
5601 / HTTPS
```

Features:

- Dashboards
- Search
- Alerting
- Monitoring

---

# ⚙️ Fleet Server Communication

Elastic Agents communicate with Fleet Server.

| Service | Port |
|---|---|
| Fleet Server | 8220 |

Features:

- Agent Enrollment
- Policy Management
- Monitoring
- Configuration Updates

---

# 🔌 Port Reference

| Port | Protocol | Purpose |
|------|-----------|----------|
| 514 | UDP | Syslog Collection |
| 5044 | TCP | Beats / Elastic Agent |
| 5514 | TCP | Syslog via Agent |
| 8220 | HTTPS | Fleet Server |
| 9200 | HTTPS | Elasticsearch API |
| 5601 | HTTPS | Kibana UI |
| 20004 | TCP | Edge to Central |
| 9800 | TCP | NAT Translated Central Port |

---

# 📥 Logstash Configurations

## Logstash Edge

```ruby
input {

  udp {
    port => 514
    type => "syslog"
  }

  tcp {
    port => 5044
    type => "beats"
  }

  tcp {
    port => 5514
    type => "agent-syslog"
  }
}

output {

  tcp {
    host => "logstash-central"
    port => 20004
    codec => json
  }
}
```

---

## Logstash Central

```ruby
input {

  tcp {
    port => 9800
    codec => json
  }
}

output {

  elasticsearch {
    hosts => ["https://elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

---

# 🔐 Firewall Rules

## Client Network

### Inbound

```text
UDP 514   ← Syslog Devices
TCP 5044  ← Elastic Agents
TCP 5514  ← Known Firewalls
```

### Outbound

```text
TCP 20004 → Comtel Network
TCP 8220  → Fleet Server
```

---

## Comtel Network

### Inbound

```text
TCP 9800 ← Logstash Edge
TCP 8220 ← Elastic Agents
```

---

# 🔐 Security Recommendations

- Enable TLS encryption
- Restrict Kibana access
- Enable Elasticsearch authentication
- Use API keys instead of passwords
- Apply firewall ACL restrictions
- Enable Logstash persistent queues

---

# 🗄️ Index Lifecycle Management

| Tier | Retention |
|------|------------|
| HOT | 0–7 Days |
| WARM | 7–30 Days |
| COLD | 30–90 Days |
| FROZEN | Archive |

---

# 📈 Monitoring Recommendations

- Monitor JVM Heap Usage
- Monitor Pipeline Delays
- Monitor Elasticsearch Cluster Health
- Monitor Disk Utilization
- Monitor Fleet Agent Status

---

# ✅ Architecture Benefits

- Centralized logging platform
- Secure multi-network architecture
- Scalable edge collection
- Fleet-based agent management
- Real-time monitoring
- Easier troubleshooting
- Distributed processing model

---

# 📌 Deployment Notes

- Use TLS wherever possible
- Configure Elasticsearch snapshots
- Enable Kibana authentication
- Monitor Logstash queues
- Configure ILM policies
- Use dedicated ingest pipelines

---

# 🔍 Troubleshooting

## Check Listening Ports

```bash
netstat -tulnp | grep -E "514|5044|5514|9800"
```

---

## Check Elasticsearch Health

```bash
curl -k https://localhost:9200/_cluster/health
```

---

## Check Fleet Server

```bash
curl -k https://fleet-server:8220
```

---

## Check Kibana

```bash
curl -k https://localhost:5601
```

---

# 📂 Repository Structure

```text
ELK-config/
│
├── README.md
├── architecture-diagram.png
├── configs/
│   ├── logstash-edge/
│   ├── logstash-central/
│   └── elastic-agent/
```

---

# 🚀 Future Improvements

- Multi-node Elasticsearch Cluster
- High Availability Logstash
- Kafka Integration
- GeoIP Enrichment
- Load Balancer for Fleet Server

---

# 👨‍💻 Maintainer

Bhautik  
Comtel Infrastructure Team

---

# 📜 License

Internal Infrastructure Documentation

---
