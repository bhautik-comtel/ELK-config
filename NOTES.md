# 📝 ELK Stack Architecture & Configuration Notes

> **Last Updated:** May 20, 2026
> **Repository:** bhautik-comtel/ELK-config

---

## 📋 Table of Contents
- [System Architecture](#system-architecture)
- [Infrastructure Flow](#infrastructure-flow)
- [Component Overview](#component-overview)
- [Server Inventory](#server-inventory)
- [Configuration Structure](#configuration-structure)
- [Deployment Guide](#deployment-guide)
- [Quick Reference](#quick-reference)

---

## 🏗️ System Architecture

### ELK Stack Topology

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLEET (Agent Management)                     │
└────────────────────┬────────────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
┌──────────────┐         ┌──────────────────┐
│ Logstash     │         │ Elasticsearch    │
│ Central      │────────▶│ HOT Tier         │
└──────────────┘         └────────┬─────────┘
        ▲                         │
        │                    ┌────┴────┐
        │                    │          │
        │                    ▼          ▼
        │            ┌──────────────┐ ┌──────────────┐
        │            │ Elastic ML   │ │ Elastic      │
        │            │ (Analytics)  │ │ Frozen Tier  │
        │            └──────────────┘ └──────────────┘
        │
   ┌────┴──────────────────────────────────────────┐
   │         Edge Logstash Instances               │
   ├─────────────────────────────────────────────┤
   │  • Karna    • SSPL    • Dhani               │
   │  • Farsight • Symphoni JP • GC • TP         │
   └────────────────────────────────────────────┘
        │
        └─────────────────────────────────┐
                                          │
                    ┌─────────────────────┘
                    │
                    ▼
        ┌─────────────────────────┐
        │   KIBANA (Dashboard)    │
        └─────────────────────────┘
```

---

## 📊 Infrastructure Flow

### Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA SOURCES                             │
│  (Applications, Systems, Devices)                               │
└────────────────────┬────────────────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼
┌─────────────────────┐   ┌──────────────────┐
│ FLEET AGENTS        │   │ DIRECT INPUTS    │
│ (Elastic Agents)    │   │ (APIs, Webhooks) │
└────────┬────────────┘   └────────┬─────────┘
         │                         │
         └────────────┬────────────┘
                      │
        ┌─────────────▼──────────────┐
        │  LOGSTASH EDGE INSTANCES   │
        ├────────────────────────────┤
        │ • Parse & Filter           │
        │ • Enrich & Transform       │
        │ • Local Processing         │
        └────────────┬───────────────┘
                     │
        ┌────────────▼───────────────┐
        │  LOGSTASH CENTRAL          │
        ├────────────────────────────┤
        │ • Aggregate Data           │
        │ • Final Transformation     │
        │ • Route to Elasticsearch   │
        └────────────┬───────────────┘
                     │
        ┌────────────▼──────────────┐
        │  ELASTICSEARCH CLUSTER    │
        ├───────────────────────────┤
        │ • HOT Tier (Recent Data)   │
        │ • ML Analytics            │
        │ • FROZEN Tier (Archive)   │
        └────────────┬──────────────┘
                     │
        ┌────────────▼──────────────┐
        │  KIBANA                   │
        ├───────────────────────────┤
        │ • Visualization           │
        │ • Dashboards              │
        │ • Analysis & Insights     │
        └───────────────────────────┘
```

---

## 🔧 Component Overview

### Fleet
- **Purpose:** Centralized agent management and deployment
- **Role:** Manages Elastic Agents across all servers
- **Config Path:** `./config/fleet/`
- **Port:** 5601 (via Kibana)

### Kibana
- **Purpose:** Data visualization and exploration platform
- **Role:** User interface for dashboards and analytics
- **Config Path:** `./config/kibana/`
- **Port:** 5601
- **Features:** Dashboards, Alerts, Canvas, Dev Tools

### Elasticsearch HOT Tier
- **Purpose:** Active data storage for real-time indexing
- **Role:** Primary node for recent data (0-7 days)
- **Config Path:** `./config/elasticsearch/hot/`
- **Port:** 9200
- **Data Retention:** Recent data

### Elasticsearch ML (Machine Learning)
- **Purpose:** Advanced analytics and anomaly detection
- **Role:** Performs ML jobs and forecasting
- **Config Path:** `./config/elasticsearch/ml/`
- **Port:** 9200
- **Features:** Anomaly detection, Forecasting, Clustering

### Elasticsearch FROZEN Tier
- **Purpose:** Long-term archival storage
- **Role:** Stores historical data (7+ days)
- **Config Path:** `./config/elasticsearch/frozen/`
- **Port:** 9200
- **Data Retention:** Archive data

### Logstash Central
- **Purpose:** Central data aggregation and final transformation
- **Role:** Receives data from edge instances and processes
- **Config Path:** `./config/logstash/central/`
- **Port:** 5000 (input), 9200 (output)
- **Pipelines:** Main processing pipelines

### Logstash Edge Instances

| Instance | Location | Config Path | Purpose |
|----------|----------|-------------|---------|
| **Karna** | Karna | `./config/logstash/edge/karna/` | Regional data collection |
| **SSPL** | SSPL | `./config/logstash/edge/sspl/` | Service-specific logs |
| **Dhani** | Dhani | `./config/logstash/edge/dhani/` | Distributed processing |
| **Farsight** | Farsight | `./config/logstash/edge/farsight/` | Advanced filtering |
| **Symphoni JP** | Japan | `./config/logstash/edge/symphoni-jp/` | Asia-Pacific region |
| **Symphoni GC** | Global Cloud | `./config/logstash/edge/symphoni-gc/` | Multi-cloud processing |
| **Symphoni TP** | TP Region | `./config/logstash/edge/symphoni-tp/` | TP region logs |

---

## 📁 Configuration Structure

```
ELK-config/
├── config/
│   ├── fleet/
│   │   ├── fleet-config.yml
│   │   ├── agent-policies.yml
│   │   └── integrations/
│   │
│   ├── kibana/
│   │   ├── kibana.yml
│   │   ├── dashboards/
│   │   ├── alerts/
│   │   └── saved-searches/
│   │
│   ├── elasticsearch/
│   │   ├── hot/
│   │   │   ├── elasticsearch.yml
│   │   │   ├── mappings/
│   │   │   └── index-templates/
│   │   ├── ml/
│   │   │   ├── elasticsearch.yml
│   │   │   ├── ml-jobs/
│   │   │   └── anomaly-detection/
│   │   └── frozen/
│   │       ├── elasticsearch.yml
│   │       └── ilm-policies/
│   │
│   ├── logstash/
│   │   ├── central/
│   │   │   ├── logstash.yml
│   │   │   ├── pipelines/
│   │   │   ├── patterns/
│   │   │   └── plugins/
│   │   │
│   │   └── edge/
│   │       ├── karna/
│   │       │   ├── logstash.yml
│   │       │   └── pipelines/
│   │       ├── sspl/
│   │       ├── dhani/
│   │       ├── farsight/
│   │       ├── symphoni-jp/
│   │       ├── symphoni-gc/
│   │       └── symphoni-tp/
│   │
│   └── docker/
│       ├── docker-compose.yml
│       ├── docker-compose.prod.yml
│       └── env-files/
│
├── scripts/
│   ├── deploy.sh
│   ├── health-check.sh
│   ├── backup.sh
│   └── restore.sh
│
├── NOTES.md (this file)
├── README.md
└── DEPLOYMENT.md
```

---

## 🚀 Deployment Guide

### Prerequisites
- Docker & Docker Compose 3.8+
- Elasticsearch 8.x+
- Kibana 8.x+
- Logstash 8.x+
- Elastic Agent 8.x+
- Minimum 16GB RAM per server
- Minimum 100GB disk space

### Deployment Steps

#### 1. Initialize Elasticsearch Cluster
```bash
# Deploy HOT tier
docker-compose -f config/elasticsearch/hot/docker-compose.yml up -d

# Deploy ML node
docker-compose -f config/elasticsearch/ml/docker-compose.yml up -d

# Deploy FROZEN tier
docker-compose -f config/elasticsearch/frozen/docker-compose.yml up -d
```

#### 2. Deploy Logstash Central
```bash
# Deploy central logstash
docker-compose -f config/logstash/central/docker-compose.yml up -d
```

#### 3. Deploy Logstash Edge Instances
```bash
# Deploy all edge instances
for instance in karna sspl dhani farsight symphoni-jp symphoni-gc symphoni-tp; do
  docker-compose -f config/logstash/edge/$instance/docker-compose.yml up -d
done
```

#### 4. Deploy Fleet & Kibana
```bash
# Deploy Kibana and Fleet
docker-compose -f config/kibana/docker-compose.yml up -d
```

#### 5. Enroll Elastic Agents
```bash
# Enroll agents via Fleet UI or CLI
elastic-agent install --url=<kibana-url> --enrollment-token=<token>
```

---

## 🔍 Server Configuration Checklist

### Each Edge Logstash Server
- [ ] Network connectivity to Central Logstash
- [ ] TLS certificates installed
- [ ] Log directory permissions set
- [ ] Pipeline configuration validated
- [ ] Monitoring enabled

### Central Logstash Server
- [ ] Elasticsearch cluster connectivity
- [ ] Dead letter queue (DLQ) configured
- [ ] Monitoring and alerting enabled
- [ ] Backpressure handling configured

### Elasticsearch Nodes
- [ ] Cluster discovery configured
- [ ] Index Lifecycle Management (ILM) policies set
- [ ] Snapshot repositories configured
- [ ] Security (TLS/RBAC) enabled

### Kibana Server
- [ ] Elasticsearch connection verified
- [ ] Fleet enabled and configured
- [ ] RBAC roles created
- [ ] Dashboards imported

---

## 📌 Quick Reference

### Health Checks

```bash
# Check Elasticsearch cluster health
curl -u elastic:password http://localhost:9200/_cluster/health

# Check Logstash connectivity
curl -u logstash:password http://localhost:9600

# Check Kibana status
curl http://localhost:5601/api/status
```

### Common Commands

```bash
# View Logstash pipeline status
curl -u logstash:password http://localhost:9600/_node/pipelines

# Create Elasticsearch index
curl -X PUT "localhost:9200/my-index"

# List Fleet agents
curl -X GET "localhost:5601/api/fleet/agents"

# Restart specific service
docker-compose -f config/service/docker-compose.yml restart
```

### Monitoring Ports

| Service | Port | Protocol |
|---------|------|----------|
| Elasticsearch | 9200 | HTTP/HTTPS |
| Kibana | 5601 | HTTP/HTTPS |
| Logstash Central | 5000 | TCP/TLS |
| Logstash Monitoring | 9600 | HTTP |
| Fleet | 5601 | HTTP/HTTPS |

---

## 🔗 Important Links

- [Elasticsearch Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)
- [Kibana Documentation](https://www.elastic.co/guide/en/kibana/current/index.html)
- [Fleet Documentation](https://www.elastic.co/guide/en/fleet/current/index.html)
- [Index Lifecycle Management](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)

---

## 📞 Support & Maintenance

**Regular Tasks:**
- ✅ Daily: Monitor cluster health and alerts
- ✅ Weekly: Review ILM policies and storage
- ✅ Monthly: Update configurations and plugins
- ✅ Quarterly: Backup and disaster recovery testing

**Escalation:**
1. Check logs: `docker-compose logs -f <service>`
2. Review health checks
3. Contact infrastructure team
4. Open GitHub issue with diagnostics

---

**Last Maintained:** May 20, 2026 | **Version:** 1.0
