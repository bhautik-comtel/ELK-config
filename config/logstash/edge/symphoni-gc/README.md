# Logstash Edge - Symphoni GC Configuration

This directory contains Logstash configuration for the **Symphoni GC** (Global Cloud) edge location.

## Purpose
Global Cloud multi-cloud log processing before forwarding to Central Logstash.

## Server Details
- **Location:** Global Cloud
- **Role:** Multi-Cloud Log Processing
- **Input:** Global Cloud data sources
- **Output:** Logstash Central (localhost:5000)

## Data Flow
```
Symphoni GC Data Sources
    ↓
Logstash Edge - Symphoni GC
    ├─ Parse multi-cloud logs
    ├─ Cloud-specific enrichment
    └─ Forward to Central
    ↓
Logstash Central (5000)
```

## Configuration Checklist
- [ ] Multi-cloud data sources connected
- [ ] Cloud-specific parsing rules configured
- [ ] Pipelines optimized for cloud traffic
- [ ] Cross-cloud aggregation working
- [ ] Monitoring enabled

## Important Links
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)