# Logstash Edge - Karna Configuration

This directory contains Logstash configuration for the **Karna** edge location.

## Purpose
Karna region log processing before forwarding to Central Logstash.

## Server Details
- **Location:** Karna Region
- **Role:** Regional Log Processing
- **Input:** Karna region data sources
- **Output:** Logstash Central (localhost:5000)

## Data Flow
```
Karna Data Sources
    ↓
Logstash Edge - Karna
    ├─ Parse regional logs
    ├─ Filter & enrich
    └─ Forward to Central
    ↓
Logstash Central (5000)
```

## Configuration Checklist
- [ ] Regional data sources connected
- [ ] Network optimized for Karna
- [ ] Pipelines tuned for Karna traffic
- [ ] Backup configured
- [ ] Monitoring enabled

## Important Links
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)