# Kibana Configuration

This directory contains **Kibana** (visualization and exploration platform) configuration.

## Purpose
Kibana is the UI for Elasticsearch data visualization, dashboarding, and analytics.

## Server Details
- **Role:** Data Visualization & UI
- **Port:** 5601
- **Components:** Dashboards, Alerts, Canvas, Dev Tools, Fleet

## Contents
- **kibana.yml** - Main Kibana configuration
- **dashboards/** - Pre-built dashboards
- **alerts/** - Alert rules configuration
- **saved-searches/** - Saved search queries
- **canvas/** - Canvas workpads
- **docker-compose.yml** - Docker deployment

## Key Features
- 📊 Interactive Dashboards
- 🔔 Alerting & Actions
- 🔍 Advanced Search
- 📈 Canvas for presentations
- 🛡️ Security & RBAC
- 🔌 Fleet agent management
- 🧠 ML job integration

## Configuration Structure
```
kibana/
├── kibana.yml              - Main configuration
├── docker-compose.yml      - Docker deployment
├── dashboards/
│   ├── system-overview.ndjson
│   ├── infrastructure.ndjson
│   └── security.ndjson
├── alerts/
│   ├── high-cpu.json
│   ├── disk-space.json
│   └── error-spike.json
├── saved-searches/
│   ├── errors.json
│   ├── warnings.json
│   └── critical.json
└── canvas/
    └── status-board.json
```

## Kibana Configuration Example
```yaml
# Server
server.name: kibana
server.host: 0.0.0.0
server.port: 5601

# Elasticsearch
elasticsearch.hosts: ["http://elasticsearch:9200"]
elasticsearch.username: elastic
elasticsearch.password: ${ELASTIC_PASSWORD}

# Kibana settings
kibana.defaultAppId: home
kibana.index: .kibana

# Fleet settings
xpack.fleet.enabled: true
xpack.fleet.agents.enabled: true

# Monitoring
xpack.monitoring.enabled: true
xpack.monitoring.collection.enabled: true

# Security
xpack.security.enabled: true
xpack.encryptedSavedObjects.encryptionKey: ${ENCRYPTION_KEY}
```

## Dashboard Categories
| Dashboard | Purpose | Data Sources |
|-----------|---------|---------------|
| System Overview | CPU, Memory, Disk | System metrics |
| Infrastructure | Services, Hosts, Network | Node metrics |
| Security | Logins, Failures, Anomalies | Security logs |
| Application | Performance, Errors, Traces | App logs |
| Alerts | Alert status, Incidents | Alert rules |

## Alert Configuration Example
```json
{
  "name": "High CPU Alert",
  "enabled": true,
  "condition": {
    "type": "threshold",
    "threshold": 80
  },
  "actions": [
    {
      "type": "email",
      "to": "ops@company.com"
    },
    {
      "type": "slack",
      "channel": "#alerts"
    }
  ]
}
```

## Configuration Checklist
- [ ] Elasticsearch connectivity verified
- [ ] Authentication configured
- [ ] Dashboards imported
- [ ] Alert rules created
- [ ] Fleet enabled and configured
- [ ] RBAC roles defined
- [ ] Monitoring enabled
- [ ] Backup procedures configured

## Fleet Integration
- Enable agent enrollment
- Create agent policies
- Configure integrations
- Set up logs/metrics collection
- Deploy to all edge locations

## Important Links
- [Kibana Documentation](https://www.elastic.co/guide/en/kibana/current/index.html)
- [Fleet Documentation](https://www.elastic.co/guide/en/fleet/current/index.html)
- [Alert Documentation](https://www.elastic.co/guide/en/kibana/current/alerting-getting-started.html)